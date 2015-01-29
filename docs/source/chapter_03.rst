======================
 Cons the Magnificent
======================

.. contents::

本章节概要：
::

   继续讲解lat/列表相关的操作及使用递归来进行代码实现

----------------------------------------

rember
======
将lat中首次出现的某个S表达式删除。

.. code-block:: scheme

   ; rember: symbol lat -> lat
   (define rember
     (lambda (a lat)
       (cond
         ((null? lat) '())
         ((eq? a (car lat)) (cdr lat))
         (else (cons (car lat) (rember a (cdr lat)))))))

.. code-block:: scheme

   ; 参数rmcount不是必需的，也可以去掉
   (define col
     (lambda (rmcount leftlat)
       (display rmcount)
       (newline)
       (display leftlat)
       (newline)
       leftlat))
  
   (define rember&co
     (lambda (a lat col)
       (cond
         ((null? lat) (col 0 '()))
         ((eq? a (car lat)) (col 1 (cdr lat)))
         (else (rember&co a
                          (cdr lat)
                          (lambda (rmcount leftlat)
                            (col rmcount (cons (car lat) leftlat))))))))

firsts
======
从一个列表中的获取其每个子列表的首个S表达式，并以列表形式返回

.. note::

   此函数并未考虑每个子列表为空表的情况

.. code-block:: scheme

   ; firsts: list -> list
   (define firsts
     (lambda (l)
       (cond
         ((null? l) '())
         (else (cons (car (car l)) (firsts (cdr l)))))))

.. code-block:: scheme
  
   (define col
     (lambda (first_ed)
       (display first_ed)
       (newline)
       first_ed))
  
   (define firsts&co
     (lambda (l col)
       (cond
       ((null? l) (col '()))
       (else (firsts&co (cdr l)
                        (lambda (first_ed)
                          (col (cons (car (car l)) first_ed))))))))

insertR
=======
将一个S表达式插入到一个列表中指定S表达式的右边，并返回修改后的列表 

.. code-block:: scheme

   ; insertR: symbol symbol lat -> lat 
   (define insertR
     (lambda (new old lat)
       (cond
         ((null? lat) '())
         ((eq? old (car lat)) (cons old (cons new (cdr lat))))
         (else (cons (car lat) (insertR new old (cdr lat)))))))

.. code-block:: scheme
  
   (define col
     (lambda (newlat)
       (display newlat)
       (newline)
       newlat))
  
   (define insertR&co
     (lambda (new old lat col)
       (cond
         ((null? lat) (col '()))
         ((eq? old (car lat))
          (col (cons old (cons new (cdr lat)))))
         (else (insertR&co new
                           old
                           (cdr lat)
                           (lambda (newlat)
                             (col (cons (car lat) newlat))))))))

insertL
=======
将一个S表达式插入到一个列表中指定S表达式的左边，并返回修改后的列表 

.. code-block:: scheme

   ; insertL: symbol symbol lat -> lat
   (define insertL
     (lambda (new old lat)
       (cond
         ((null? lat) '())
         ((eq? old (car lat)) (cons new lat))
         (else (cons (car lat) (insertL new old (cdr lat)))))))

.. code-block:: scheme
  
   (define col
     (lambda (newlat)
       (display newlat)
       (newline)
       newlat))
  
   (define insertL&co
     (lambda (new old lat col)
       (cond
         ((null? lat) (col '()))
         ((eq? old (car lat)) (col (cons new lat)))
         (else (insertL&co new
                           old
                           (cdr lat)
                           (lambda (newlat)
                             (col (cons (car lat) newlat))))))))

subst
=====
用新的S表达式替代列表中指定的S表达式，并返回修改后的列表 
  
.. code-block:: scheme

   ; subst: symbol symbol lat -> lat
   (define subst
     (lambda (new old lat)
       (cond
         ((null? lat) '())
         ((eq? old (car lat)) (cons new (cdr lat)))
         (else (cons (car lat) (subst new old (cdr lat)))))))

.. code-block:: scheme
  
   (define col
     (lambda (newlat)
       (display newlat)
       (newline)
       newlat))
  
   (define subst&co
     (lambda (new old lat col)
       (cond
         ((null? lat) (col '()))
         ((eq? old (car lat))
         (col (cons new (cdr lat))))
         (else (subst&co new
                         old
                         (cdr lat)
                         (lambda (newlat)
                           (col (cons (car lat) newlat))))))))

subst2
======
用来替代列表中指定的两个S表达式

.. code-block:: scheme

   ; subst2: symbol symbol symbol lat -> lat
   (define subst2
     (lambda (new old1 old2 lat)
       (cond
         ((null? lat) '())
         ((eq? old1 (car lat)) (cons new (cdr lat)))
         ((eq? old2 (car lat)) (cons new (cdr lat)))
         (else (cons (car lat) (subst2 new old1 old2 (cdr lat)))))))
  
   ;subst2简化版本
   (define subst2
     (lambda (new old1 old2 lat)
       (cond
         ((null? lat) '())
         ((or (eq? old2 (car lat))
              (eq? old1 (car lat)))
          (cons new (cdr lat)))
         (else (cons (car lat)
                     (subst2 new old1 old2 (cdr lat)))))))

.. code-block:: scheme
  
   (define col
     (lambda (newlat)
       (display newlat)
       (newline)
       newlat))
  
   (define subst2&co
     (lambda (new old1 old2 lat col)
       (cond
         ((null? lat) (col '()))
         ((or (eq? old1 (car lat))
              (eq? old2 (car lat)))
          (col (cons new (cdr lat))))
         (else (subst&co new
                         old1
                         old2
                         (cdr lat)
                         (lambda (newlat)
                           (col (cons (car lat) newlat))))))))

multirember
===========
基本同rember, 只不过是列表中所有符合的S表达式都会删除.

.. code-block:: scheme

   ; multirember: symbol lat -> lat
   (define multirember
     (lambda (a lat)
       (cond
         ((null? lat) '())
         ((eq? a (car lat)) (multirember a (cdr lat)))
         (else (cons (car lat) (multirember a (cdr lat)))))))

.. code-block:: scheme
  
   (define col
     (lambda (rmcount newlat)
       (display rmcount)
       (display " ")
       (display newlat)
       (newline)
       newlat))
  
  (define multirember&co
    (lambda (a lat col)
      (cond
       ((null? lat) (col 0 '()))
       ((eq? a (car lat))
        (multirember&co a
                        (cdr lat)
                        (lambda (rmcount newlat)
                          (col (+ rmcount 1) newlat))))
       (else
        (multirember&co a
                        (cdr lat)
                        (lambda (rmcount newlat)
                          (col rmcount (cons (car lat) newlat))))))))

multiinsertR
============
基本同insertR，只不过是列表中所有符合的s表达式都会插入其右边。

.. code-block:: scheme

   ; multiinsertR: symbol symbol lat -> lat
   (define multiinsertR
     (lambda (new old lat)
       (cond
        ((null? lat) '())
        ((eq? old (car lat))
         (cons old (cons new (multiinsertR new old (cdr lat)))))
        (else (cons (car lat) (multiinsertR new old (cdr lat)))))))

.. code-block:: scheme
    
   (define col
     (lambda (inscount newlat)
       (display inscount)
       (display " ")
       (display newlat)
       (newline)
       newlat))
   
   (define multiinsertR&co
     (lambda (new old lat col)
       (cond
        ((null? lat) (col 0 '()))
        ((eq? old (car lat))
           (multiinsertR&co new
                            old
                            (cdr lat)
                            (lambda (inscount newlat)
                              (col (+ inscount 1)
                                   (cons old (cons new newlat))))))
        (else
           (multiinsertR&co new
                            old
                            (cdr lat)
                            (lambda (inscount newlat)
                              (col inscount
                                   (cons (car lat) newlat))))))))

multiinsertL
============
基本同insertL, 只不过是列表中所有符合的s表达式都会插入其左边.

.. code-block:: scheme

   ; multiinsertL: symbol symbol lat -> lat
   (define multiinsertL
     (lambda (new old lat)
       (cond
        ((null? lat) '())
        ((eq? old (car lat))
         (cons new (cons old (multiinsertL new old (cdr lat)))))
        (else (cons (car lat) (multiinsertL new old (cdr lat)))))))

.. code-block:: scheme
  
   (define col
     (lambda (inscount newlat)
       (display inscount)
       (display " ")
       (display newlat)
       (newline)
       newlat))
   
   (define mutlinsertL&co
     (lambda (new old lat col)
       (cond
        ((null? lat) (col 0 '()))
        ((eq? old (car lat))
         (mutlinsertL&co new
                         old
                         (cdr lat)
                         (lambda (inscount newlat)
                           (col (+ inscount 1)
                                (cons new (cons old newlat))))))
        (else
         (mutlinsertL&co new
                         old
                         (cdr lat)
                         (lambda (inscount newlat)
                           (col inscount
                                (cons (car lat) newlat))))))))
 
multisubst
==========
基本同subst, 只不过是列表中所有符合的s表达式都替换

.. code-block:: scheme

   ; multisubst: symbol symbol lat -> lat
   (define multisubst
     (lambda (new old lat)
       (cond
        ((null? lat) '())
        ((eq? old (car lat))
         (cons new (multisubst new old (cdr lat))))
        (else (cons (car lat) (multisubst new old (cdr lat)))))))

.. code-block:: scheme
  
   (define col
     (lambda (stcount newlat)
       (display stcount)
       (display " ")
       (display newlat)
       (newline)
       newlat))
   
   (define mutlsubst&co
     (lambda (new old lat col)
       (cond
        ((null? lat) (col 0 '()))
        ((eq? old (car lat))
         (mutlisubst&co new
                        old
                        (cdr lat)
                        (lambda (inscount newlat)
                          (col (+ inscount 1)
                               (cons new newlat)))))
        (else
         (mutlisubst&co new
                        old
                        (cdr lat)
                        (lambda (inscount newlat)
                          (col inscount
                               (cons (car lat) newlat))))))))
