.. _11 Fibonacci 2:

Memoization: Fibonacci Sequence, Part 2
=======================================

Memoizing by list
^^^^^^^^^^^^^^^^^

Quite simply, 'memoization' is a form of caching. Before looking at memoization for Fibonacci numbers, let's do a simpler example, one that computes factorials. From there we'll build out a series of related solutions that will get us to a clearly understandable memoized solution for ``fib()``.

For the factorial exercise in :ref:`03 Frames`, you probably came up with something like the following code:

.. code:: python

    def fact(n):
        if n == 1:
            return n
        else:
            return n * fact(n-1)

    print(fact(5))

.. code-block:: text

    >>> 120

As it stands, the program discards all the intermediate values of ``fact(n - 1)`` as it returns its way through the call stack. What if we wanted to capture those values? In a way, we already did this with our second version of ``pascal()`` in :ref:`09 Pascal`. There, we returned not just the *nth* row of the triangle, but all the rows preceding it. It took the form of a list of lists that was passed through each recursed frame, starting from the base case. We can modify ``fact()`` to do something similar:

.. code:: python

    def factMemList(n, L):
        if n == 1:
            return L
        else:
            factMemList(n - 1, L)
            L.append(n * L[-1])
            return L

    print(factMemList(5, [1]))

.. code-block:: text

    >>> [1, 2, 6, 24, 120]

Here, we want to return the entire list ``L``. So within the ``else`` block we make the recursive call, and then do some computation on the returned ``L``. You'll notice that we don't assign the recursive call to any variable! Why is that? Didn't I claim earlier that we have to explicitly update the state of a variable?

Actually, we could write ``L = factMemList(n - 1, L)`` but we don't have to, since ``factMemList(n - 1, L)`` already *is* ``L``, simply by the fact that ``L`` is what's being returned. In other words, the namespace of ``L`` for a given frame is automatically updated by the ``return L`` statement. 

For each frame, all that we are interested in getting is the right ``n`` and ``L``. We get the right ``n`` by seeding each frame with ``n - 1`` on the way to the base case, ie, pre-recursively. Once we get to the base case, we return ``L``. Post-recursively, each successive frame performs the computation ``(n * L[-1])`` and *appends it to ``L``*. Then that modified ``L`` is passed back to the next calling frame. The result is that, at each point, the correct ``n`` interacts with ``L[-1]``, which is always the last item in the list.

When designing a recursive solution, remember the three basic design decisions that need to be made. In the process of getting to the base case, how do I want to 'seed' the function's state for each frame? At what point do I 'split' the function with the recursive call? And what do I want to do post-recursively - that is, what am I returning, and what kind of computation do I want to perform on that returned object before I reach the current frame's return statement?

Our print-tracing approach tracks the recursion as follows:

.. code:: python

    frame = 0
    def factMemList(n, L):
        global frame
        frame += 1
        if n == 1:
            print('\nframe =', frame)
            print('base case:', L)
            return L
        else:
            print('\nframe =', frame)
            print('n =', n, 'L =', L)
            factMemList(n - 1, L)
            frame -= 1
            print('\nframe =', frame)
            print('n =', n, 'L =', L)
            L.append(n * L[-1])
            print('returning L as', L)
            return L

    print('global frame =', frame)
    print(factMemList(5, [1]))

.. code-block:: text

    >>> global frame = 0

    >>> frame = 1
    >>> n = 5 L = [1]

    >>> frame = 2
    >>> n = 4 L = [1]

    >>> frame = 3
    >>> n = 3 L = [1]

    >>> frame = 4
    >>> n = 2 L = [1]

    >>> frame = 5
    >>> base case: [1]

    >>> frame = 4
    >>> n = 2 L = [1]
    >>> returning L as [1, 2]

    >>> frame = 3
    >>> n = 3 L = [1, 2]
    >>> returning L as [1, 2, 6]

    >>> frame = 2
    >>> n = 4 L = [1, 2, 6]
    >>> returning L as [1, 2, 6, 24]

    >>> frame = 1
    >>> n = 5 L = [1, 2, 6, 24]
    >>> returning L as [1, 2, 6, 24, 120]
    >>> [1, 2, 6, 24, 120]

