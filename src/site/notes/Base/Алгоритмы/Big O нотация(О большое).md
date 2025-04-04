---
{"dg-publish":true,"permalink":"/base/algoritmy/big-o-notacziya-o-bolshoe/"}
---



**Big O **– это мера эффективности «в худшем случае», верхняя граница того, сколько времени потребуется для выполнения задачи.

O(n) - количество операций, которые необходимо выполнить алгоритму.

К примеру поиск числа обычным поиском(перебор по порядку) выполняется за время O(n), а поиск числа бинарным поиском за O(log<sub>2</sub><sup>n</sup>). Подставив вместо n размер исследуемой области, мы найдём количество операций в худшем случае.

При простом поиске необходимо будет пролистать всю книгу, чтобы найти номер O(n).
n - количество записей в книге, к примеру 100 O(100) = 100 операций, при бинарном поиске 
O(log<sub>2</sub><sup>100</sup>) = 7 операций.

### Скорость алгоритмов измеряется не в секундах, а в темпе роста количества операций.

**О большее игнорирует числа задействованные в операциях сложения, вычитания, умножение, деления.**

1. O(n+2321)=O(n)
2. O(n/323)=O(n)

**Если в O есть сумма, нас интересует самое быстрорастущее слагаемое.  
Примеры: ** 
1. O(n^2 + n) = O(n^2)
2. O(n^3 + 100n * log n) = O(n^3)
3. O(n! + 999) = O(n!)
4. O(1,1^n + n^100) = O(1,1^n)
  
В О-нотации на операции с одной или двумя переменными вроде i++, a * b, a / 1024, max(a,b) уходит всего одна единица времени.  
  
**Константы откидываются.** Нас интересует только часть формулы, которая зависит от размера входных данных. Проще говоря, это само число n, его степени, логарифмы, факториалы и экспоненты, где число находится в степени n.  
Примеры:  
1. O(3n) = O(n)
2. O(10000 n^2) = O(n^2)
3. O(2n * log n) = O(n * log n)
