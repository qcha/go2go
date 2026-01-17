# Reading from close channel

## Условие

Что выведет следующий код:

```go
import "runtime"

func main() {
    ch := make(chan string)
    close(ch)
    go func() {
        text := <-ch
        println("Hello, ", text)
    }()
    runtime.GC()
}
```

## Решение

Здесь мы создаем канал chan string и по сути тут же его закрываем. Писать в такой канал уже ничего нелья, но читать - можно и будет возвращено zero value.
Для строк zero value - это пустая строка.

Поэтому, при чтении из пустого канала мы получим пустую строку и вывод будет:

```go
Hello, 
```
