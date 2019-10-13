.. _10 Fibonacci 1:

Multiple Recursive Calls: Fibonacci Sequence, Part 1
========================================================

Introducing multiple recursive calls
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Along with factorials and the Towers of Hanoi, the Fibonacci sequence is one of the classic introductions to recursion. I've chosen to include it at a significantly later point in this guide, since Fibonacci has deep implications for understanding recursion, and particularly the efficiency of certain recursive algorithms. It shows that the most intuitive recursive solution may not be desirable, and will force us to look for better ways of implementing this technique.

Let's first look at the sequence itself, which is seeded with 0 and 1. Each succeeding Fibonacci number is the sum of the two previous ones. Written in the form of a list, the first ten Fibonacci numbers are:

.. code-block:: text

    n       0  1  2  3  4  5  6   7   8   9
    fib(n) [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

When we ask for ``fib(n)`` we are asking for the place that nth number occupies in the Fibonacci sequence, similar to if we asked for the 10th prime number, or the 6th triangular number. If we were to represent this recursively, a few things immediately stand out. 

The first is that, like ``pascal()``, we are generating the sequence by looking backwards to retrieve earlier terms that we need to perform the computation. In fact, we need to go all the way back to the base case to be able to calculate the intermediate values. 

Secondly, both 0 and 1 are the seeds needed to kick off the sequence What does this imply for a recursive solution? We've seen something similar with our function to determine palindromicity: for ``pal()``, we had to account for the fact that any string either has an even or an odd number of characters, which meant we needed two base cases. Likewise, ``fib()`` has to account for 0 and 1: 

.. code:: python

    fib(n):
        if n == 0 or n == 1:
            return n
        else:
            # some code

We return ``n`` here because if we want ``fib(0)``, we can't very well return 1, can we? (We can also write ``if n < 2: return n``, which is a bit more compact.) It's true that, as we decrement ``fib(n)``, we will always return 1, but we'll also hit ``fib(0)`` far more often than you think, as we'll see.

When considering the recursive case, let's go back to the way the sequence is generated. In terms of any number ``n > 2``, another way to say "add the previous two numbers" is "add n - 1 with n - 2". In terms of generalizing this to the function, we can write:

.. code-block:: text

    fib(n) == fib(n - 1) + fib(n - 2)

In other words, we need to consult the sequence twice in order to get to ``n``. This seems fairly straightforward if we are calculating ``fib(2)``: we just add up both base cases and return the result, which is 1.

.. code-block:: text

    n       0  1      0  1  2  
    fib(n) [0, 1] —> [0, 1, 1]

Adding the next term in a sequence is always easy if you have the formula and all the preceding values. But what if we wanted to know ``fib(3)``?

.. code-block:: text

    n       0  1      0  1  2      3
    fib(n) [0, 1] —> [0, 1, ?, fib(3)]

Working from our '(n - 1) + (n - 2)' formula, to get ``fib(3)`` we first need to get ``fib(2)`` so we can add it to ``fib(1)``, which we already know is 1. And to get ``fib(2)`` we'll need to invoke ``fib(1)`` and ``fib(0)``, which we know are 1 and 0. This need to work backwards to the base case and then move forwards again suggests that the Fibonacci sequence lends itself to recursion. In fact, there's not much more to the code than what we've just reasoned through:

.. code:: python

    fib(n):
        if n == 0 or n == 1:
            return n
        else:
            return fib(n - 1) + fib(n - 2)

Recursively speaking, we go back to the base case in order to get the value we are sure of, then add up all the returned values on the trip back through the call stack. Since all of the computation happens at the moment of the recursive calls, it's appropriate to put ``fib(n - 1) + fib(n - 2)`` in the return statement. It's just a matter of backfilling the sequence until we have all of the terms needed to derive the desired nth number. 

At first glance this solution looks entirely reasonable. But a closer look at the return statement reveals quite a devil in the details. Let's map out what this might look like for ``fib(3)`` using our usual 'talking algorithm' approach:

1) Global frame: Hey ``fib()``, compute ``fib(3)`` for me.

2) Frame 1 of ``fib()``: I don't know what ``fib(3)`` is because first I need to compute ``fib(2)`` and add it to ``fib(1)`` (``return fib(2) + fib(1)``). Since ``fib(2)`` is first, let me evaluate it:

3) Frame 2 of ``fib()``: I don't know what ``fib(2)`` is because I first need to compute ``fib(1)`` and add it to ``fib(0)``: ``return fib(1) + fib(0)``. Since ``fib(1)`` comes first, let me evaluate it:

4) Frame 3 of ``fib()``: I know that ``fib(1) == 1``, since it's a base case, so I'll return the result as ``1``. Closing frame 3.

5) Frame 2 of ``fib()``: Thanks. I now have: ``return 1 + fib(0)``, so let's evaluate ``fib(0)``:

