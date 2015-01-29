========================================
 ... and Again, and Again, and Again ...
========================================

.. contents::

本章概要：
::

   这一章节的内容基本讲的都是数学上的概念或者理论。

   一上来先讲到了偏函数（数学概念），然后由此引入了停机问题，

   最后讲到了递归的理论基础：Y算子。

----------------------------------------

偏函数
======
looking
-------
从lat的第一元素找起，如果第一元素与a相同同为#t，\
如果不相同，则跳转到第一元素所对应的lat中的位置再进行比较，\
直到找到与a相同的S表达式为止。

.. code-block:: scheme

   ; looking: symbol lat -> bool
   (define looking
     (lambda (a lat)
       (keep-looking a (pick 1 lat) lat)))

.. code-block:: scheme
  
   (define keep-looking
     (lambda (a sorn lat)
       (cond
         ((number? sorn)
          (keep-looking a (pick sorn lat) lat))
         (else (eq? sorn a)))))

细心的读者看上面的代码或者定义就知道looking函数有个问题：
::

   并不所有的输入它都能生成对应的结果。

比如：
::

   a = caviar

   lat = (7 1 2 caviar 5 6 3)

上面两个参数代入looking函数的后果就是一直死循环，永远不会有结果。

作者将这类不能对所有参数都能产生结果的函数称为 **偏函数** 。

.. tip::

   这里的 **偏函数** 是指数字概念上的偏函数，\
   和许多编程语言里的偏函数是不一样的。

   数学上偏函数定义请参见 `Wiki: 函数`_

eternity
--------
然后作者举例了一个更彻底更简短的偏函数：

.. code-block:: scheme

   (define eternity
     (lambda (x)
       (eternity x)))

eternity对所有的参数都不能产生结果。

shift
-----
该函数的参数为一个点对，且该点对的第一个S表达式也为一个点对(first)，\
函数会将该参数的中的第中的第二个S表达式(second)和first中的second合并。

.. tip::
   
   Sorry，本人水平有限，看不明白上面的话就直接看代码吧。

.. code-block:: scheme
  
   (define shift
     (lambda (pair)
       (build (first (first pair))
              (build (second (first pair))
                     (second pair)))))

align
-----
.. code-block:: scheme

   (define align
     (lambda (pora)
       (cond
        ((atom? pora) pora)
        ((a-pair? (first pora))
         (align (shift pora)))
        (else (build (first pora)
                     (align (second pora)))))))

align仅仅是调整了参数中S表达式的位置。

从它的表面行为来看，它貌似也违反了十戒中的第七戒：
::

   当一个对象的子对象是与其本身表现一致时，这时候可用递归操作。比如：

   * 一个列表的子列表

   * 一个算术表达式的子表达式

因为参数经过align操作后，并非原参数的子列表，\
所以我们目前无法一眼就确认align是一个全函数还是一个偏函数。

那么，咱们接下来找出方法来确定align是一个全函数还是一个偏函数。

.. tip::

   其实，参与递归的参数要有一个趋向性，第七戒只能算是其中一种。

length*
-------
计算align参数的长度。

.. code-block:: scheme

   (define length*
     (lambda (pora)
       (cond
        ((atom? pora) 1)
        (else
         (+ (length* (first pora))
            (length* (second pora)))))))

我们先看看这个指标是能否表示align的趋向性。

