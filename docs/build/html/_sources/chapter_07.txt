=======================
 Friends and Relations
=======================

.. contents::

本章概要：
::

   介绍了Scheme中的几种常见数据结构。

   为最后一章节的Scheme解释器实现打基础。

----------------------------------------

set
===
集合, 类似于列表, 但是它包含的每个元素在集合内都是唯一的

set?
----
判断一个S表达式是否为set

.. code-block:: scheme

   ; set?: lat -> bool
   (define set?
     (lambda (lat)
       (cond
        ((null? lat) #t)
        ((member? (car lat) (cdr lat)) #f)
        (else (set? (cdr lat))))))

.. code-block:: scheme

   (define col
     (lambda (bflag)
       (display bflag)
       (newline)
       bflag))

   (define set?&co
     (lambda (lat col)
       (cond
        ((null? lat) (col #t))
        ((member? (car lat) (cdr lat)) (col #f))
        (else (set?&co (cdr lat) col)))))

makeset
-------
生成一个set

.. code-block:: scheme

   ; makeset: lat -> set
   (define makeset
     (lambda (lat)
       (cond
        ((null? lat) '())
        ((member? (car lat) (cdr lat))
         (makeset (cdr lat)))
        (else (cons (car lat) (makeset (cdr lat)))))))
    
   ;使用multirember, 另外一种思路
   (define makeset
     (lambda (lat)
       (cond
        ((null? lat) '())
        (else (cons (car lat)
                    (makeset (multirember (car lat)
                                          (cdr lat))))))))

.. code-block:: scheme

   (define col
     (lambda (set)
       (display set)
       (newline)
       set))

   (define makeset&co
     (lambda (lat col)
       (cond
        ((null? lat) (col '()))
        ((member? (car lat) (cdr lat))
         (makeset&co (cdr lat) col))
        (else makeset&co (cdr lat)
                         (lambda (newset)
                           (col (cons (car lat) newset)))))))

subset?
-------
判断set1是否是set2的子集

.. code-block:: scheme

   ; subset?: set set -> bool
   (define subset?
     (lambda (set1 set2)
       (cond
        ((null? set1) #t)
        ((member? (car set1) set2)
         (subset? (cdr set1) set2))
        (else #f))))

   ; 另一版本
   (define subset?
     (lambda (set1 set2)
       (cond
        ((null? set1) #t)
        ((and (member? (car set1) set2)
              (subset? (cdr set1) set2))))))

.. code-block:: scheme

   (define col
     (lambda (bflag)
       (display bflag)
       (newline)
       bflag))

   (define subset&co
     (lambda (set1 set2 col)
       (cond
        ((null? set1) (col #t))
        ((member? (car set1) set2)
         (subset&co (cdr set1) set2))
        (else (col #f)))))

eqset?
------
判断两个set是否相等

.. code-block:: scheme

   ; eqset?: set set -> bool
   (define eqset?
     (lambda (set1 set2)
       ((and (subset? set1 set2)
             (subset? set2 set1)))))

interset?
---------
判断set1是否至少有一个S表达式在set2中

.. code-block:: scheme

   ; interset?: set set -> bool
   (define intersect?
     (lambda (set1 set2)
       (cond
        ((null? set1) #t)
        ((member? (car set1) set2) #t)
        (else (intersect? (cdr set1) set2)))))

   ; 另一版本
   (define intersect?
     (lambda (set1 set2)
       (cond
        ((null? set1) #t)
        ((or (member? (car set1) set2)
             (intersect? (cdr set1) set2))))))

.. code-block:: scheme

   (define col
     (lambda (bflag)
       (display bflag)
       (newline)
       bflag))

   (define intersect?&co
     (lambda (set1 set2 col)
       (cond
        ((null? set1) (col #t))
        ((member? (car set1) set2) (col #t))
        (else (intersect?&co (cdr set1) set2 col)))))

interset
--------
求两个set的交集

.. code-block:: scheme

   ; set set -> set
   (define intersect
     (lambda (set1 set2)
       (cond
        ((or (null? set1)
             (null? set2))
         '())
        ((member? (car set1) set2)
         (cons (car set1)
               (intersect (cdr set1) set2)))
        (else (intersect (cdr set1) set2)))))

.. code-block:: scheme

   (define col
     (lambda (set)
       (display set)
       (newline)
       set))

   (define intersect&co
     (lambda (set1 set2 col)
       (cond
        ((or (null? set1) (null? set2))
         (col '()))
        ((member? (car set1) set2)
         (intersect&co (cdr set1)
                       set2
                       (lambda (newset)
                          (col (cons (car set1) newset)))))
        (else (intersect&co (cdr set1) set2 col)))))

union
-----
求两个set的并集

.. code-block:: scheme

   ; union: set set -> set
   (define union
     (lambda (set1 set2)
       (cond
        ((null? set1) set2)
        ((null? set2) set1)
        ((member? (car set1) set2)
         (union (cdr set1) set2))
        (else (cons (car set1)
                    (union (cdr set1) set2))))))

.. code-block:: scheme

   (define col
     (lambda (set)
       (display set)
       (newline)
       set))

   (define union&co
     (lambda (set1 set2 col)
       (cond
        ((null? set1) (col set2))
        ((null? set2) (col set1))
        ((member? (car set1) set2)
         (union&co (cdr set1) set2 col))
        (else union&co (cdr set1)
                       set2
                       (lambda (newset)
                         (col (cons (car set1) newset)))))))

intersectall
------------
获取set中每个子set的交集

.. code-block:: scheme

   ; intersectall: set -> set
   (define intersectall
     (lambda (l-set)
       (cond
        ((null? (cdr l-set)) (car l-set))
        (else (intersect (car l-set)
                         (intersectall (cdr l-set)))))))

.. code-block:: scheme

   (define col
     (lambda (set)
       (display set)
       (newline)
       set))

   (define intersectall&co
     (lambda (l-set col)
       (cond
        ((null? (cdr l-set))
         (col (car l-set)))
        (else (intersectall&co (cdr l-set)
                               (lambda (newset)
                                 (col (interset (car l-set) newset))))))))

pair
====
点对, 只包含两个S表达式的列表

a-pair?
-------
判断一个S表达式是否为pair

.. code-block:: scheme
    
   (define a-pair?
     (lambda (x)
       (cond
        ((atom? x) #f)
        ((null? x) #f)
        ((null? (cdr lat) #f))
        ((null? (cdr (cdr lat))) #t)
        (else #f))))

first
-----
获取pair的第一个S表达式

.. code-block:: scheme
  
   (define first
     (lambda (p)
       (car p)))

second
------
获取pair的第二个S表达式

.. code-block:: scheme
    
   (define second
     (lambda (p)
       (car (cdr p))))

build
-----
生成一个pair

.. code-block:: scheme
    
   (define build
     (lambda (s1 s2)
       (cons s1 (cons s2 '()))))

rel
===
是一个内部嵌套pair的list, 但是其所有子pair是唯一的

.. tip::

   相当于一个pair集合

fun
===
基本同rel, 但其所有子pair的第一个元素也是唯一的

fun?
----
判断一个S表达式是否为fun

.. code-block:: scheme
    
   (define fun?
     (lambda (rel)
       (set? (firsts rel))))

revrel
------
将rel中所有子pair的两个元素对调

.. code-block:: scheme

   ; revrel: rel -> rel
   (define revrel
     (lambda (rel)
       ((null? rel) '())
       (else (cons (build (second (car rel))
                          (first (car rel)))
                   (revrel (cdr rel))))))

如果将其中的pair中两元素对调写成一个单独的函数，\
则revrel看起来会更简洁明了。

.. code-block:: scheme

   (define revpair
     (lambda (pair)
       (build (second pair)
              (first pair))))
   
   (define revrel
     (lambda (rel)
       ((null? rel) '())
       (else (cons (revpair (car rel))
                   (revrel (cdr rel))))))

.. code-block:: scheme

   (define col
     (lambda (rel)
       (display rel)
       (newline)
       rel))

   (define revrel&co
     (lambda (rel col)
       ((null? rel) (col '()))
       (else (revrel&co (cdr rel)
                        (lambda (newrel)
                          (col (cons (revpair (car rel))
                                     newrel)))))))

fullfun
=======
基本同fun, 但其所有子pair的第二个元素也是唯一的

fullfun?
--------
判断一个S表达式是否为fullfun

.. code-block:: scheme
    
   (define fullfun?
     (lambda (fun)
       (set? (seconds fun))))
   
   (define fullfun?
     (lambda (fun)
       (fun? (revrel fun))))
