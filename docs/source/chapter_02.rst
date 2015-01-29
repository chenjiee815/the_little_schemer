===========================================
 Do It, Do It Again, and Again, and Again…
===========================================

.. contents::

本章节概要：
::

   首先介绍了一个基本概念 `lat` ，然后定义了一系列与该概念相关的操作，以及如何通过递归来实现这些操作。

----------------------------------------

基本概念
========
lat

  不包含子列表的列表

lat?
====
判断某个列表是否为 `lat`

.. tip::

   通过判断列表中是否每个S表达式都是原子来实现

.. code-block:: scheme
   
   ; lat?: list -> bool
   (define lat?
     (lambda (l)
       (cond
         ((null? l) #t)
         ((atom? (car l)) (lat? (cdr l)))
         (else #f))))

`lat?` 经过CPS变换后的代码

.. tip::

   建议阅读完第8章，了解CPS变换之后，再回头从2章将每个函数写一个对应的CPS变换。

   用于连续的那个函数基本没有固定的参数和代码，按照自己所想的来。

.. code-block:: scheme

   ;; 添加一个连续函数，参数中为两个列表
   ;; 第一个表示lat列表，第二表示非lat列表
   (define col
     (lambda (lat nolat)
       (null? nolat)))
  
   (define lat?&co
     (lambda (l col)
       (cond
         ((null? l) (col '() '()))
         ((atom? (car l)) (lat?&co (cdr l)
                                   (lambda (lat nolat)
                                     (col (cons (car l) lat) nolat))))
         (else (lat?&co (cdr l)
                        (lambda (lat nolat)
                          (col lat (cons (car l) nolat))))))))

member?
=======
用来判断一个S表达式是否在一个lat之内。

.. code-block:: scheme

   ; member?: symbol lat -> bool
   (define member?
     (lambda (a lat)
       (cond
         ((null? lat) #f)
         ((eq? a (car lat)) #t)
         (else (member? a (cdr lat))))))

.. code-block:: scheme
  
   ; CPS变换
   (define col
     (lambda (in out)
       (not (null? in))))
  
   (define member?&co
     (lambda (a lat col)
       (cond
         ((null? lat) #f)
         ((eq? a (car lat)) (member?&co a
                                        (cdr lat)
                                        (lambda (in out)
                                          (col (cons a in) out))))
         (else (member?&co a
                           (cdr lat)
                           (lambda (in out)
                             (col in (cons a out))))))))