.. code-block:: scheme

   > (align '((a b) (c d)))
   $25 = (a (b (c d)))

   > (length* '((a b) (c d)))
   $26 = 4

   > (length* '(a (b (c d))))
   $27 = 4

好吧，这个指标明显不能表示align的趋向性，\
传入的参数和输出的结果的长度都一样，只能继续找其它指标。
   
weight*
-------
计算align参数的权重。

align每次都会对点对中的第一个S表达式特殊对待，所以我们可以增加它的权重，\
在计算其长度的基础上，设定第一个S表达式占的权重为2,第二个S表达式占的权重为1。

.. code-block:: scheme

   (define weight*
     (lambda (pora)
       (cond
        ((atom? pora) 1)
        (else
         (+ (* (weight* (first pora)) 2)
            (weight* (second pora)))))))

我们再来试试看。

.. code-block:: scheme

   > (align '((a b) (c d)))
   $25 = (a (b (c d)))

   > (weight* '((a b) (c d)))
   $26 = 9

   > (weight* '(a (b (c d))))
   $27 = 7

嗯，看出苗头了，如果我们用更多的参数进行测试的话，\
这个趋向性就更明显了，所以说，align不是一个偏函数。

shuffle
-------
还是刚才align的逻辑，只不过用revpair函数来替换shift函数。

.. code-block:: scheme

   (define shuffle
     (lambda (pora)
       (cond
        ((atom? pora) pora)
        ((a-pair? (first pora))
         (shuffle (revpair pora)))
        (else (build (first pora)
                     (shuffle (second pora)))))))

现在咱们再来求证shuffle是否是一个偏函数。

.. code-block:: scheme

   > (shuffle '(a (b c)))
   $1 = (a (b c))
   > (shuffle '(a b))
   $2 = (a b)
   > (shuffle '((a b) (c d)))
   ... ...

(shuffle '((a b) (c d)))一直运行不出结果（死循环），所以它是偏函数。

其它例子
--------
接下来作者又列出了几个偏函数的例子并且直接给出结果，未给出证明步骤。

.. code-block:: scheme

   (define C
     (lambda (n)
       (cond
        ((one? n) 1)
        (else
         (cond
          ((even? n) (C (/ n 2)))
          (else (C (add1 (* 3 n)))))))))

C函数除了传入参数0没有值外，其它都能得到结果，所以它是一个偏函数。

.. code-block:: scheme

   (define A
     (lambda (n m)
       (cond
        ((zero? n) (add1 m))
        ((zero? m) (A (sub1 n) 1))
        (else (A (sub1 n) (A n (sub1 m)))))))

A虽然是一个全函数，但是对于某些参数，它的运行时间可能会非常地长。

.. tip::

   作者给出了一系列的例子，是为了引入了停机问题。

停机问题
========

.. tip::

   相关概念可参考：

   1. `Wiki: 停机问题`_

   2. `如何通俗地解释停机问题（Halting Problem）？`_

既然有偏函数这种求不出结果（一直在运算，不会返回结果）的函数存在，\
那我们能不能编写出一个函数，能够判断某个函数是否是偏函数呢？

我们先假设有这样一个函数：

.. code-block:: scheme

   (define will-stop?
     (lambda (f)
       ...))

然后利用这个函数再编写一个函数：

.. code-block:: scheme

   (define last-try
     (lambda (x)
       (and (will-stop? last-try)
            (eternity x))))

1. 假设(will-stop? last-try)返回的是#t，\
   则(and (will-stop? last-try) (eternity x)) == (eternity x)，\
   (eternity x)是偏函数，不会返回结果，所以last-try也不会返回结果。\
   该假设不成立。
   
2. 假设(will-stop? last-try)返回的是#f，\
   则(and (will-stop? last-try) (eternity x))为#f，\
   所以last-try也返回#f。该假设也不成立。

通过上面的证明， 我们无法定义出will-stop?这种函数出来。

Y算子
=====
Scheme通过(define ...)来定义一个函数。\
上面我们说到我们无法定义出will-stop?函数，\
作者此时话峰一转，问道：那递归函数呢？

我在定义某个递归函数的时候（以length为例），\
length还在正在被定义当中，怎么能自己调用自己呢？

通过这个问题作者就引入了Y算子(Y Combiniator)，也叫Y组合子。
::

   不动点组合子（Fixed-point combinator；不动点算子）是计算其他函数的一个不动点的高阶函数。

   函数f的不动点是一个值x使得f(x) = x。

   鉴于一阶函数（在简单值比如整数上的函数）的不动点是个一阶值，
   高阶函数f的不动点是另一个函数g使得f(g) = g。
   那么，不动点算子是任何函数fix使得对于任何函数f都有

     f(fix(f)) = fix(f)。

   不动点组合子允许定义匿名的递归函数。它们可以用非递归的lambda抽象来定义。

.. tip::

   相关概念可参考：

   1. `The Why of Y`_

   2. `Wiki: 不动点组合子`_

   此外，下面的推导过程大量使用了 `Wiki: λ演算`_ 中的 **α-变换** 和 **β-归约** 这两个概念。

接下的代码就是就以length函数为例，一步步推导出Y算子。

.. code-block:: scheme

   ;; 使用define的length函数定义
   (define length
     (lambda (l)
       (cond
        ((null? l) 0)
        (else (add1 (length (cdr l)))))))

现在没有了define这个定义功能，也就是说没有了length这个名称了，\
我们怎么调用自身呢？

好吧，让我先从最简单的情形一步步分析。

.. code-block:: scheme

   ;; length0
   ;; 计算元素个数为0的列表长度
   (lambda (l)
     (cond
      ((null? l) 0)
      (else (add1 (eternity (cdr l))))))
   
.. code-block:: scheme

   ;; length<=1
   ;; 计算元素个数为1的列表长度
   (lambda (l)
     (cond
      ((null? l) 0)
      (else (add1 (length0 (cdr l))))))

   ;; 将length0替换成它的定义
   (lambda (l)
     (cond
      ((null? l) 0)
      (else (add 1 ((lambda (l)
                      (cond
                       ((null? l) 0)
                       (else (add1 (eternity (cdr l))))))
                    (cdr l))))))

.. code-block:: scheme

   ;; length<=2
   ;; 计算元素个数为1的列表长度
   (lambda (l)
     (cond
      ((null? l) 0)
      (else (add1 (length<=1 (cdr l))))))


   ;; 将length<=1替换为它的定义
   (lambda (l)
     (cond
      ((null? l) 0)
      (else (add1 ((lambda (l)
                     (cond
                      ((null? l) 0)
                      (else (add1 ((lambda (l)
                                     (cond
                                      ((null? l) 0)
                                      (else (add1 (eternity (cdr l))))))
                                   (cdr l))))))
                   (cdr l))))))

参照上面的思路，我们可以整出length<=3...length∞

但是按这种思路写出来的代码不实现，\
越到后面的函数最大，根本无法真正写出来。

我们可以重新调整一下length：
::

   根据上面的代码发现一个现象，每个length函数的大体逻辑是一样，
   每次只是eternity这个位置的代码发生了变化。

   好的，既然只有它是可变的，那么我们就将之提取出来，当作参数传入。

.. code-block:: scheme

   ;; length0
   ((lambda (length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 (length (cdr l)))))))
    eternity)

.. code-block:: scheme

   ;; length<=1
   ((lambda (f)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 (f (cdr l)))))))
    ((lambda (g)
       (lambda (l)
         (cond
          ((null? l) 0)
          (else (add1 (g (cdr l)))))))
     eternity))

.. code-block:: scheme

   ;; length<=2
   ((lambda (length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 (length (cdr l)))))))
    ((lambda (length)
       (lambda (l)
         (cond
          ((null? l) 0)
          (else (add1 (length (cdr l)))))))
     ((lambda (length)
        (lambda (l)
          (cond
           ((null? l) 0)
           (else (add1 (length (cdr l)))))))
      eternity)))

