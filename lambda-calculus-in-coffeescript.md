# λ-Calculus in CoffeeScript - Part (1)
---

*Warning.* The things discussed in this article have very little "real world" applications.



**Task.** Using λ-calculus, calculate the first 10 prime numbers.


First let's define some numbers

```
ZERO = (f) -> (x) -> x
 
```
Ok. Thats one number. But we can do better. Define the *successor function* (adds 1 to its input).

```
SUCC = (n) -> (f) -> (x) -> f(n(f) x)
```
This gives us the abbility to define more numbers

```
ONE = SUCC ZERO
TWO = SUCC ONE
```







---
Resources:

[1] [Wikipedia: Lambda-Calculus](http://en.wikipedia.org/wiki/Lambda_calculus)
