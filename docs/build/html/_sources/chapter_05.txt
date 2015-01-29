====================================
 \*Oh My Gawd\*: It's Full of Stars
====================================

.. contents::

本章概要：
::

   本章节稍微添加了一点难度。

   以前都只处理一层列表，现在需要处理嵌套列表（含有子列表的列表）。

----------------------------------------

rember*
=======
基本同rember, 但其处理的对象包括一个嵌套的列表

.. code-block:: scheme

   ; rember*: symbol list -> list
   (define rember*
     (lambda (a l)
       (cond
        ((null? l) '())
        ((atom? (car l))
         (cond
          ((eq? a (car l))
           (rember* a (cdr l)))
          (else (cons (car l) (rember* a (cdr l))))))
        (else (cons (rember* a (car l))
                    (rember* a (cdr l)))))))

.. code-block:: scheme
    
   (define col
     (lambda (l)
       (display l)
       (newline)
       l))
    
   (define rember*&co
     (lambda (a l col)
       (cond
        ((null? l) (col '()))
        ((atom? (car l))
         (cond
          ((eq? a (car l))
           (rember*&co a (cdr l) col))
          (else
           (rember*&co a
                       (cdr l)
                       (lambda (newl)
                         (col (cons (car l) newl)))))))
        (else
         (rember*&co a
                     (car l)
                     (lambda (carl)
                       (rember*&co a
                                   (cdr l)
                                   (lambda (cdrl)
                                     (col (cons carl cdrl))))))))))

insertR*
========
基本同insertR, 但其处理的对象包括一个嵌套的列表

.. code-block:: scheme

   ; insertR*: symbol symbol list -> list
   (define insertR*
     (lambda (new old l)
       (cond
        ((null? l) '())
        ((atom? (car l))
         (cond
          ((eq? old (car l))
           (cons old (cons new (insertR* new old (cdr l)))))
          (else (cons (car l) (insertR* new old (cdr l))))))
        (else (cons (insertR* new old (car l))
                    (insertR* new old (cdr l)))))))

.. code-block:: scheme
    
   (define col
     (lambda (l)
       (display l)
       (newline)
       l))
   
   (define insertR*&co
     (lambda (new old l col)
       (cond
        ((null? l) (col '()))
        ((atom? (car l))
         (cond
          ((eq? old (car l))
           (insertR*&co new
                        old
                        (cdr l)
                        (lambda (newl)
                          (col (cons old
                                     (cons new newl))))))
          (else
           (insertR*&co new
                        old
                        (cdr l)
                        (lambda (newl)
                          (col (cons (car l)
                                     newl)))))))
        (else
         (insertR*&co new
                      old
                      (car l)
                      (lambda (carl)
                        (insertR*&co new
                                     old
                                     (cdr l)
                                     (lambda (cdrl)
                                       (col (cons carl cdrl))))))))))

occur*
======
基本同occur, 但其处理的对象包括一个嵌套的列表

.. code-block:: scheme

   ; occur*: symbol list -> number
   (define occur*
     (lambda (a l)
       (cond
        ((null? l) 0)
        ((atom? (car l))
         (cond
          ((eq? a (car l))
           (add1 (occur* a (cdr l))))
          (else (occur* a (cdr l)))))
        (else (+ (occur* a (car l))
                 (occur* a (cdr l)))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
   
   (define occur*&co
     (lambda (a l col)
       (cond
        ((null? l) (col 0))
        ((atom? (car l))
         (cond
          ((eq? a (car l))
           (occur*&co a (cdr l) (lambda (num)
                                  (col (add1 num)))))
          (else
           (occur*&co a (cdr l) col))))
        (else
         (occur*&co a
                    (car l)
                    (lambda (carnum)
                      (occur*&co a
                                 (cdr l)
                                 (lambda (cdrnum)
                                   (col (+ carnum cdrnum))))))))))

subst*
======
基本同subst, 但其处理的对象包括一个嵌套的列表

.. code-block:: scheme

   ; subst*: symbol symbol list -> list
   (define subst*
     (lambda (new old l)
       (cond
        ((null? l) '())
        ((atom? (car l))
         (cond
          ((eq? old (car l))
           (cons new (subst* new old (cdr l))))
          (else (cons (car l) (subst* new old (cdr l))))))
        (else (cons (subst* new old (car l))
                    (subst* new old (cdr l)))))))

.. code-block:: scheme
    
   (define col
     (lambda (l)
       (display l)
       (newline)
       l))
   
   (define subst*&co
     (lambda (new old l col)
       (cond
        ((null? l) (col l))
        ((atom? (car l))
         (cond
          ((eq? old (car l))
           (subst*&co new
                      old
                      (cdr l)
                      (lambda (newl)
                        (col (cons new newl)))))
          (else
           (subst*&co new
                      old
                      (cdr l)
                      (lambda (newl)
                        (col (cons (car l) newl)))))))
        (else
         (subst*&co new
                    old
                    (car l)
                    (lambda (carl)
                      (subst*&co new
                                 old
                                 (cdr l)
                                 (lambda (cdrl)
                                   (col (cons carl cdrl))))))))))

insertL*
========
基本同insertL, 但其处理的对象包括一个嵌套的列表

.. code-block:: scheme

   ; insertL*: symbol symbol list -> list
   (define insertL*
     (lambda (new old l)
       (cond
        ((null? l) '())
        ((atom? (car l))
         (cond
          ((eq? old (car l))
           (cons new (cons old (insertL* new old (cdr l)))))
          (else (cons (car l) (insertL* new old (cdr l))))))
        (else (cons (insertL* new old (car l))
                    (insertL* new old (cdr l)))))))

.. code-block:: scheme
    
   (define col
     (lambda (l)
       (display l)
       (newline)
       l))
   
   (define insertL*&co
     (lambda (new old l col)
       (cond
        ((null? l) (col l))
        ((atom? (car l))
         (cond
          ((eq? old (car l))
           (insertL*&co new
                        old
                        (cdr l)
                        (lambda (newl)
                          (col (cons new (cons old newl))))))
          (else
           (insertL*&co new
                        old
                        (cdr l)
                        (lambda (newl)
                          (col (cons old newl)))))))
        (else
         (insertL*&co new
                      old
                      (car l)
                      (lambda (carl)
                        (insertL*&co new
                                     old
                                     (cdr l)
                                     (lambda (cdrl)
                                       (col (cons carl cdrl))))))))))

member*
=======
基本同member, 但其处理的对象包括一个嵌套的列表

.. code-block:: scheme

   ; member*: symbol list -> bool
   (define member*
     (lambda (a l)
       (cond
        ((null? l) #f)
        ((atom? a (car l))
         (cond
          ((eq? a (car l)) #t)
          (else (member* a (cdr l)))))
        (else (or (member* a (car l))
                  (member* a (cdr l)))))))

.. code-block:: scheme
    
   (define col
     (lambda (num)
       (display num)
       (newline)
       num))
   
   (define member*&co
     (lambda (a l col)
       (cond
        ((null? l) (col #f))
        ((atom? (car l))
         (cond
          ((eq? a (car l))
           (col #t))
          (else
           (member*&co a (cdr l) col))))
        (else
         (member*&co a
                     (car l)
                     (lambda (incar)
                       (member*&co a
                                   (cdr l)
                                   (lambda (incdr)
                                     (col (or incar incdr))))))))))

leftmost
========
找出不包含空列表的列表/嵌套列表中的最左边的一个atom

.. code-block:: scheme

   ; leftmost: list -> symbol
   (define leftmost
     (lambda (l)
       (cond
        ((atom? (car l)) (car l))
        (else (leftmost (car l))))))

.. code-block:: scheme

   (define col
     (lambda (s)
       (display s)
       (newline)
       s))

   (define leftmost&co
     (lambda (l col)
       (cond
        ((atom? (car l)) (col (car l)))
        (else (leftmost (car l) col)))))

eqlist?
=======
判断两个列表是否相等（包含嵌套列表）


.. code-block:: scheme

   ; eqlist?: list list -> bool
   (define eqlist?
     (lambda (l1 l2)
       (cond
        ((and (null? l1) (null? l2)) #t)
        ((and (null? l1) (atom? (car l2))) #f)
        ((null? l1) #f)
        ((and (atom? (car l1)) (null? l2)) #f)
        ((and (atom? (car l1)) (atom? (car l2)))
         (and (eqan? (car l1) (car l2))
              (eqlist? (cdr l1) (cdr l2))))
        ((atom? (car l1)) #f)
        ((null? l2) #f)
        ((atom? (car l2)) #f)
        (else
         (and (eqlist? (car l1) (car l2))
              (eqlist? (cdr l1) (cdr l2)))))))
   
   (define eqlist?
     (lambda (l1 l2)
       (cond
        ((and (null? l1) (null? l2)) #t)
        ((or (null? l1) (null? l2)) #f)
        ((and (atom? (car l1))
              (atom? (car l2)))
         (and (eqan? (car l1) (car l2))
              (eqlist? (cdr l1) (cdr l2))))
        ((or (atom? (car l1))
             (atom? (car l2))) #f)
        (else (and (eqlist? (car l1) (car l2))
                   (eqlist? (cdr l1) (cdr l2)))))))

equal?
======
判断两个S表达式是否相等

.. code-block:: scheme
    
   (define equal?
     (lambda (s1 s2)
       (cond
        ((and (atom? s1) (atom? s2))
         (eqan? s1 s2))
        ((or (atom? s1) (atom? s2)) #f)
        (else (eqlist? s1 s2)))))

简化版eqlist?
=============
作者通过equal?来简化了eqlist?，

而且equal?也是通过eqlist?来实现的。

SO，只当函数正确的前提下再进行简化/优化。

.. code-block:: scheme

   (define eqlist?
     (lambda (l1 l2)
       (cond
        ((and (null? l1) (null? l2)) #t)
        ((or (null? l1) (null? l2)) #f)
        (else
         (and (equal? (car l1) (car l2))
              (eqlist? (cdr l1) (cdr l2)))))))

rember
======
重写之前的rember，使用equal?来简化之。

参数s代表任何S表达式，参数l代码任何列表。

.. code-block:: scheme

   ; rember: symbol list -> list
   (define rember
     (lambda (s l)
       (cond
        ((null? l) '())
        ((equal? s (car l)) (cdr l))
        (else (cons (car l) (rember s (cdr l)))))))
