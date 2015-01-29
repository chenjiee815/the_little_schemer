=========
 Shadows
=========

.. contents::

本章概要：
::

   本章节主要两个方面：

   1. 一个计算器的实现（计算算术表达式的值），

      为最后一章节讲解Scheme解释器打基础。

   2. church encoding，使用列表来表示数字，

      然后再参照第四章，根据原语函数定义加、减等函数。

----------------------------------------

numbered?
=========
判断一个S表达式是否为算术表达式

.. code-block:: scheme

   ; numbered?: symbol -> bool
   (define numbered?
     (lambda (aexp)
       (cond
        ((atom? aexp) (number? aexp))
        ((or (eq? (car (cdr aexp)) '+)
             (eq? (car (cdr aexp)) '*)
             (eq? (car (cdr aexp)) '^))
         (and (numberd? (car aexp))
              (numberd? (car (cdr (cdr aexp))))))
        (else #f))))

value
=====
获取一个算术表达式的值

.. tip::

   ; 在Guile环境，请添加如下代码
   (define ^ expt)

.. code-block:: scheme
    
   (define value
     (lambda (nexp)
       (cond
        ((atom? nexp) nexp)
        ((eq? (car nexp) '+)
         (+ (value (cdr nexp))
            (value (cdr (cdr nexp)))))
        ((eq? (car nexp) '*)
         (* (value (cdr nexp))
            (value (cdr (cdr nexp)))))
        (else (^ (value (cdr nexp))
                 (value (cdr (cdr nexp))))))))

.. code-block:: scheme
   
   ; 帮助函数
   (define 1st-sub-exp
     (lambda (aexp)
       (car (cdr aexp))))
   
   ; 帮助函数
   (define 2nd-sub-exp
     (lambda (aexp)
       (car (cdr (cdr aexp)))))
   
   ; 帮助函数
   (define operator
     (lambda (aexp)
       (car aexp)))
   
   (define value
     (lambda (nexp)
       (cond
        ((atom? nexp) nexp)
        ((eq? (operator nexp) '+)
         (+ (value (1st-sub-exp nexp))
            (value (2nd-sub-exp nexp))))
        ((eq? (operator nexp) '*)
         (* (value (1st-sub-exp nexp))
            (value (2nd-sub-exp))))
        (else (^ (value (1st-sub-exp nexp))
                 (value (2nd-sub-exp nexp)))))))

使用帮助函数重写的value函数，\
其实该版本的value函数可以用在前缀、中缀、后缀表达式上，\
只要修改三个帮助函数即可。

church encoding
===============
翻译为：邱奇数，关于它的定义详见 `Wiki 邱奇数`_

作者用 `()` 来表示0, `(())` 表示1, `(() ())` 表示2... ...

然后定义了对应的原语函数: sero?，edd1，zub1。

1. sero?: 对应zero?

   .. code-block:: scheme

      (define sero?
        (lambda (n)
          (null? n)))

2. edd1: 对应add1

   .. code-block:: scheme
   
      (define edd1
        (lambda (n)
          (cons '() n)))

3. zub1: 对应sub1

   .. code-block:: scheme
   
      (define zub1
        (lambda (n)
          (cdr n)))

和第四章一样，根据上面三个原语函数，然后再来定义加、减等这些函数。

.. code-block:: scheme

   (define +
     (lambda (n m)
       (cond
        ((sero? m) n)
        (else (edd1 (+ n (zub1 m)))))))

.. code-block:: scheme

   (define -
     (lambda (n m)
       (cond
        ((sero? m) n)
        (else (- (zub1 n) (zub1 m))))))

.. _`Wiki 邱奇数`: http://zh.wikipedia.org/wiki/%E9%82%B1%E5%A5%87%E6%95%B0