This is pretty good! We now have a complete list of factorials up to and including *n*, and every list element in index position ``L[n]`` corresponds to the *n+1th* factorial, so it's not hard to look up whatever we might need. 

Memoizing by dictionary
^^^^^^^^^^^^^^^^^^^^^^^
We can do even better, though, by using Python's ``dict`` data type. Like lists, dictionaries are mutable, so we can add to them on the fly. Unlike lists, dictionaries consist of key-value pairs. Dictionaries also require uniqueness - but only for keys. So mapping *n —> fn(n)* as key-value pairs is more readable than a list. In a list, when we append each *fn(n)* based on a continuously incrementing *n*, *n* is implied by the index position of *fn(n)*. In a dictionary, that *n* is explicitly stated as the key. (Dictionaries have the added flexibility that we can use any immutable data type as a key, but we won't need to do that here - integers are all that's required).

Let's call our dictionary ``factdict``. If ``n == 6`` and ``fact(6) == 720``, in dictionary syntax the key-value pair would read as ``{6:720}``. We populate ``factdict`` simply by declaring ``factdict[6] = 720``, creating both key and value at the same stroke. So if ``factdict`` had the key-value pair ``{1:1}`` and we wanted to compute ``fact(n)`` for some ``n``, we would write:

.. code:: python

    factdict = {1:1}
    n = 2
    factdict[n] = n * factdict[n - 1]    #value of factdict[n - 1] is 1

.. code-block:: text

    >>> {1:1, 2:2}

This simply multiplies ``n`` and the value assigned to the last key, and sets that product as a value to the new key ``n``. Iteratively, we can easily populate a dictionary with the pairs ``{n:fn(n)}``:

.. code:: python

    n = 6
    factdict = {1:1}
    for i in range(2, n + 1):
        factdict[i] = i * factdict[i - 1]
    print(factdict)

.. code-block:: text

    >>> {1: 1, 2: 2, 3: 6, 4: 24, 5: 120, 6: 720}

Here we set a new key for each ``n``, represented as ``i`` in the loop, and give it the value of ``n * factdict[n-1]``, since ``factdict[n-1]`` will always give us the value from the last available key-value pair. Since we've now expanded the dictionary by one pair, the next time the loop goes around ``facdict[i-1]`` will continue to access the most recently added key-value pair. Even though dictionaries, like lists, are unordered, we can guarantee this, because we are searching by key, and our keys always increment by 1. 

In fact, you could argue that…

.. code:: python

    factdict[i] = i * factdict[i-1]

…is just a dictionary rewrite of what we were doing with a list in ``factMemList()``:

.. code:: python

    L.append(n * L[-1])

The other bit that makes this code work is that we don't start from an empty dictionary - otherwise ``factdict[i-1]`` would throw an error. But starting with ``{1:1}`` harmonizes nicely with the fact that our base case for ``factMemList`` was ``[1]`` - both examples are variations on saying that we know ``1! == 1``, the original base case for ``fact()``.

Let's flesh out the code for ``factMemDict()``:

.. code:: python

    def factMemDict(n, factdict):
        if n == 1:
            return factdict
        else:
            factMemDict(n - 1, factdict)
            factdict[n] = n * factdict[n-1]
            return factdict

    print(factMemDict(5, {1:1}))

.. code-block:: text

    >>> {1: 1, 2: 2, 3: 6, 4: 24, 5: 120}

As you can see, except for the penultimate dictionary-specific line, it's virtually identical to ``factMemList()``! There's no reason not to split the function in exactly the same place. Pre-recursively, we wanted to seed each frame with the correct value of ``n``. Post-recursively, we wanted to return the whole dictionary, after adding a new key-value pair with the computation against the ``n`` that belongs to each frame.

As a side note, I want to point out that the code we've developed so far is a bit different from what you usually see in discussions of memoization, like this one:

.. code:: python

    def factMem(n, factdict):
        if n == 1:
            return factdict[n]
        else:
            factdict[n] = n * factMem(n - 1, factdict)
            return factdict[n]

    print(factMem(5, {1:1}))