6) Frame 4 of ``fib()``: I know that ``fib(0) == 0``, since it's a base case, so I'll return the result as ``0``. Closing frame 4.

7) Frame 2 of ``fib()``: I can now complete ``return 1 + 0`` so I'm returning ``1`` to frame 1. Closing frame 2.

8) Frame 1 of ``fib()``: Ok, I now know ``fib(2) == 1`` so I can update my return statement to ``return 1 + fib(1)``, but I still can't return the answer since I now need to evaluate ``fib(1)``

9) Frame 5 of ``fib()``: I know that ``fib(1) == 1`` is a base case so I will return ``1``. Frame 5 now closed.

10) Frame 1 of ``fib()``: Ok, I now know ``fib(1) == 1`` so I can finish evaluating the return statement and close frame 1: ``return 1 + 1``

11) Global frame: I've received ``2`` from ``fib()``

That turns out to be a lot of steps to compute the next term in the sequence. Here's an illustration of the calls as they unfold, frame by frame.

.. figure:: img/fib(3).jpg
   :scale: 50 %
   :alt: alternate text
   :align: center

   Figure 1. The Fibonacci call stack for ``fib(3)``

So far we've been illustrating the call stack from the bottom up. Just as cafeteria trays are stacked on top of one another and removed in a last-in-first-out (LIFO) order, that's the way recursion creates and destroys frames. But multiple recursive calls introduce a branching structure that is conceptually closer to a tree. I find top-down diagrams much easier to read, because we've been taught to be top-down readers. So it's much more intuitive to follow the branches until we 'get to the bottom' of a chain of logic. In this case, branches are successive recursive calls, and every time the base case is achieved (and the recursive cascade reversed), we record it as a leaf.

An important part of reading these diagrams is the order in which computation happens. You can follow this by the frame numbering, keeping in mind that a computation completed at the base case will be returned to the calling frame, but that that frame may well initiate another branch of frame(s) in order to address the second recursive call in the return statement. So the program flow for the sequence of frames for ``fib(3)`` would be:

.. code-block:: text

    global-1-2-3-2-4-2-1-5-1-global

If this diagram of ``fib(3)`` seems a bit more complex than expected, things get noticeably worse for the next value of ``n``. If we were to ask for the fourth Fibonacci number, our diagram would look like this:

.. figure:: img/fib(4).jpg
   :scale: 50 %
   :alt: alternate text
   :align: center

   Figure 2. The Fibonacci call stack for ``fib(4)``

You can see that all the computations that make up ``fib(3)`` are a subset of ``fib(4)``, and that all of ``fib(2)`` is a subset of ``fib(3)``, but also that ``fib(2)`` shows up a second time in the right portion of ``fib(4)``'s tree! From a more general perspective, you can now see the return statement embodies two separate sides of the tree, where the computation of the first (left) term must be completed before computation of the second (right) term can be undertaken:

.. code-block:: text

    return fib(3) + fib(2)
           ^^^^^^   ^^^^^^
           left     right

The diagram for ``fib(4)`` points out two important consequences of multiple recursive calls. We're already familiar with the first one: once a function begins recursing, it continues until the base case. But in the case of multiple recursive calls, getting to the base case means splitting off and leaving the second (right) call for later. Of course, since the function calls itself, this means there is a split *for every call*. Likewise, what gets addressed first every time is the left term, so the recursion works its way *through every left term* until the base case is reached. Multiple recursion implies that we traverse the entire height of the tree before computing any other branches. In keeping with the top-down visualization, this is known as 'depth-first search'. 

In addition to depth-first search, the tree is gradually traversed left-to-right, which you can follow by the ``1-2-3-2-4-2-1-5-1`` sequence described above. 

It's worth pointing out that, at the moment we return to frame 1, we have recursed our way through the left-hand side of the tree. We now have the answer to the first term of ``fib(n - 1) + fib(n - 2)``. If the tree were symmetrical, we would say that we were at its midpoint. We repeat the process for the right side, continuing to move depth-first and gradually left-to-right. The last base case we hit is the leaf at the rightmost position of the diagram. 

We'll use this attribute of recursion to our advantage in the algorithms following this section.

Of course, there are several ways to think about this: you could think of it as a metaphoric tree, in which case branches spring from a trunk and each branch ends in leaves. Here you might think of it as a 'height-first' search. There are also diagrams that grow the recursive calls horizontally, such that the 'tree' is on its side, branching horizontally from left to right. However, most texts use the top-to-bottom, left-to-right metaphor.

Measuring efficiency and complexity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The second consequence of multiple recursion is even more important, at least for our need to compute larger Fibonacci numbers. It may not have been apparent from ``fib(3)``, but ``fib(4)`` shows that there is some amount of redundant computation going on, and if there's one thing that programmers despise it's redundant computation. 

