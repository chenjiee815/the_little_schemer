===============
 Numbers Games
===============

.. contents::


本章节概要：
::

   主要引入了数字，以及数字列表tup，

   围绕着上面两个概念，定义了相关的一系列操作，并用递归实现之。

----------------------------------------

基本概念
========
* 数字也是一个atom

* number?: 判断一个S表达式是否为数字

* add1: 对数字加1

  .. code-block:: scheme

     (define add1 1+)

* sub1: 对数字减1

  .. code-block:: scheme

     (define sub1 1-)

* zero?: 判断一个数字是否为0

* tup: 列表中包含的每个s表达式都是数字的列表。

.. note::

   书中为了简化概念，不考虑负数的情况。

\+
===
将两个数字相加

.. code-block:: scheme

   ; +; number number -> number
   (define +
     (lambda (a b)
       (cond
        ((zero? b) a)
        (else (add1 (+ a (sub1 b)))))))

   ; 另外一个版本
   (define +
     (lambda (a b)
       (cond
        ((zero? b) a)
        (else (+ (add1 a) (sub1 b))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
    
   (define +&co
     (lambda (a b col)
       (cond
        ((zero? b) (col a))
        (else (+&co a
                    (sub1 b)
                    (lambda (num)
                       (col (add1 num))))))))

   ; 另外一个版本
   (define +&co
     (lambda (a b col)
       (cond
        ((zero? b) (col a))
        (else (+&co (add1 a)
                        (sub1 b)
                        col)))))

\-
===
将两个数字相减

.. note::

   不考虑相减的结果为负数的情况

.. code-block:: scheme

   ; -: number number -> number
   (define -
     (lambda (a b)
       (cond
        ((zero? b) a)
        (else (sub1 (- a (sub1 b)))))))

   ; 另一个版本
   (define -
     (lambda (a b)
       (cond
        ((zero? b) a)
        (else (- (sub1 a) (sub1 b))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
    
   (define -&co
     (lambda (a b col)
       (cond
        ((zero? b) (col a))
        (else (-&co a
                    (sub1 b)
                    (lambda (num)
                      (col (sub1 num))))))))

   ; 另一个版本
   (define -&co
     (lambda (a b col)
       (cond
        ((zero? b) (col a))
        (else (-&co (sub1 a) (sub1 b) col)))))

addtup
======
将一个tup中的所有数字相加

.. code-block:: scheme

   ; addtup: tup -> number
   (define addtup
     (lambda (tup)
       (cond
        ((null? tup) 0)
        (else (+ (car tup) (addtup (cdr tup)))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
    
   (define addtup&co
     (lambda (tup col)
       (cond
        ((null? tup) (col 0))
        (else
         (addtup&co (cdr tup)
                    (lambda (sum)
                      (col (+ sum (car tup)))))))))

\*
===
将两个数字相乘

.. code-block:: scheme
    
   ; *: number number -> number
   (define *
     (lambda (n m)
       (cond
        ((zero? m) 0)
        (else (+ n (* n (sub1 m)))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
    
   (define *&co
     (lambda (n m col)
       (cond
        ((zero? m) (col 0))
        (else (*&co n
                    (sub1 m)
                    (lambda (num)
                       (col (+ num n))))))))

tup+
====
将两个tup中相对的数字相加,然后返回相加后的tup

.. code-block:: scheme

   ; tup1+: tup tup -> tup
   ; 该函数允许两个参数的长度不一样
   (define tup1+
     (lambda (tup1 tup2)
       (cond
        ((null? tup2) tup1)
        ((null? tup1) tup2)
        (else (cons (+ (car tup1) (car tup2))
                    (tup+ (cdr tup1) (cdr tup2)))))))

   ; tup2+: tup tup -> tup
   ; 该函数的两个参数长度必须一样
   (define tup2+
     (lambda (tup1 tup2)
       (cond
        ((and (null? tup1) (null? tup2)) '())
        (else (cons (+ (car tup1) (car tup2))
                    (tup+ (cdr tup1) (cdr tup2)))))))

.. code-block:: scheme
    
   (define col
     (lambda (newtup)
       (display newtup)
       (newline)
       newtup))
    
   (define tup+&co
     (lambda (tup1 tup2 col)
       (cond
        ((null? tup1) (col tup2))
        ((null? tup2) (col tup1))
        (else (tup+&co (cdr tup1)
                       (cdr tup2)
                       (lambda (newtup)
                         (col (cons (+ (car tup1)
                                       (car tup2))
                                    newtup))))))))

<
===
比较两个数字的大小 

.. note::

   < 函数未考虑两个数字都为0的情况

.. code-block:: scheme

   ; >: number number -> bool
   (define >
     (lambda (n m)
       (cond
        ((zero? n) #f)
        ((zero? m) #t)
        (else (> (sub1 n) (sub1 m))))))

.. code-block:: scheme
  
   (define col
     (lambda (bflag)
       (display bflag)
       (newline)
       bflag))
  
   (define >&co
     (lambda (n m col)
       (cond
        ((zero? n) (col #f))
        ((zero? m) (col #t))
        (else (>&co (- n 1) (- m 1) col)))))

\>
===
比较两个数字的大小

.. note::

   < 函数未考虑两个数字都为0的情况

.. code-block:: scheme

   ; <: number number -> bool
   (define <
     (lambda (n m)
       (cond
        ((zero? m) #f)
        ((zero? n) #t)
        (else (< (sub1 n) (sub1 m))))))

.. code-block:: scheme
  
   (define col
     (lambda (bflag)
       (display bflag)
       (newline)
       bflag))
  
   (define <&co
     (lambda (n m col)
       (cond
        ((zero? m) (col #f))
        ((zero? n) (col #t))
        (else (<&co (sub1 n) (sub1 m) col)))))

\=
===
比较两个数字是否相等

.. code-block:: scheme

   ; =: number number -> bool
   (define =
     (lambda (n m)
       (cond
        ((zero? m) (zero? n))
        ((zero? n) #f)
        (else (= (sub1 n) (sub1 m))))))

   ; 另一版本
   (define =
     (lambda (n m)
       (cond
        ((or (> n m) (< n m)) #f)
        (else #t))))

.. code-block:: scheme

   (define col
     (lambda (bflag)
       (display bflag)
       (newline)
       bflag))

   (define =&co
     (lambda (n m)
       (cond
        ((zero? m) (col (zero? n)))
        ((zero? n) (col #f))
        (else (=&co (sub1 n) (sub1 m) col)))))

\^
===
阶乘

.. code-block:: scheme

   ; ^: number number -> number
   (define ^
     (lambda (n m)
       (cond
        ((zero? n) 0)
        ((zero? m) 1)
        (else (* n (^ n (sub1 m)))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
   
   (define ^&co
     (lambda (n m col)
       (cond
        ((zero? n) (col 0))
        ((zero? m) (col 1))
        (else (^&co n
                    (sub1 m)
                    (lambda (num)
                      (col (* n num))))))))

\/
===
除

.. code-block:: scheme

   ; /: number number -> number
   (define /
     (lambda (n m)
       (cond
        ((< n m) 0)
        (else (add1 (/ (- n m) m))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
    
   (define /&co
     (lambda (n m col)
       (cond
        ((< n m) (col 0))
        (else (/&co (- n m)
                    m
                    (lambda (num)
                      (col (add1 num))))))))

length
======
返回一个lat的长度

.. code-block:: scheme

   ; length: lat -> number
   (define length
     (lambda (lat)
       (cond
        ((null? lat) 0)
        (else (add1 (length (cdr lat)))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
    
   (define length&co
     (lambda (lat col)
       (cond
        ((null? lat) (col 0))
        (else (length&co (cdr lat)
                         (lambda (num)
                           (col (add1 num))))))))

pick
====
根据传入的数字,获取其对应在lat中位置的S表达式

.. code-block:: scheme

   ; pick: number lat -> symbol
   (define pick
     (lambda (n lat)
       (cond
        ((zero? (sub1 n)) (car lat))
        (else (pick (sub1 n) (cdr lat))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
    
   (define pick&co
     (lambda (n lat col)
       (cond
        ((null? lat) (col '()))
        ((zero? (sub1 n)) (col (car lat)))
        (else (pick&co (sub1 n) (cdr lat) col)))))

rempick
=======
根据传入的数字, 删除其对应在lat中位置的S表达式,并返回剩余列表

.. code-block:: scheme

   ; rempick: number lat -> lat
   (define rempick
     (lambda (n lat)
       (cond
        ((null? lat) '())
        ((zero? (sub1 n)) (cdr lat))
        (else (cons (car lat)
                    (rempick (sub1 n) (cdr lat)))))))

.. code-block:: scheme

   (define col
     (lambda (lat)
       (display lat)
       (newline)
       lat))
    
   (define rempick&co
     (lambda (n lat col)
       (cond
        ((null? lat) (col '()))
        ((zero? (sub1 n)) (col (cdr lat)))
        (else (rempick&co (sub1 n)
                          (cdr lat)
                          (lambda (newlat)
                            (col (cons (car lat)
                                       newlat))))))))

no-nums
=======
选出列表中的非数字S表达式, 并以列表形式返回

.. code-block:: scheme

   ; no-nums: lat -> lat
   (define no-nums
     (lambda (lat)
       (cond
        ((null? lat) '())
        ((number? (car lat)) (no-nums (cdr lat)))
        (else (cons (car lat)
                    (no-nums (cdr lat)))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
    
   (define no-nums&co
     (lambda (lat col)
       (cond
        ((null? lat) (col '()))
        ((number? (car lat))
         (no-nums&co (cdr lat) col))
        (else
         (no-nums&co (cdr lat)
                     (lambda (newlat)
                       (col (cons (car lat) newlat))))))))

all-nums
========
跟上面相反, 只返回数字列表

.. code-block:: scheme

   ; all-nums: lat -> lat
   (define all-nums
     (lambda (lat)
       (cond
        ((null? lat) '())
        ((not (number? (car lat))) (all-nums (cdr lat)))
        (else (cons (car lat) (all-nums (cdr lat)))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
    
   (define all-nums&co
     (lambda (lat col)
       (cond
        ((null? lat) (col '()))
        ((not (number? (car lat)))
         (all-nums&co (cdr lat) col))
        (else
         (all-nums&co (cdr lat)
                      (lambda (newlat)
                        (col (cons (car lat) newlat))))))))

.. tip::

   `no-nums` 和 `all-nums` 的逻辑基本一样，

   其实可以将它们的共同点抽出来。

   .. code-block:: scheme
       
      (define col
        (lambda (no-nums all-nums)
          (dispaly no-nums)
          (newline)
          (display all-nums)))
       
      (define split-nums&co
        (lambda (lat col)
          (cond
           ((null? lat) (col '() '()))
           ((number? (car lat))
            (split-nums&co (cdr lat)
                           (lambda (no all)
                             (col no (cons (car lat) all)))))
           (else
            (split-nums&co (cdr lat)
                           (lambda (no all)
                             (col (cons (car lat) no) all)))))))

eqan?
=====
比较两个S表达式是否是相等

.. tip::

   实现该函数是为了扩展 `eq?` 的功能，后面会用到该函数

.. code-block:: scheme

   ; eqan? symbol symbol -> bool
   (define eqan?
     (lambda (a1 a2)
       (cond
        ((and (number? a1) (number? a2))
         (= a1 a2))
        ((or (number? a1) (number? a2))
         #f)
        (else (eq? a1 a2)))))

occur?
======
检查列表中有几个指定的S表达式

.. code-block:: scheme

   ; occur: symbol lat -> number
   (define occur
     (lambda (a lat)
       (cond
        ((null? lat) 0)
        ((eqan? a (car lat))
         (add1 (occur a (cdr lat))))
        (else (occur a (cdr lat))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
    
   (define occur&co
     (lambda (n lat col)
       (cond
        ((null? lat) (col 0))
        ((eq? n (car lat))
         (occur&co n
                   (cdr lat)
                   (lambda (num)
                     (col (add1 num)))))
        (else
         (occur&co n
                   (cdr lat)
                   col)))))

one?
====
判断一个数字是否为1

.. tip::

   写这个函数，应该是稍微提一下Scheme中的第八戒。

.. code-block:: scheme

   ; one?: number -> bool
   (define one?
     (lambda (n)
       (cond
        ((zero? n) #f)
        (else (zero? (sub1 n))))))

   ; 另一版本
   (define one?
     (lambda (n)
       (cond
        (else (= n 1)))))

通过 `one?` 咱们就可以简写 `rempick` 函数了。

.. code-block:: scheme

   ; rempick: number lat -> lat
   (define rempick
     (lambda (n lat)
       (cond
        ((null? lat) '())
        ((one? n) (cdr lat))
        (else (cons (car lat)
                    (rempick (sub1 n)
                             (cdr lat)))))))
