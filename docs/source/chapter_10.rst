===================================
 What Is the Value of All of This?
===================================

.. contents::

本章概要：
::

   本章基于前面几章节的基础，
   引入了table这个数据结构，

   然后用Scheme实现了一个简化版本的Scheme解释器（自身实现自身，亦叫自举）。

----------------------------------------

.. tip::

   王垠前辈也写过一篇：怎样写一个解释器（没有原文地址，搜索即可）。

   讲的内容差不多，也是比较好理解，如果对英文没耐心的，可以参考他的文章。

entry
=====
entry是这样一种数据结构：
::

   它是一个点对，里面包含两个长度相同的子列表。

   第一个子列表中的所有S表达式各不相同（即为一个集合）。

new-entry
---------
生成一个entry数据结构

.. code-block:: scheme

   ;; build在第7章节定义过，用来生成点对
   (define new-entry build)

lookup-in-entry
---------------
根据name在entry中查找对应的值。

entry-f用来处理在entry中不存在name的情况。

.. code-block:: scheme
    
   (define lookup-in-entry
     (lambda (name entry entry-f)
       (lookup-in-entry-help name
                             (first entry)
                             (second entry)
                             entry-f)))

   (define lookup-in-entry-help
     (lambda (name names values entry-f))
       (cond
        ((null? names) (entry-f name))
        ((eq? (car names) name)
         (car values))
        (else (lookup-in-entry-help name
                                    (cdr names)
                                    (cdr values)
                                    entry-f))))

table
=====
也叫环境(environment)。它是这样一种数据结构：
::
   
   它是一个列表，里面所有的S表达式都为entry。

extend-table
------------
扩展一个table

.. code-block:: scheme

   (define extend-table cons)

lookup-in-table
---------------
根据name在table中查找其对应的值。

.. code-block:: scheme

   (define lookup-in-table
     (lambda (name table table-f)
       (cond
         ((null? table) (table-f name))
         (else (lookup-in-table name
                                (car table)
                                (lambda (name)
                                  (lookup-in-table name
                                                   (cdr table)
                                                   table-f)))))))

简化版Scheme解释器
==================
作者通过一系列的对话，总结了Scheme中几种基本语义类型。

* *const
  
* *quote

* *identfier

* *lambda

* *cond

* *application

既然Scheme作为一门函数式编程语言，\
自然就用函数(function)来表示上面这些语义类型了。

.. tip::

   如果有兴趣的同学可以在网上搜索一下其它语言的Scheme解释器实现（比如：Python），
   有的是用类来实现的。

   当然用什么来表示这些类型不是重点，重点是要了解解释器的原理。

那写解释器的第一步：
::

   我们需要知道某个S表达式属于上面归纳的那种语义类型，

   并且找到该类型对应的处理函数。

expression-to-action
--------------------
实现S表达式属于哪种语义类型，并返回其对应的处理函数。

从第一章节我们就开始了Scheme中S表达式有两种：

1. atom

2. list

.. code-block:: scheme

   (define expression-to-action
     (lambda (e)
       (cond
        ((atom? e) (atom-to-action e))
        (else (list-to-action e)))))

atom-to-action
--------------
实现atom类型的S表达式属于哪种语义类型，并返回其对应的处理函数。