Run this code. What's the difference? Which do you think is more useful? 

Honestly, whichever version you prefer, it doesn't look like we've achieved much. Factorial is a linearly recursing algorithm, so we can't improve its efficiency very much through memoization. But we now have a handle on what 'memoizing' an algorithm looks like. Let's get back to where it does matter - ``fib()``.

Memoizing Fibonacci
^^^^^^^^^^^^^^^^^^^

Given the Fibonacci sequence's nature, we need dictionaries for the job and not lists, as we want to be able to look up existing values quickly (via their keys). So let's convert our seeds 0 and 1 to a dictionary, ``{0:0, 1:1}``, and modify the base case accordingly:

.. code:: python

    def fibMem(n, fibdict):
        if n in fibdict:
            return fibdict
        else:
            # a bunch of code

    fibdict = {0: 0, 1: 1}

Since we are now using a dictionary, we also have to change our base case test. Rather than evaluating whether ``n == 0 or n == 1`` (or ``n < 2``), we see if ``n`` is already a key in the dictionary by asking ``if n in fibdict``. Since both 0 and 1 are in the dictionary by definition, we've got both base cases covered. Also, we are no longer returning an integer or a list, but the seed dictionary itself.

Turning to the recursive case, we want to address both the pre-recursive and post-recursive parts. For the former, we want the correct value of ``n`` to be seeded in each frame. For the latter, we want to add each ``n`` as a new key to ``fibdict``, and each freshly computed ``fibmemdict(n)`` as its corresponding value. As a one-shot, the dictionary syntax would look like this:

.. code:: python

    fibdict = {0: 0, 1: 1, 2: 1, 3: 2, 4: 3, 5: 5}
    fibdict[6] = fibdict[5] + fibdict[4]
    print(fibdict)

.. code-block:: text

    >>> {0: 0, 1: 1, 2: 1, 3: 2, 4: 3, 5: 5, 6: 8}

It's often a good idea to sketch out an iterative solution prior to the recursive one. We can look to the layout of ``factMemDict()`` to assert the iterative case:

.. code:: python

    n = 6
    fibdict = {0:0, 1:1}
    for i in range(2, n + 1):
        fibdict[i] = fibdict[i - 1] + fibdict[i - 2]
    print(fibdict)

.. code-block:: text

    >>> {0: 0, 1: 1, 2: 1, 3: 2, 4: 3, 5: 5, 6: 8}

Pulling together the final code is actually as intuitive as you might hope. The same principles of 'splitting the function' apply here, and it's simply a matter of substituting the factorial-generating dictionary operation with the Fibonacci-generating one:

.. code:: python

    def fibMem(n, fibdict):
        if n in fibdict:
            return fibdict
        else:
            fibMem(n - 1, fibdict)
            fibdict[n] = fibdict[n - 1] + fibdict[n - 2]
            return fibdict

    print(fibMem(8, {0: 0, 1: 1}))

.. code-block:: text

    >>> {0: 0, 1: 1, 2: 1, 3: 2, 4: 3, 5: 5, 6: 8, 7: 13, 8: 21}

This shows how we can incrementally build a solution to a problem that may at first seem to be difficult to approach. Instead of trying to go straight from ``fib()`` to ``fibMem()``, we stepped back, removing the distraction of ``fib()``'s multiple recursive calls, and built simple solutions for factorial, first using lists and then, based on what we learned there, dictionaries. From there, we transferred that learning to ``fibMem()``, addressing the unique attributes of the Fibonacci sequence while, at the same time, changing the fundamental structure of our algorithm only slightly. This kind of incremental development is a very useful skill indeed.

Let's contrast this with the code usually given for memoization of Fibonacci:

.. code:: python

    def fibMem2(n, fibdict):
        if n in fibdict:
            return fibdict[n]
        else:
            fibdict[n] = fibMem2(n - 1) + fibMem2(n - 2)
            return fibdict[n]

    print(fibMem2(8, {0: 0, 1: 1}))

.. code-block:: text

    >>> 21

