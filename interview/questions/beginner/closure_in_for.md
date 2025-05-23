# Closure in For

## Условие

Что выведен на экран следующая программа и почему?

```go
package main

func main() {
    first := []int{10, 20, 30, 40}
    second := make([]*int, len(first))
    
    for i, v := range first {
        second[i] = &v
    }
    
    fmt.Println(*second[0], *second[1])
}
```

## Решение

Давайте по строчно разберем то, что происходит в программе:

```go
package main

func main() {
    first := []int{10, 20, 30, 40}
    second := make([]*int, len(first))
    
    for i, v := range first {
        second[i] = &v
    }
    
    fmt.Println(*second[0], *second[1])
}
```

Разберем по шагам:

1. first := []int{10, 20, 30, 40}

    Создается слайс с len 4, cap 4. В нем будут перечисленные элементы.

2. second := make([]*int, len(first))

    Создается слайс для указателей на int-ы, len 4, cap 4.

3. for i, v := range first

    При инициализации переменных в операторах (как например for) под i и v создается область памяти и работа идет с этой областью пока мы не закончится наш оператор (в нашем случае for).

    Т.е. при итерации в i и в v будут копироваться элементы (индекс в массиве и значение).

4. second[i] = &v

    Таким образом в second будет записываться одна и та же область памяти, выделенная для v.

5. fmt.Println(*second[0], *second[1])

    Так как все элементы в second указывают на область памяти, где был записан последний элемент слайса (перед выходом из for), то вывод будет:

    ```text
    40 40
    ```

### Go 1.22

В версии Go 1.22 было введено изменение, которое решает проблему с переиспользованием переменной цикла v в конструкции for range. С выходом этой версии и далее будет создаваться новая переменная на каждой итерации цикла, что предотвращает ошибку, когда все указатели в замыканиях или срезах указывают на одно и то же место в памяти.

Т.е. неявно будет что-то подобное:

```go
    for i, v := range first {
        v := v
        second[i] = &v
    }
```

И вывод будет уже ожидаемый:

```text
10 20
```
