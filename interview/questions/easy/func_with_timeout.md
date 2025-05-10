# Вызов функции с таймаутом

## Условие

Представлен следующий кусок кода:

```go
package main

import (
    "fmt"
    "net/http"
)

func getDiscount() float64 {
    discount, _ := http.Get("url")

    return discount
}

func main() {
    fmt.Printf("Ваша скидка: %v", getDiscount())
}
```

Вызов стороннего API по получению скидки непредсказуем - это может быть как долго (вплоть до зависания), так и быстро.

Добавьте возможность вызывать стороннее API с таймаутом.

## Решение

Для подобных задач необходимо ввести контекст вызова:

```go
package main

import (
    "fmt"
    "net/http"
    "context"
    "time"
)

func main() {
    ctx := context.Background()
    
    res, err := getDiscountWithContextTimeout(ctx, 2 * time.Second)
    if err != nil {
        fmt.Println("Ошибка вызова", err.Error())
        return
    }
    fmt.Printf("Ваша скидка: %v", res)
}

func getDiscount() float64 {
    discount, _ := http.Get("url")

    // some work

    return discount
}

func getDiscountWithContextTimeout(ctx context.Context, timeout time.Duration) (float64, error) {
    ctx, cancel := context.WithTimeout(ctx, timeout)
    defer cancel()

    ch := make(chan float64)

    go func() {
        ch <- getDiscount()
        close(ch)
    }()

    select {
        case <- ctx.Done():
            return 0.0, ctx.Err()
        case res := <- ch:
            return res, nil
    }
}
```