Here, in the first, left-handed branch of ``fib(4)`` we compute ``fib(2)`` as part of ``fib(3)``. As we return the results to frame 1, we also find ourselves having to compute ``fib(2)`` from scratch for the second, right-handed recursive call, ``fib(n - 2)``. The fact that ``fib(2)`` is a subtree of both ``fib(3)`` and ``fib(4)`` isn't doing us any favors. You could say it's a classic case of the right hand not knowing what the left hand is doing.

The problem is only compounded when we get into double-digit Fibonacci numbers. This becomes computationally *very* expensive. In fact, try calculating ``fib(45)`` and see if you can't make and eat a sandwich while you wait. If we include a step counter that simply ticks off the number of times ``fib()`` calls itself, we see a real explosion:

.. code-block:: text

      n       fib(n)        calls
      0            0            1
      1            1            1
      2            1            3
      3            2            5
      4            3            9
      5            5           15
      6            8           25
      7           13           41
      8           21           67
      9           34          109
     10           55          177
     11           89          287
     12          144          465
     13          233          753
     14          377        1,219
     15          610        1,973
      …            …            …
     35    9,227,465   29,860,703

While dramatic, tabulating the number of calls doesn't give us much of an indication of how long this might take. Fortunately, we can further analyize our algorithms by measuring code execution speed. A good tool for this is Python's ``timeit`` module, which is designed for testing small pieces of code. To do so, first write this at the top of your code:

.. code:: python

    import timeit

Then wrap up your function call like so:

.. code:: python

    print(timeit.timeit('fib(35)', globals=globals(), number=1))

The keyword argument ``number`` sets the number of times ``timeit()`` runs the function. Since we're only interested in a rough cut, you can simply set it to ``1``.

.. note:: If you're running Python from the command line or a similar REPL session, include ``globals=globals()`` as another kwarg, so that ``timeit()`` can find the global frame. Otherwise Python will throw an error.

Our complete code for measuring ``fib()`` looks like this:

.. code:: python

    import timeit

    def fib(n):
        if n == 0 or n == 1:
            return n
        else:
            return fib(n - 1) + fib(n - 2)

    print(timeit.timeit('fib(35)', globals=globals(), number=1), 'seconds')

.. code-block:: text

    >>> 2.999023314 seconds

However, given hardware differences, my execution time will likely be different from yours. Computer science has therefore come up with a formal way of studying computational complexity, independent of hardware and such variables. I'm not going to dwell on complexity in this guide, but you can see that ``fib()`` is quite an expensive way of going about things. The complexity of what people have come to call the 'naive Fibonacci algorithm' isn't quite ``2**n``, where ``n`` is the Fibonacci number we're after - ``2**n`` would be the complexity of a perfect binary tree, where both the left and right sides have an equal number of nodes and leaves. But it's not far off either. Since the left-hand side of the tree generated by ``fib(n - 1)`` will always be larger than the right-hand side, ``fib(n - 2)``, we say that the ``fib(n)`` tree is asymmetrical. In the end, the efficiency is closer to ``1.6**n``, which is still exponential and therefore to be avoided. 

It looks like recursion has finally failed us. Sure, the code is very pretty to look at, but useless if it doesn't give the answer. On the other hand, if we could find a way to 1) store the result of each computation once we made it, and 2) refer back to that result when needed, we could potentially cut down on the number of steps. And we can, using a technique known as 'memoization', which I'll cover in the next section.

Heuristics and Exercises
^^^^^^^^^^^^^^^^^^^^^^^^

♦ Recursion may be powerful, elegant and compact, but can also yield certain solutions that are computationally unwieldy, if not inachievable. Solutions that imply returning multiple recursive calls should be approached with extreme caution.

♦ Multiple recursive calls expand call diagrams as linear 'stacks' to branching 'trees'. When following the flow of a multiply recursing function, keep in mind that the function will recurse fully to the base case of its first recursive call before evaluating any other calls. 

♦ This depth-first search is complemented by the fact that the tree will fill itself out by moving in a gradual, left-to-right direction. The final base case to be evaluated will be the rightmost leaf in the diagram.

**Exercise:** The tribonacci series is a generalization of the Fibonacci sequence where each term is the sum of the three preceding terms.

The first few terms of the sequence are:

.. code-block:: text

    [0, 0, 1, 1, 2, 4, 7, 13, 24, 44, 81, 149, 274]

The tribonacci sequence is defined as follows:

.. code-block:: text

    fn(0) == fn(1) == 1
    fn(2) == 2
    fn(n) == fn(n - 1) + fn(n - 2) + fn(n - 3)

Create a recursive function ``trib()`` that returns the tribonacci number for any given ``n``. How would you write the base case? At what point does the sequence noticeably slow down? 

Take a few minutes to draw out, by hand, the call diagram for ``trib(5)``. What does the branching look like? How would you characterize the computational complexity of a function like ``trib()`` versus that of ``fib()``?

Now measure performance by modifying both ``fib()`` and ``trib()`` to use ``timeit()``. Write a loop that will record the times for a range of values for both functions. Do the results match up with your intuition? Why or why not?