照上面的逻辑写下去，和以前没差啊，还是要写很多。

咱们再重新调整一下length：
::

   我们再观察一下上面的代码：
   (lambda (length)
        (lambda (l)
          (cond
           ((null? l) 0)
           (else (add1 (length (cdr l)))))))

   貌似就这个函数每次在重复。
   好的，我们也将它提取出来，当作参数传入。

.. code-block:: scheme

   ;; length0
   ((lambda (mk-length)
      (mk-length eternity))
    (lambda (length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 (length (cdr l))))))))

    ;; length0简化为(mk-length eternity)

.. code-block:: scheme
                
   ;; length<=1
   ((lambda (mk-length)
      (mk-length (mk-length eternity)))
    (lambda (length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 (length (cdr l))))))))

    ;; length<=1简化为(mk-length (mk-length eternity))

.. code-block:: scheme

   ;; length<=2
   ((lambda (mk-length)
      (mk-length (mk-length (mk-length eternity))))
    (lambda (length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 (length (cdr l))))))))

   ;; length<=2简化为(mk-length (mk-length (mk-length eternity)))
                
.. code-block:: scheme

   ;; length<=3
   ((lambda (mk-length)
      (mk-length (mk-length (mk-length (mk-length eternity)))))
    (lambda (length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 (length (cdr l))))))))

   ;; length<=3简化为(mk-length (mk-length (mk-length (mk-length eternity))))
   
上面的逻辑/代码比之前简单了，\
但还是没有根本解决问题，所以还需要继续调整：
::

   还是先从length0看起，
   eternity没起什么实质的作用，\
   我们就用mk-length来代替之；
   length也是，不就一个参数名么，要统一就全部统一。

.. code-block:: scheme

   ;; length0
   ;; 将eternity替换为mk-length
   ((lambda (mk-length)
      (mk-length mk-length))
    (lambda (length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 (length (cdr l))))))))

   ;; length0
   ;; 使用α-变换将length替换为mk-length
   ((lambda (mk-length)
      (mk-length mk-length))
    (lambda (mk-length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 (mk-length (cdr l))))))))

