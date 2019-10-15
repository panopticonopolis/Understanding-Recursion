.. _09 Pascal:

Expanding a Series: Pascal's Triangle
=====================================

Returning the *nth* layer
^^^^^^^^^^^^^^^^^^^^^^^^^

Deriving the power set showed us that recursion could be used to expand an input at a literally exponential rate. The implementation also demonstrated the power of performing the same set of calculations on a frame-by-frame basis, and passing those results on to the next frame further down the stack. We'll extend this even further with Pascal's triangle, where we'll derive an entire series from a minimal base case.

Pascal's triangle is complex and beautiful (and pre-dates Pascal substantially). Many other sequences can be derived from it; in turn, we can calculate its values in many ways. We'll focus on deriving it from its starting point, the number 1. As always, let's look at how the triangle 'works' before we start coding.

The triangle itself can be rendered as follows:

.. code-block:: text

                              1
                             1 1
                            1 2 1
                           1 3 3 1
                          1 4 6 4 1
                        1 5 10 10 5 1
                              …

We can represent each row of the triangle as a list that has one more element than the previous one:

.. code-block:: text

    [1]                     len = 1
    [1, 1]                  len = 2
    [1, 2,  1]              len = 3
    [1, 3,  3,  1]          len = 4
    [1, 4,  6,  4,  1]      len = 5
    [1, 5, 10, 10,  5, 1]   len = 6

The method of expansion is simple: each next row is constructed by adding the number above and to the left with the number above and to the right, treating blank entries as 0. Traditionally, the first row is designated as the 0th row:

.. code-block:: text

                n        triangle

                0            1
                1         1+0 1+0
                2        1  1+1  1
                3      1  1+2 2+1  1
                             …

There is a way to calculate any nth row without knowing the value of the preceding row, but we are more interested in leveraging recursion so that we can derive the whole triangle from first principles. 

If ``n`` designates a given row of the triangle, we can decrement it until ``n == 0`` gives us the 0th row, whose value we know is 1. Following our trusty basic template, the base case practically writes itself:

.. code:: python

    def pascal(n):
        if n == 0:
            return [1]
        else:
            # a whole bunch of code

Getting from row 0 to row 1 looks a little tricky, but there's no reason why we need to deal with it immediately. As we did with ``powerSet()``, sometimes an easier next step is to model a way to get from the nth row to the (n + 1)th row, eg:

.. code-block:: text

    1 4 6 4 1 —> 1 5 10 10 5 1

In Pythonic terms, how do we get from the fourth row, call it ``n4 == [1, 4, 6, 4, 1]`` to the fifth row, ``n5 == [1, 5, 10, 10, 5, 1]``? If we design this correctly, then the algorithm should work for every value of ``n``, including the base case, since recursion mandates that a function's behavior will never change, only its inputs and state. On the other hand, it may work for all recursive cases, but not for the transition from the base case to the recursive case. Then we'll know that we need to tweak something in the base case.

To just test for the recursive case, we can set up a 'fake' recursive algorithm with the needed input, so we just have to compute the expected output as the return. Since we're not having ``pascal()`` call itself, we don't have to worry about getting tripped up if something goes wrong. It's more like a one-shot function:

.. code:: python

    def pascal(n):
        if n == 0:                # we want to skip this clause…
            return [1]
        else:
            n4 = [1, 4, 6, 4, 1]      # in place of recursive call
            n5 =                      # some computation involving n4
            return n5

    print(pascal(4))               # …so we make the passed arg > 0

If we do it correctly, ``return n5`` will give us ``[1, 5, 10, 10, 5, 1]``. We can then further test our model using ``[1, 2, 1]``; if it works, we'll get ``[1, 3, 3, 1]``, and so forth. Finally, we'll create a connection between these spot tests and the base case: can the same logic convert ``[1]`` to ``[1, 1]``? If so, we'll be well on our way towards a solution.

So what can we observe about the relationship between these two lists?

.. code-block:: text

    n4 == [1, 4, 6, 4, 1]
    n5 == [1, 5, 10, 10, 5, 1]

We know that, for ``n5``, the first term in the row is 1, so we may as well declare our list with an initial value of ``[1]``. We derive the inner terms of ``n5`` by adding consecutive pairs of terms from ``n4``. Finally, the last term of ``n5`` is again 1, making it 1 term longer than ``n4``. Here's a first draft:

.. code:: python

    def pascal(n):
        if n == 0:
            return [1]
        else:
            n4 = [1, 4, 6, 4, 1]
            n5 = [1]
            for i in range(len(n4) - 1):
                n5.append(n4[i] + n4[i + 1])
            n5.append(1)
            return n5

    print(pascal(4))

.. code-block:: text

    >>> [1, 5, 10, 10, 5, 1]

**Question:** Why are we ranging over ``len(n4) - 1`` and not ``len(n4)``?