.. code-block:: scheme

   (define atom-to-action
     (lambda (e)
       (cond
        ((number? e) *const
        ((eq? e #t) *const)
        ((eq? e #f) *const)
        ((eq? e (quote cons)) *const)
        ((eq? e (quote car)) *const)
        ((eq? e (quote cdr)) *const)
        ((eq? e (quote null?)) *const)
        ((eq? e (quote eq?)) *const)
        ((eq? e (quote atom?)) *const)
        ((eq? e (quote zero?)) *const)
        ((eq? e (quote add1)) *const)
        ((eq? e (quote sub1)) *const)
        ((eq? e (quote number?)) *const)
        (else *identfier)))))

list-to-action
--------------
实现atom类型的S表达式属于哪种语义类型，并返回其对应的处理函数。

.. code-block:: scheme

   (define list-to-action
     (lambda (e)
       (cond
        ((atom? (car e))
         (cond
          ((eq? (car e) (quote quote)) *quote)
          ((eq? (car e) (quote lambda)) *lambda)
          ((eq? (car e) (quote cond)) *cond)
          (else *application)))
        (else *application))))

value
-----
虽然上面两个函数里面有一堆的help函数未实现，\
但是咱们先假定它们实现，先把最外层的框架写好，再实现底层细节。

有了expression-to-action，那么解释器的雏形就有了。

.. code-block:: scheme

   ; (quote ()) 表示一个空的table
   (define value
     (lambda (e)
       (meaning e (quote ()))))

   (define meaning
     (lambda (e table)
       ((expression-to-action e) e table)))

e好理解，就是需要解释的S表达式么，table是做什么的？

table是用来存储与S表达式对应的上下文环境。

比如：S表达式 `(+ n 1)` ，当我们计算 `(+ n 1)` 怎样知道n的值？
::

   答案就在table里。

   解释器会在给n赋值的时候就将其对应关系保存到了table中，
   当调用 `(+ n 1)` 时，解释器就会从table中获取n对应的数值，然后再进行计算。

*const
------

.. code-block:: scheme

   (define *const
     (lambda (e table)
       (cond
        ((number? e) e)
        ((eq? e #t) #t)
        ((eq? e #f) #f)
        (else (build (quote primitive) e)))))

*quote
------

.. code-block:: scheme

   (define *quote
     (lambda (e table)
       (text-of e)))

   (define text-of second)

*identfier
----------
这里就是我上面关于table的说明了。

`(+ n 1)` 中的n会被当作 `identfier` 来处理，\
就会调用下面的 `*identfier` 处理函数。

`*identfier` 则从table中找出与n对应的值。

.. code-block:: scheme

   (define *identfier
     (lambda (e table)
       (lookup-in-table e table initial-table)))

   (define initial-table
     (lambda (name)
       (car (quote ()))))

*lambda
-------
元语函数和非元语函数有什么区别？

元语函数是指Scheme语言本身提供的一些基本函数。
其实就是一些我们在写Scheme解释器时预先定义好的函数。

非元语函数是用户通过(lambda ...)形式来定义的函数。\
那我们在解析到非元语函数时，需要将它的参数和函数体保存到table中，\
等待之后的*identfier调用。

.. code-block:: scheme

   (define *lambda
     (lambda (e table)
       (build (quote non-primitive)
              (cons table (cdr e)))))

*lambda会将table、参数、函数体绑定在一个列表中。

那么为了方便我们之后取出对应的数据，我再定义一些help函数。

.. code-block:: scheme

   (define table-of first)
   (define formals-of second)
   (define body-of third)

*cond
-----
.. code-block:: scheme

   (define *cond
     (lambda (e table)
       (evcon (cond-lines-of e) table)))

   (define cond-lines-of cdr)

evcon实现了cond函数的功能。

.. code-block:: scheme

   (define evcon
     (lambda (lines table)
       (cond
        ((else? (question-of (car lines)))
         (meaning (answer-of (car lines)) table))
        ((meaning (question-of (car lines)) table)
         (meaning (answer-of (car lines)) table))
        (else (evcon (cdr lines) table)))))

   ;; 判断某个S表达式是否为else分支
   (define else?
     (lambda (x)
       (cond
        ((atom? x) (eq? x (quote else)))
        (else #f))))

   (define question-of first)
   (define answer-of second)

.. note::

   evcon没有判断lines是否为空。

   可能作者是为了实现的简洁，暂时忽略这种异常情况。

   所以在使用本章节实现的Scheme解释器时，cond函数中最好至少代入一个判断分支语句。

*application
------------
application表示的是这样一种S表达式：
::

   它为一个列表，列表的第一个元素(car)表示一个函数，
   剩余的元素(cdr)表示该函数的调用参数。

.. code-block:: scheme

   (define *application
     (lambda (e table)
       (apply
         (meaning (function-of e) table)
         (evlis (arguments-of e) table))))

   (define function-of car)
   (define arguments-of cdr)

.. code-block:: scheme

   ; 获取调用参数中每参数所对应的值。
   (define evlis
     (lambda (args table)
       (cond
        ((null? args) (quote ()))
        (else (cons (meaning (car args) table)
                    (evlis (cdr args) table))))))

apply
-----
执行函数调用。

上面讲过函数分为两种：

1. 元语函数

2. 非元语函数（自定义函数）

.. code-block:: scheme
   
   (define apply
     (lambda (func vals)
       (cond
        ((primitive? fun)
         (apply-primitive (second fun) vals))
        ((non-primitive? fun)
         (apply-closure (second fun) vals)))))

.. code-block:: scheme

   (define primitive?
     (lambda (l)
       (eq? (first l) (quote primitive))))

   (define non-primitive?
     (lambda (l)
       (eq? (first l) (quote non-primitive))))

.. code-block:: scheme

   (define apply-primitive
     (lambda (name vals)
       (cond
        ((eq? name (quote cons))
         (cons (first vals) (second vals)))
        ((eq? name (quote car))
         (car (first vals)))
        ((eq? name (quote cdr))
         (cdr (first vals)))
        ((eq? name (quote null?))
         (null? (first vals)))
        ((eq? name (quote eq?))
         (eq? (first vals)))
        ((eq? name (quote atom?))
         (:atom? (first vals)))
        ((eq? name (quote zero?))
         (zero? (first vals)))
        ((eq? name (quote add1))
         (add1 (first vals)))
        ((eq? name (quote sub1))
         (sub1 (first vals)))
        ((eq? name (quote number?))
         (number? (first vals))))))

   (define :atom?
     (lambda (x)
       (cond
        ((atom? x) #t)
        ((null? x) #f)
        ((eq? (car x) (quote primitive)) #t)
        ((eq? (car x) (quote non-primitive)) #t)
        (else #f))))

.. code-block:: scheme

   ;; 将调用参数及非元语函数的参数组成一个entry（组成对应关系）
   ;; 然后添加到table中，再对该非元语函数的函数体求值即可。
   (define apply-closure
     (lambda (closure vals)
       (meaning (body-of closure)
                (extend-table (new-entry (formals-of closure)
                                         vals)
                              (table-of closure)))))

结语
====
到这里，一个简化版的Scheme解释器就完成了，\
当然，这个解释器离实用还很远，\
如果想对其进一步了解，请阅读《The Seasoned Schemer》。