继续调整length<=1。
::

   上面的length0如果变成length<=1的话，就会走到else分支，
   走到else分支就出错了，因为mk-length本身是一个函数，然后它将自身当作参数传入。
   而(add1 (mk-length (cdr l)))却给mk-length传入了一个列表。

   所以我们简单改一下(add1 (mk-length (cdr l)))中的mk-length即可，
   让它支持计算长度不超过1的列表。

.. code-block:: scheme

   ;; length<=1
   ((lambda (mk-length)
      (mk-length mk-length))
    (lambda (mk-length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 ((mk-length eternity) (cdr l))))))))

咱们再做最后一次的调整：
::

   再将eternity替换为mk-length，然后length函数就诞生了。

   我们简单梳理一下（以计算长度为2的列表为例）：

   1. (mk-length mk-length)这个操作生成一个计算列表长度的函数length<=2。

   2. 由于列表不为空，它会走到else分支。

   3. 在else分支中，又生成了一个计算列表长度的函数，并且对(cdr l)计算其长度，然后+1返回。

   4. 现在(cdr l)长度为1，然后又走到else分支。

   5. 在else分支中，又生成了一个计算列表长度的函数，并且对(cdr l)计算其长度，然后+1返回。

   6. (cdr l)长度为0，这次不走else分支了，直接返回0。

   7. 0 + 1 + 1 = 2， 最后返回2。

.. code-block:: scheme

   ;; length
   ((lambda (mk-length)
      (mk-length mk-length))
    (lambda (mk-length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 ((mk-length mk-length) (cdr l))))))))

不过还是有一个问题：

.. code-block:: scheme

   ;; 使用define的length函数定义
   (define length
     (lambda (l)
       (cond
        ((null? l) 0)
        (else (add1 (length (cdr l)))))))

这是使用define的length函数定义，但现在不用define之后，和原来的代码写法不一样了。

一眼看去，不一样的地方在 `(mk-length mk-length)` ，将之提取出来吧。

.. code-block:: scheme

   ;; length
   ((lambda (mk-length)
      (mk-length mk-length))
    (lambda (mk-length)
      ((lambda (length)
         (lambda (l)
           (cond
            ((null? l) 0)
            (else (add1 (length (cdr l)))))))
       (lambda (x)
         ((mk-length mk-length) x)))))

最里的函数貌似和原来的定义一样了，但是它陷在里面，将之提到外面来。

.. code-block:: scheme

   ;; length
   ((lambda (le)
      ((lambda (mk-length)
         (mk-length mk-length))
       (lambda (mk-length)
         (le (lambda (x)
               ((mk-length mk-length) x))))))
    (lambda (length)
      (lambda (l)
        (cond
         ((null? l) 0)
         (else (add1 (length (cdr l))))))))

好了，这次length函数的定义提取出来了，那我们得到一个额外的东西：

.. code-block:: scheme

   (lambda (le)
     ((lambda (mk-length)
        (mk-length mk-length))
      (lambda (mk-length)
        (le (lambda (x)
              ((mk-length mk-length) x))))))

就是它提供了一种能力：将某个非递归函数转换为递归形式。

我们可以给它起个名字： **应用序Y算子**

.. code-block:: scheme

   (define Y
     (lambda (le)
       ((lambda (f)
          (f f))
        (lambda (f)
          (le (lambda (x)
                (f f) x))))))

.. tip::

   关于应用序，可以参考： `Wiki: 求值策略`_

   可同时参考： `scheme下的停机问题和Y组合子`_

.. _`Wiki: 函数`: http://zh.wikipedia.org/zh-cn/%E5%87%BD%E6%95%B0
.. _`Wiki: 停机问题`: http://zh.wikipedia.org/wiki/%E5%81%9C%E6%9C%BA%E9%97%AE%E9%A2%98
.. _`如何通俗地解释停机问题（Halting Problem）？`: http://www.zhihu.com/question/20081359
.. _`Wiki: 不动点组合子`: http://zh.wikipedia.org/wiki/%E4%B8%8D%E5%8A%A8%E7%82%B9%E7%BB%84%E5%90%88%E5%AD%90
.. _`The Why of Y`: http://www.dreamsongs.com/Files/WhyOfY.pdf#search=%22The%20Why%20of%20Y%22
.. _`scheme下的停机问题和Y组合子`: http://www.cppblog.com/huaxiazhihuo/archive/2013/07/13/201689.html
.. _`Wiki: 求值策略`: http://zh.wikipedia.org/wiki/%E6%B1%82%E5%80%BC%E7%AD%96%E7%95%A5
.. _`Wiki: λ演算`: http://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97