So this is looking pretty good. Spot-testing other rows also gives us the correct values. Best of all, our little algorithm generates row 1 from the base case, that is, row 0. But before we put it all together, let's rewrite the loop as a (slightly verbose) list comprehension:

.. code:: python

    def pascal(n):
        if n == 0:
            return [1]
        else:
            n4 = [1, 3, 3, 1]
            return [1] + [(n4[i] + n4[i + 1]) for i in range(len(n4) - 1)] + [1]

    print(pascal(3))

.. code-block:: text

    >>> [1, 4, 6, 4, 1]

This restatement allows us to see, perhaps more clearly than in the ``for`` loop, why the computation of the 0th row to the first row works: 

.. code:: python

    return [1] + [(n4[i] + n4[i + 1]) for i in range(len(n4) - 1)] + [1]

We are guaranteed to return a list with first and last elements ``[1, 1]``. This is true *even if* the entire list comprehension in the middle computes to nothing (ie, an empty list), since ``[1] + [] + [1] == [1, 1]``. And this is precisely what happens when the returned value is ``[1]``, which is the base case: plugging ``[1]`` into the list comprehension yields an empty list. This is how we get from the 0th row to the 1st row, or from the base case to the first recursed frame!

Finally, if we swap out the defined input ``n4 = [1, 3, 3, 1]`` with a decrementing recursive call such as ``pascal(n - 1)`` we are close to being finished. All we have to do is update our variable names and we have our final code:

.. code:: python

    def pascal(n):
        if n == 0:
            return [1]
        else:
            r = pascal(n - 1)
            return [1] + [(r[i] + r[i + 1]) for i in range(len(r) - 1)] + [1]

    print(pascal(4))

.. code-block:: text

    >>> [1, 4, 6, 4, 1]

With full print-tracing (and a little bit of variable re-arranging, since we want to print between the calculation of ``row`` and the return statement), we have:

.. code:: python

    frame = 0
    n = 5

    def pascal(n):
        global frame
        frame += 1
        if n == 0:
            print('\nbase case frame', frame)
            print('n = 0; returning [1]')
            return [1]
        else:
            print('\npre-recursive, frame', frame)
            print('n =', n)
            r = pascal(n - 1)
            row = [1]+[(r[i]+r[i+1]) for i in range(len(r)-1)]+[1]
            frame -= 1
            print('\npost-recursive, frame', frame)
            print('n =', n)
            print('returning', row)
            return row

    print('global frame =', frame)
    print('n =', n)
    print(pascal(n))

.. code-block:: text

    >>> global frame = 0
    >>> n = 5

    >>> pre-recursive, frame 1
    >>> n = 5

    >>> pre-recursive, frame 2
    >>> n = 4

    >>> pre-recursive, frame 3
    >>> n = 3

    >>> pre-recursive, frame 4
    >>> n = 2

    >>> pre-recursive, frame 5
    >>> n = 1

    >>> base case frame = 6
    >>> n = 0; returning [1]

    >>> post-recursive, frame 5
    >>> n = 1
    >>> returning [1, 1]

    >>> post-recursive, frame 4
    >>> n = 2
    >>> returning [1, 2, 1]

    >>> post-recursive, frame 3
    >>> n = 3
    >>> returning [1, 3, 3, 1]

    >>> post-recursive, frame 2
    >>> n = 4
    >>> returning [1, 4, 6, 4, 1]

    >>> post-recursive, frame 1
    >>> n = 5
    >>> returning [1, 5, 10, 10, 5, 1]
    >>> [1, 5, 10, 10, 5, 1]

If you don't like the verbosity of the list comprehension, here is a very elegant use of the ``zip()`` and ``map()`` methods that cuts down on the clutter. Otherwise the code is exactly the same:

.. code:: python

    def pascal(n):
        if n == 1:
            row = [1]
        else:
            r = pascal(n - 1)
            pairs = zip(r[:-1], r[1:])
            row = [1] + map(sum, pairs) + [1]
        return row

Spend a few minutes with Python's documentation to figure out exactly how these two methods work. They don't do anything loops and such can't do, but they do provide a very convenient shorthand.

Returning the entire series
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Hang on a minute, though. We're not really returning the triangle, are we? We're just getting back the specific row that we asked for as ``n``. All the other rows that get computed on the way are discarded, which seems a bit of a shame.

We could set up, outside the function, a loop to append all returned values from ``pascal()`` to a list ``p``:

.. code:: python

    n = 5
    p = [pascal(n) for n in range(n)]
    print(p)

.. code-block:: text

    >>> [[1], [1, 1], [1, 2, 1], [1, 3, 3, 1], [1, 4, 6, 4, 1]]

