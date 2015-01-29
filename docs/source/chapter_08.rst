=====================
 Lambda the Ultimate
=====================

.. contents::

本章概要：
::

   这一章才算开始高能, 下面的九,十章则更是要下一翻功夫了.

   说高能, 并不是指有多难(除了连续概念的讲解),

   而是指这一章揭示了很多更深入的东西, 更深入的抽象：

   **高阶函数** 与 **CPS变换** 。

----------------------------------------

rember-f
========
基本同rember，但是其中的eq/equal比较函数当作参数传入进来。

这样具体的比较操作就抽象比来，可以用来支持各种对象/类型的删除操作，\
只需要你定义好其对象/类型的eq/equal比较函数即可。

.. code-block:: scheme

   ; rember-f: function symbol list -> list
   (define rember-f
     (lambda (test? a l)
       (cond
        ((null? l) '())
        ((test? a (car l)) (cdr l))
        (else (cons (car l)
                    (rember-f (test? a (cdr l))))))))
   
简单修改一下，将之定义成为一个高阶函数，\
即返回函数的函数。

.. code-block:: scheme

   (define rember-f
     (lambda (test?)
       (lambda (a l)
         (cond
          ((null? l) '())
          ((test? a (car l)) (cdr l))
          (else (cons (car l)
                      (rember-f (test? a (cdr l)))))))))
   
   (define rember-eq?
     (lambda (a l)
       (rember-f eq?)))
   
   (define rember-equal?
     (lambda (a l)
       (rember-f equal?)))

eq?-c
=====
返回一个函数, 用来与固定S表达式比较.

.. code-block:: scheme
    
   (define eq?-c
     (lambda (a)
       (lambda (x)
         (eq? x a))))
   
   ; 与salad比较
   (define eq?-salad
   (eq?-c 'salad))

insert-q
========
返回一个函数, 具体的操作函数当作参数传入

.. tip::

   其实在第3章，细心的话，\
   会发现 `insertL` 、 `insertR` 、 `subst` 、 `rember` 的逻辑基本是一样的，\
   只是各别的代码不一样，那么我们可以通过高阶函数来将公共的代码抽出来。

.. code-block:: scheme

   (define insert-g
     (lambda (seq)
       (lambda (new old l)
         (cond
          ((null? l) '())
          ((eq? (car l) old)
           (seq new old (cdr l)))
          (else (cons (car l)
                      ((insert-g seq) new old (cdr l))))))))

.. code-block:: scheme
    
   (define seqL
     (lambda (new old l)
       (cons new (cons old l))))

   (define insertL
     (insert-g seqL))

.. code-block:: scheme
    
   (define seqR
     (lambda (new old l)
       (cons old (cons new l))))

   (define insertR
     (insert-g seqR))

.. code-block:: scheme
   
   (define seqS
     (lambda (new old l)
       (cons new l)))

   (define subst
     (insert-g seqS))

.. code-block:: scheme

   (define seqrem
     (lambda (new old l)
       l))

   (define rember
     (lambda (a l)
       ((insert-g seqrem) #f a l)))

value
=====
重写之前的value, 将里面的操作抽象出来

.. code-block:: scheme
    
   (define atom-to-function
     (lambda (x)
       (cond
        ((eq? x '+) +)
        ((eq? x '*) *)
        (else ^))))

.. code-block:: scheme

   (define value
     (lambda (nexp)
       (cond
        ((atom? nexp) nexp)
        (else
         ((atom-to-function (operator nexp))
          (value (1st-sub-exp nexp))
          (value (2nd-sub-exp nexp)))))))

multirember-f
=============

.. code-block:: scheme
    
   (define multirember-f
     (lambda (test?)
       (lambda (a lat)
         (cond
          ((null? lat) '())
          ((test? a (car lat))
           ((multirember-f test?) a (cdr lat)))
          (else (cons (car lat)
                      ((multirember-f test?) a (cdr lat))))))))

.. code-block:: scheme
    
   (define multirember-eq?
     (multirember-f eq?))

.. code-block:: scheme
    
   (define multirember-equal?
     (multirember-f equal?))

.. warning::

   下面开始介绍到连续概念(continuation)

multiremberT
============
基本同上，\
不过test?可以带参数，\
将每次递归都不会变化的test?和a参数都存放到test?函数中，\
以后写函数，可以将哪些参数是不变的，哪些参数是变化的区分开来。

.. code-block:: scheme
    
   (define multiremberT
     (lambda (test? lat)
       (cond
        ((null? lat) '())
        ((test? (car lat))
         (multiremberT test? (cdr lat)))
        (else (cons (car lat)
                    (multiremberT test? (cdr lat)))))))

multirember&co
==============
添加一个连续函数col，multirember&co函数的最后一步就是调用col。

将具体的操作放入col中，其中的col相当于一个收集器(collector)，\
它将lat中和a参数不相同的S表达式放入col的第一个参数中，\
相同的放入第二个参数中。

.. code-block:: scheme
    
   (define multirember&co
     (lambda (a lat col)
       (cond
        ((null? lat) (col '() '()))
        ((eq? (car lat) a)
         (multirember&co a
                         (cdr lat)
                         (lambda (newlat seen)
                           (col newlat
                                (cons (car lat) seen)))))
        (else
         (multirember&co a
                         (cdr lat)
                         (lambda (newlat seen)
                           (col (cons (car lat) newlat)
                                seen)))))))

.. code-block:: scheme
    
   (define a-friend
     (lambda (x y)
       (null? y)))

如果col为a-friend，那么multirember&co的意思就是：
::

   memeber?

.. code-block:: scheme

   (define last-friend
     (lambda (x y)
       (length x)))

如果col为last-friend，那么multirember&co的意思就是：
::

   lat中有多少个与a不同的S表达式

multiinsertLR
=============
将new插入到oldL的左边,oldR的右边.

.. code-block:: scheme

   (define multiinsertLR
     (lambda (new oldL oldR lat)
       (cond
        ((null? lat) '())
        ((eq? oldL (car lat))
         (cons new
               (cons oldL
                     (multiinsertLR new
                                    oldL
                                    oldR
                                    (cdr lat)))))
        ((eq? oldR (car lat))
         (cons oldR
               (cons new
                     (multiinsertLR new
                                    oldL
                                    oldR
                                    (cdr lat)))))
        (else
         (cons (car lat)
               (multiinsertLR new
                              oldL
                              oldR
                              (cdr lat)))))))

multiinsertLR&co
================
col的newlat参数存放最后插入new参数后的newlat，\
L参数是在oldL参数左边插入的次数, R参数是在oldR参数右边插入的次数。

.. code-block:: scheme
    
   (define multiinsertLR&co
     (lambda (new oldL oldR lat col)
       (cond
        ((null? lat)
         (col '() 0 0))
        ((eq? oldL (car lat))
         (multiinsertLR&co new
                           oldL
                           oldR
                           (cdr lat)
                           (lambda (newlat L R)
                             (col (cons new (cons oldL newlat))
                                  (add1 L)
                                  R))))
        ((eq? oldR (car lat))
         (multiinsertLR&co new
                           oldL
                           oldR
                           (cdr lat)
                           (lambda (newlat L R)
                             (col (cons oldR (cons new newlat))
                                  L
                                  (add1 R)))))
        (else
         (multiinsertLR&co new
                           oldL
                           oldR
                           (cdr lat)
                           (lambda (newlat L R)
                             (col (car lat)
                                  L
                                  R)))))))

evens-only*
===========
找出嵌套队列中所有的偶数

.. code-block:: scheme

   ; 此处要用之前定义的运算符号,用系统自带的会出错
   ; lisp支持分数, 即3/2不缺失其精度
   (define even?
     (lambda (n)
       (= (* (/ n 2) 2) n)))

.. code-block:: scheme
    
   (define evens-only*
     (lambda (l)
       (cond
        ((null? l) '())
        ((atom? (car l))
         (cond
           ((even? (car l))
            (cons (car l)
                  (evens-only* (cdr l))))
           (else
            (evens-only* (cdr l)))))
        (else
         (cons (evens-only* (car l))
               (evens-only* (cdr l)))))))

evens-only*&co
==============
col函数的第一个参数表示l列表中所有的偶数列表，\
第二个参数表示所有偶数之积，\
第三个参数表示l列表中所有非偶数之和。

.. code-block:: scheme

   (define evens-only*&co
     (lambda (l col)
       (cond
        ((null? l) (col '() 1 0))
        ((atom? (car l))
         (cond
          ((even? (car l))
           (evens-only*&co (cdr l)
                             (lambda (newlat m s)
                               (col (cons (car l) newlat)
                                    (* (car l) m)
                                    s))))
          (else
           (evens-only*&co (cdr l)
                           (lambda (newlat m s)
                             (col newlat
                                  m
                                  (+ (car l) s)))))))
         (else
          (evens-only*&co (car l)
                          (lambda (al am as)
                            (evens-only*&co (cdr l)
                                            (lambda (dl dm ds)
                                              (col (cons al dl)
                                                   (* am dm)
                                                   (+ as ds))))))))))

.. code-block:: scheme

   (define the-last-friend
     (lambda (newl product sum)
       (cons sum (cons product newl))))

如果col为the-last-friend，那么evens-only*&co意思为：
::

   返回一个列表，列表的第一个S表达式为l参数中所有的偶数之积，

   列表的第二个S表达式为l参数中所有偶数之和，

   列表的剩余S表达式为l参数中所有的偶数。

.. tip::

   这里建议一下，最好将从第二章开始的所有递归函数都用\
   cps形式手动重写一遍。

   第一次接触连续的概念不用担心，虽然看起来很难，其实只是\
   你不熟悉而已，按照上面的建议做一遍，就都明白了。
