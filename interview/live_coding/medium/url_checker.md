# Проверка URL

## Условие

### Часть 1

Напишите программу, которая:

- Поочередно выполняет HTTP-запросы по предложенному списку ссылок.
- Если HTTP-код ответа — 200 OK, печатает: адрес url — ok.
- Если HTTP-код ответа отличается от 200 OK либо произошла ошибка, печатает: адрес url — not ok.

### Часть 2

Модифицируйте программу, чтобы основной поток и горутины общались через каналы.

Пример:

- Запросы по списку выполняются в горутинах.
- Результаты печатаются в основном потоке.

### Часть 3

Модифицируйте программу так, чтобы нигде не использовалась длина слайса URL. Считайте, что URL приходят из внешнего источника и заранее неизвестно, сколько их будет. Предложите идиоматичный вариант, как программа узнает об окончании списка и передает этот сигнал действующим горутинам дальше.

### Часть 4

Модифицируйте программу так, чтобы после получения первых двух ответов 200 OK остальные запросы штатно завершались. При этом нужно напечатать сообщение о завершении запроса.

## Решение

### Решение часть 1

Здесь достаточно просто пройти по списку и стандартным GET попробоват получить ресурс:

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    urls := []string{
        "url1",
        "url2",
        "url3",
    }

    for _, url := range urls {
        resp, err := http.Get(url)
        if err != nil {
            fmt.Println(url, "- not ok")
            continue
        }

        if resp.StatusCode == http.StatusOK {
            fmt.Println(url, "- ok")
        } else {
            fmt.Println(url, "- not ok")
        }

        resp.Body.Close()
    }
}
```

### Решение часть 2

В данном случае можно завести канал для результатов и туда скидывать результат выполенния GET.

Для удобства можно также завести структуру:

```go
package main

import (
    "fmt"
    "net/http"
    "sync"
)

type result struct {
    url  string
    isOK bool
}

func checkURL(url string, results chan<- result) {
    resp, err := http.Get(url)
    if err != nil {
        results <- result{url: url, isOK: false}
        return
    }
    defer resp.Body.Close()

    results <- result{
        url:  url,
        isOK: resp.StatusCode == http.StatusOK,
    }
}

func main() {
    urls := []string{
        "url1",
        "url2",
        "url3",
    }

    results := make(chan result)
    var wg sync.WaitGroup

    for _, url := range urls {
        wg.Add(1)

        go func(url string) {
            defer wg.Done()
            checkURL(url, results)
        }(url)
    }

    go func() {
        wg.Wait()
        close(results)
    }()

    for res := range results {
        if res.isOK {
            fmt.Println(res.url, "- ok")
        } else {
            fmt.Println(res.url, "- not ok")
        }
        }
}
```

Важно: не забыть, что надо завести wait group - для отслеживания ситуации, что все горутины завершились и мы можем закрыть канал.

### Решение часть 3

В данном случае можно завести канал для получения url-ов. Закрытие этого канала - это и будет сигнал 'об окончании списка':

```go
package main

import (
    "fmt"
    "net/http"
    "sync"
)

type result struct {
    url  string
    isOK bool
}

func checkURL(url string, results chan<- result) {
    resp, err := http.Get(url)
    if err != nil {
        results <- result{url: url, isOK: false}
        return
    }
    defer resp.Body.Close()

    results <- result{
    url:  url,
        isOK: resp.StatusCode == http.StatusOK,
    }
}

func main() {
    urlsCh := make(chan string)
    results := make(chan result)    
    var wg sync.WaitGroup   

    go func() {
        urlsCh <- "https://go.dev"
        urlsCh <- "https://example.com"
        close(urlsCh) // сигнaл: URL закончились
    }() 
    
    go func() {
        for url := range urlsCh {
            wg.Add(1)   
            go func(url string) {
                defer wg.Done()
                checkURL(url, results)
            }(url)
        }   
        wg.Wait()
        close(results)
    }() 
    
    for res := range results {
        if res.isOK {
            fmt.Println(res.url, "- ok")
        } else {
            fmt.Println(res.url, "- not ok")
        }
    }
}
```

### Решение часть 4

Здесь воспользуемся отменой контекста после получения двух 200-ок. Для этого будем прокидывать context в checkURL.

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "net/http"
    "sync"
)

type result struct {
    url  string
    isOK bool
    err  error
}

func checkURL(ctx context.Context, url string, results chan<- result) {
    req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
    if err != nil {
        results <- result{url: url, err: err}
        return
    }

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        results <- result{url: url, err: err}
        return
    }
    defer resp.Body.Close()

    results <- result{
        url:  url,
        isOK: resp.StatusCode == http.StatusOK,
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()  
    urlsCh := make(chan string)
    results := make(chan result)    
    var wg sync.WaitGroup   

    go func() {
        defer close(urlsCh) 
        for _, url := range []string{
            "https://go.dev",
            "https://example.com",
            "https://httpstat.us/200",
        } {
            select {
            case urlsCh <- url:
            case <-ctx.Done():
                return
            }
        }
    }()

    go func() {
        for {
            select {
            case <-ctx.Done():
                wg.Wait()
                close(results)
                return  
            case url, ok := <-urlsCh:
                if !ok {
                    wg.Wait()
                    close(results)
                    return
                }

                wg.Add(1)
                go func(url string) {
                    defer wg.Done()
                    checkURL(ctx, url, results)
                }(url)
            }
        }
    }() 
    
    okCount := 0    
    for res := range results {
        if res.err != nil {
            if errors.Is(res.err, context.Canceled) {
                fmt.Println(res.url, "- context canceled")
            } else {
                fmt.Println(res.url, "- not ok")
            }
            fmt.Println(res.url, "- request done")
            continue
        }   
        if res.isOK {
            fmt.Println(res.url, "- ok")
            okCount++   
            if okCount == 2 {
                fmt.Println("Получены два ответа 200 OK, отменяем остальные запросы")
                cancel()
            }
        } else {
            fmt.Println(res.url, "- not ok")
        }   
        fmt.Println(res.url, "- request done")
    }
}
```

Здесь важно отметить обработку ctx.Done.

Для первой горутины:

```go
    go func() {
        defer close(urlsCh) 
        for _, url := range []string{
            "https://go.dev",
            "https://example.com",
            "https://httpstat.us/200",
        } {
            select {
            case urlsCh <- url:
            case <-ctx.Done():
                return
            }
        }
    }()
```

На каждом URL ждем одно из событий:

- адрес успешно отправлен в urlsCh
- сработала отмена через cancel() — тогда выходим из горутины

При выходе отработает отложенный close(urlsCh)

Вторая горутина:

```go
    go func() {
        for {
            select {
            case <-ctx.Done():
                wg.Wait()
                close(results)
                return  
            case url, ok := <-urlsCh:
                if !ok {
                    wg.Wait()
                    close(results)
                    return
                }

                wg.Add(1)
                go func(url string) {
                    defer wg.Done()
                    checkURL(ctx, url, results)
                }(url)
            }
        }
    }() 
```

Одновременно ждет:

- отмену контекста
- следующий URL из канала

Если произошла отмена контекста, то больше новые запросы не запускаем, но ждем завершения всех уже созданных запросов.
Если нет, то запускаем проверку url-ов, но в случае, если !ok (канал урлов закрылся) - также ждем завершения всех уже созданных запросов.