This gives us the correct values for rows 0-4. But it's a little expensive, in the sense that we are repeating the calculations leading up to ``n = 3`` all over again in order to get to ``n = 4``, etc. Is there a way to write the recursion so that it returns the complete list?

One of the things that we can do is send a second argument to ``pascal()`` that will store all layers so far computed. We still use ``n`` to designate the last row/frame that we want, and it still works as our counter to get us down to the base case of ``if n == 0``. But we also create a list ``tri`` that scoops up every row as it is created. Here's a first draft:

.. code:: python

    def pascal(n, tri):
        if n == 0:
            return [[1]]
        else:
            r = pascal(n - 1, tri)
            row = [1] + [(r[i] + r[i + 1]) for i in range(len(r) - 1)] + [1]
            tri.append(row)
            print('tri =', tri)
        return row

    print(pascal(4, [[1]]))

.. code-block:: text

    >>> tri = [[1], [1, 1]]
    >>> tri = [[1], [1, 1], [1, 2, 1]]
    >>> tri = [[1], [1, 1], [1, 2, 1], [1, 3, 3, 1]]
    >>> tri = [[1], [1, 1], [1, 2, 1], [1, 3, 3, 1], [1, 4, 6, 4, 1]]
    >>> [1, 4, 6, 4, 1]]

The recursive call ``r = pascal(n - 1, tri)`` may look a little odd. Obviously, now that ``pascal()`` has two arguments, the interpreter requires that we pass two arguments every time we call it, but it also looks like we're mashing two values into one variable ``r``. Except we're not, because that's not what's being returned. The value returned is ``row``. If you print out ``r`` right after the recursion call, you'll see this:

.. code-block:: text

    >>> [1]
    >>> [1, 1]
    >>> [1, 2, 1]
    >>> [1, 3, 3, 1]

What you're seeing is ``row``, not ``n`` or ``tri``. Keep in mind that what we are returning to ``r`` is first the *base case*, which is ``[[1]]``, followed by each recursed value of ``row``. You may well protest that there is, in fact, an ``n``, because you can print for it and it will yield a value. That value of ``n`` you're accessing was computed on the way towards the base case and is still residing in the frame as a part of the function's state. It was there since the creation of that frame, and has nothing to do with the chain of ``return`` statements.

Also note the subtle change in the base case: we now want to return ``[[1]]`` and not ``[1]`` since we are appending lists to the base case's return value, which is itself a list whose first element is ``[1]``.

Back to our larger problem. We can see from ``tri`` that we're accumulating the rows correctly, but in the end there is nowhere for them to go, since the return statement (ie, what is returned by ``pascal(n - 1, tri)`` and bound to ``r``) must be a list that represents the row on which the new row will be based - and not a list of lists. If we have any chance of seeing the entire triangle, what we need to do is return all of ``tri``. This then means that we only want the last item in the ``tri`` list. But even if we write…

.. code:: python

        return tri[-1]

…as the return statement we get the same output as above - the last row of the triangle. 

What have to re-state the way in which we compute the row: if we are sending all of ``tri`` to ``r``, then we need to tell the function to operate on the last item of the list in ``r``, which is the most recently calculated row, in order to compute ``row``. For example, if we have been generating the whole list and at a certain point we returned…

.. code:: python

    r = [[1], [1, 1], [1, 2, 1], [1, 3, 3, 1]]

…then we know that the last element (in this case, ``[1, 3, 3, 1]``) is always represented by ``r[-1]``. We want our calculation of ``row`` to take this into account. Looking at the listcomp we built…

.. code:: python

    row = [1] + [(r[i] + r[i + 1]) for i in range(len(r) - 1)] + [1]

…it's clear that if we are applying a list of lists to this we will get a mess, if not an outright error. For example, in the first iteration, ``r[i] == [1]`` and ``r[i + 1] == [1, 1]``. Instead of operating on a single list we are mashing entire lists together. What a disaster.

Fortunately, Python allows us to specify an element that belongs to a list, even if that list is part of another, larger list:

.. code:: python

    L = [[1, 2], 3, [4, 5]]
    L[0][1]
    L[1]
    L[2][0]

.. code-block:: text

    >>> 2
    >>> 3
    >>> 4

We can integrate this into a list comprehension, rewriting the ``row`` computation as:

.. code:: python

    row = [1] + [(r[-1][i] + r[-1][i + 1]) for i in range(len(r[-1]) - 1)] + [1]

In other words, we are saying "take the ith element of the last item in ``r`` and add it to the next element of that same item in ``r``". Thanks to this tweak, our new code doesn't look that different from the original:

.. code:: python

    def pascal(n, tri):
        if n == 0:
            return [[1]]
        else:
            r = pascal(n - 1, tri)
            row = [1] + [(r[-1][i] + r[-1][i + 1]) for i in range(len(r[-1]) - 1)] + [1]
            tri.append(row)
        return tri

    print(pascal(4, [[1]]))

