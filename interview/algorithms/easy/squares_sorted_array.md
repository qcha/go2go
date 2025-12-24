# Squares of sorted array

Задача с [Leetcode#977](https://leetcode.com/problems/squares-of-a-sorted-array/description/)

## Условие

Дан целочисленный массив nums, отсортированный в порядке возрастания. Верните массив квадратов каждого числа, также отсортированный в порядке возрастания.

```go
Input:  [-4, -1, 0, 3, 10]
Output: [0, 1, 9, 16, 100]

Input:  [-7, -3, 2, 3, 11]
Output: [4, 9, 9, 49, 121]
```

## Решение

### В лоб

Решение в лоб - это просто пройтись по массиву, возвести все в квадрад и после этого отсортировать встроенной сортировкой.

```go
func sortedSquaresNlogN(nums []int) []int {
	for i, x := range nums {
		nums[i] = x * x
	}

	slices.Sort(nums)

	return nums
}
```

Если мы поступим так, то сложность будет: O(N) + O(N * log(N)) = O(N * log(N))

### Два указателя

Заведем два указателя на начало и на конец массива. И результирующий массив, в который будем класть значения.

После этого заведем индекс idx и будем проходить справа налево массив. Сверять будем два числа по нашим указателям, по модулю.
Если то, на что указывает левый указатель, по модулю больше, чем число по правому указателю - то помещаем в результирующий массив квадрат левого числа, сдвигаем указатель вправо. Если нет - то помещаем квадрат правого и указатель сдвигаем влево.

```go
func sortedSquares(nums []int) []int {
	left := 0
	right := len(nums) - 1

	result := make([]int, len(nums))
	for idx := len(nums) - 1; idx >= 0; idx-- {
		lhs := abs(nums[left])
		rhs := abs(nums[right])

		if lhs >= rhs {
			result[idx] = lhs * lhs
			left++
		} else {
			result[idx] = rhs * rhs
			right--
		}
	}

	return result
}

func abs(num int) int {
	if num < 0 {
		return -num
	}

	return num
}
```

Сложность: O(N)