Like the alternative example for factorial memoization, the output is not the entire dictionary, but just ``fibMem2(8)``. Fair enough, if that's all you need. And the complete dictionary is there - if you insert ``print(fibdict)`` right before the return statement you can have a peek. It's just that you can't get it out of the function - it gets discarded along with everything else once the final return of ``fibdict[n]`` occurs. In this way, it's identical to how ``tri`` was behaving when we first tried to revise ``pascal()``.

Finally, recall the table at the end of the previous section that showed just how computationally expensive ``fib()`` was. Let's add our latest results as a new column:

.. code-block:: text

                           fib(n)    fibmem()
      n       fib(n)        calls       calls
      0            0            1           1
      1            1            1           1
      2            1            3           2
      3            2            5           3
      4            3            9           4
      5            5           15           5
      6            8           25           6
      7           13           41           7
      8           21           67           8
      9           34          109           9
     10           55          177          10  
     11           89          287          11
     12          144          465          12
     13          233          753          13
     14          377        1,219          14
     15          610        1,973          15
      …            …            …           …
     35    9,227,465   29,860,703          35

As you can see, the number of times ``fibMem()`` calls itself remains steady at ``n``, no matter how large of an ``n`` we choose. In other words, its computational complexity is linear, which is vastly preferable to exponential complexity of the 'naive' version.

There are several other ways of integrating memoization into recursively expensive code. One is to leave the original recursive function untouched, and define another function to handle the memoization instead - a feature known as a 'decorator'. Since this section has gone on long enough already, you can read more about decorators `here <https://www.python-course.eu/python3_memoization.php>`_.

Heuristics and Exercises
^^^^^^^^^^^^^^^^^^^^^^^^

♦ Reduce the complexity of the problem. Can you simplify it by removing a recursive call, by simulating what the memoized data structure should look like, or by using an iterative solution to model the computation?

♦ Consider that the return statement will have to carry back the memoized data structure. What changes have to be made to the rest of the function - pre-recursive, post-recursive and the recursive call itself?

♦ [something about how the base case and the return statement can be two different things - has this shown up yet? may need to be explicit in the text re: this]

**Exercise:** The Padovan sequence is defined as follows:

.. code:: python

    f(0) == f(1) == f(2) == 1
    f(n) == f(n - 2) + f(n - 3)

The first few numbers of the sequence are:

.. code-block:: text

    [1, 1, 1, 2, 2, 3, 4, 5, 7, 9, 12, 16, 21, 28]

Create a recursive function ``padovan()`` that returns the Padovan number for any given ``n``. How would you write the base case? At what point does the sequence noticeably slow down? 

Now write a memoized version of the same algorithm, ``padovanMem()``. As with the tribonacci exercise, apply the ``timeit()`` method to see how performance improves.

**Exercise:** The Collatz conjecture is known as the "hardest simple problem in mathematics". For any positive number *n*, if *n* is even, *f(n) = n / 2*. If *n* is odd, *f(n) = 3* * *n + 1*. While it remains unproven, the conjecture states that after a finite number of operations *fn(n) = 1* for any positive number *n*.

Whether or not the conjecture is true, the number of steps needed to reduce any given ``n`` to 1 is certainly unpredictable:

.. code-block:: text

      n        steps
      1          0 
      2          1 
      3          7 
      4          2 
      5          5 
      6          8
      7         16
      8          3
      9         19
     10          6
     11         14
     12          9
     13          9
     14         17
     15         17
     16          4
     17         12
     18         20
     19         20
     20          7

Write a recursive function ``collatz()`` that returns the integer that takes the greatest number of steps to reach 1 for a given range. For example, between 1 and 1,000,000, the integer 837,799 takes 524 steps to get to 1. 

You'll see that to test for such a range takes a long amount of time. So write a second function, ``collatzMem()``, that uses memoization to reduce the computation by an order of magnitude.

This is a tricky problem. Here are some hints about how to break it down.

1. Write an iterative, brute force solution so you see how the function behaves. 
2. Write the recursive version. 
3. Go back to the iterative solution and write the memoized modification. 
4. Finally, synthesize a solution that is both recursive and memoized.