.. code-block:: text

    >>> [[1], [1, 1], [1, 2, 1], [1, 3, 3, 1], [1, 4, 6, 4, 1]]

I admit that this listcomp is even more verbose than the first time around, so we can also restate this in terms of the original ``for`` loop formulation:

.. code:: python

    def pascal(n, tri):
        if n == 0:
            return [[1]]
        else:
            r = pascal(n - 1, tri)
            row = [1]
            for i in range(len(r[-1]) - 1):
                row.append(r[-1][i] + r[-1][i + 1])
            row.append(1)    
            tri.append(row)
        return tri

    print(pascal(4, [[1]]))

.. code-block:: text

    >>> [[1], [1, 1], [1, 2, 1], [1, 3, 3, 1], [1, 4, 6, 4, 1]]

To see for yourself, insert a complete set of print-tracing elements and inspect how the recursion unfolds.

Heuristics and Exercises
^^^^^^^^^^^^^^^^^^^^^^^^

♦ As we did with ``powerSet()``, if you find yourself stuck for how to think through a problem recursively, solve a small portion of the problem first by creating a 'fake' recursive function. Once this one-shot function works, test it for other inputs, and then see if it works for what you chose to return from the base case.

This is very different from solving the entire problem iteratively. While an iterative approach may give you visibility into the problem's general behavior, it may not translate easily (or at all) into a recursive solution. The 'fake recursion' approach is more closely aligned with thinking recursively: we work within a function that's set up to work recursively but doesn't actually recurse. We attempt to solve for a single frame within the larger problem; by the principle of induction, we then continue testing the hypothesis. If it works for 'n', it should work for 'n + 1', 'n - 1', 'n +/- x' and, finally, 'n == 0', our base case.

♦ Sometimes the recursive call just drives to the base case and doesn't need to do anything more than that. With ``summ()`` we added the namespace of ``n`` in each frame to the returning sum. Here, ``pascal(n - 1)`` merely sets up the correct number of frames for the post-recursive cascade.

♦ What is returned by each frame and what is computed within each frame always works together. If we alter what each frame returns, we will probably have to change the computation inside each frame. But recursion demands that each frame receive the same returned variable(s), and perform the same computations. This is one of the frustrations people experience with recursion, as it can lead to situations where nothing works until everything (suddenly) works.

♦ Multiple arguments can be passed to the recursive function to create containers for more comprehensive data. If we cannot alter the way the function is being called (ie, ``pascal()`` will only accept one argument), then we can set a default parameter which in many cases will fulfill the requirement, eg: ``def pascal(n, tri=[[1]])``.

♦ Always worth re-stating: A recursive function's work is basically divisible into two parts: the pre-recursive computation and setup on the way to the base case, and the post-recursive computation, on the way back. There is no setup on the way back - you have to work with what you've got. When designing a recursive solution, you have to determine what needs to happen on the way in, and what needs to happen on the way back out. The distinct dividing line is the recursive call itself.

We've already seen two extreme examples. In ``pascal()``, all of the work happens on the return trip from the base case; this is also known as 'corecursion'. Whereas in ``pal()``, all of the work happens on the way to the base case. Recursion is flexible like that. 

**Exercise:** Building on one of the above heuristics, rewrite our last version of ``pascal()`` to use ``tri=[[1]]`` as a default argument. What else do you need to change inside and outside the function to make it work? What stays the same? What can you change that may not make a difference at all?

**Exercise:** If we examine Pascal's triangle, one of its sequences is the triangular numbers:

.. code-block:: text

              0                1
              1               1 1
              2              1 2 1
              3             1 3 3 1
              4            1 4 6 4 1
              5          1 5 10 10 5 1

One way to visualize the triangular numbers is as the number of dots needed to create an `equilateral triangle <https://en.wikipedia.org/wiki/Triangular_number#/media/File:First_six_triangular_numbers.svg>`_. If we omit 0, the sequence is as follows:

.. code-block:: text

    0, 1, 3, 6, 10, 15, 21, 28, 36…

You can see that Pascal's triangle has this sequence represented (twice!) as an interior diagonal: the 1st element of row 2, the second element of row 3, the third element of row 4, etc. Conversely, the same sequence can be read from: the last element of row 2, the second-to-last element of row 3, the third-to-last element of row 4, etc.

Modify ``pascal()`` so that it compiles the triangular numbers in a separate list and returns it along with the triangle layers. See if you can add each triangular number as each new layer is generated. Make sure that you include ``0`` at the start of your sequence of triangular numbers. 

Some hints: 
 - Get rid of the pretty formatting and left-justify the triangle to see how the triangular numbers line up.
 - If you get stuck using lists, what other data structure might be more effective?

Bonus: What if you are asked to return *only* the triangular numbers at the end of the computation, but everything must still be written as one function? 