Язык программирования:
* Элементарные выражения
* Средства комбинации
* Средства абстракции

Подстановочная модель вычисления: 
* Нормальная - заменять переменные на выражения, пока не получится выражение, состоящие только из элементарных операторов, затем редуцировать
* Апликативная - вычислять аргументы и применять к ним оператор


Задача 1.7
```
; Исходник программы вычисления квадратного корня
(define (sqrt-iter guess x) 
	(if (good-enough? guess x) 
		guess
		(sqrt-iter (improve guess x) x))
)

(define (improve guess x) 
	(average guess (/ x guess)))

(define (average x y) (/ (+ x y) 2))

(define (good-enough? guess x) (< (abs (- (* guess guess) x)) 0.0001))


(define (sqrt x) 
	(sqrt-iter 1.0 x))
```
```
;Мое решение
(define (sqrt-iter guess p x) 
	(if (good-enough? guess p x)
		guess
		(sqrt-iter (improve guess x) guess x))
)

(define (good-enough? guess p) (< (/ (abs (- guess p)) guess) 0.0001))

(define (sqrt x) 
	(sqrt-iter 1.0 x x))

(sqrt 9)
```

Задача 1.8
```
(define (iter guess x) 
	(if (good-enough? guess x) 
	guess
	(iter (improve guess x) x))
)

(define (improve y x) (/ (+ (/ x (* y y)) (* 2 y)) 3))

(define (good-enough? guess x) (< (abs (- (* guess guess guess) x)) 0.0001))

(define (cube x) (iter 1.0 x))

(cube 27)
```

## Внутренние определения
```
(define (sqrt x) 
	(define (good-enough? guess x) 
		(< (abs (- (square guess) x)) 0.001))
	(define (improve guess x)
		(average guess (/ x guess)))
	(define (sqrt-iter guess x)
		(if (good-enough? guess x)
			guess
			(sqrt-iter (improve guess x) x)))
	(sqrt-iter 1.0 x))
```
Так как `x` уже связан (bind) в определении процедуры sqrt, можно воспользоваться этим значением и убрать связывание во внутренних переменных.

```
(define (sqrt x) 
	(define (good-enough? guess) 
		(< (abs (- (square guess) x)) 0.001))
	(define (improve guess)
		(average guess (/ x guess)))
	(define (sqrt-iter guess)
		(if (good-enough? guess x)
			guess
			(sqrt-iter (improve guess x) x)))
	(sqrt-iter 1.0 x))
```

## Вычисление факториала

**Рекурсивно**
```
(define (factorial n) 
		(if (= n 1)
			1
			(* n (factorial (- n 1)))
		)
)
```

**Итеративно** 
```
(define (factorial n)
	(define (iter counter product)
		(if (> counter n) 
			product
			(iter (+ counter 1) (* counter product))))
	(iter 1 1))
```
Хвостовая рекурсия - особый вид рекурсии при котором любой вызов рекурсивной функции является последним. Такую рекурсию можно легко заменить итерацией.
```python
def fact_iter(n, product):
	if n == 0:
		return product
	else:
		return fact_iter(n-1, product*n)

def factorial(n):
	return fact_iter(n, 1)

# fact_iter можно заменить на цикл
while n != 0:
	product *= n
	n -= 1

def factorial(n):
	'''
	Эту функцию нельзя заменить на цикл, потому что последней операцией здесь является умножение
	'''
	if n == 0:
		return 1

	else:
		return n * factorial(n-1)
```