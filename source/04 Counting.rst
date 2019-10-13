.. _04 Counting:

Counting In Recursion
=====================

Steps vs Frames
^^^^^^^^^^^^^^^

As the exercise at the end of the previous section implied, counting in recursion is not as intuitive as it is, say, for iterative code. For the latter, it is easy to insert print statements at any point in a loop or any other construction, and quickly understand the state of computation at that point. Inserting counters and print statements into recursive code, however, yields some initially baffling results. Look at what happens if we try to print ``n`` every time the function is called:

.. code:: python

    def summ(n):
        print('n at top of summ() =', n)
        if n == 1:
            return n
        else:
            return n + summ(n - 1)

    print(summ(4))

.. code-block:: text

    >>> n at top of summ() = 4
    >>> n at top of summ() = 3
    >>> n at top of summ() = 2
    >>> n at top of summ() = 1
    >>> 10

This is fine, but doesn't really tell us anything we didn't already know. (Also, it seems like something is missing - what do you think it is?). It's more useful if we place the print statements based on where ``n`` occurs in the ``if/else`` block. Now we can see the exact value at which we hit the base case:

.. code:: python

    def summ(n):
        if n == 1:
            print('n at if =', n)
            return n
        else:
            print('n at else =', n)
            return n + summ(n - 1)

    print(summ(4))

.. code-block:: text

    >>> n at else = 4
    >>> n at else = 3
    >>> n at else = 2
    >>> n at if = 1
    >>> 10

It would be more useful still if we could count the number of frames as they are generated. We can achieve this by using the rules of scope. Recall that any variable declared in the global frame is, by default, a global variable. So let's create a variable ``frame`` that will keep track of things for us. In order for ``summ()`` to access it, we need to use the ``global`` keyword. This tells Python that the variable we are citing within the function is not a new one, but rather one that already exists in the global frame, and the interpreter should go find it there, instead of creating it within - and restricting it to the scope of - ``summ()``.

.. code:: python

    frame = 0
    n = 4
    print('at global frame =', frame, 'n =', n)

    def summ(n):
        global frame
        frame += 1
        if n == 1:
            print('base case frame =', frame, 'n =', n)
            return n
        else:
            print('recursive frame =', frame, 'n =', n)
            return n + summ(n - 1)

    print(summ(n))

.. code-block:: text

    >>> at global frame = 0 n = 4
    >>> recursive frame = 1 n = 4
    >>> recursive frame = 2 n = 3
    >>> recursive frame = 3 n = 2
    >>> base case frame = 4 n = 1
    >>> 10

Using global variables is generally regarded as a suspect practice, since you can modify the variable from inside a function. If another function is relying on that global variable to stay constant, then this can cause some unpredictable and difficult-to-trace behavior. Fortunately, in this case modifying the global variable is exactly what we want. 

Now we know how many frames there are, and the exact value of the argument ``n`` being passed from frame to frame. Things are looking hopeful. The problem with all of these examples, however, is that we are seeing only half the picture. What about after the base case, when the recursive cascade reverses and clears up all the unresolved computations? How do we access all of the post-recursive goodness we know is hiding in there?

Counting the post-recursive cascade
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It would be great to see the values of ``n`` and the reversed order of frames printed to the console as well. As it stands, we can guess that the number of steps would equal the number of pre-recursive calls where an ``else`` statement is triggered, but as we look at more complex examples of recursion, this hypothesis will become difficult to verify, to put it kindly.

The rub is in the final line:

            ``return n + summ(n - 1)``

That is, both the ``return`` statement and the recursive call occupy the same line. Each result returned from the previous frame is immediately added to ``n`` and returned again, until we reach the global frame. So there's literally no opportunity for our print statements to record any of these computations.

We can fix this by putting some daylight between the recursive call and the return with a simple little intervention:

.. code-block:: python
   :emphasize-lines: 13-16

    frame = 0
    n = 4
    print('at global frame =', frame, 'n =', n)

    def summ(n):
        global frame
        frame += 1
        if n == 1:
            print('base case frame =', frame, 'n =', n)
            return n
        else:
            print('recursive frame =', frame, 'n =', n)
            r = summ(n - 1)
            frame -= 1
            print('recursive frame =', frame, 'n =', n, 'r =', r)
            return n + r

    print(summ(n))

.. code-block:: text

    >>> at global frame = 0 n = 4
    >>> recursive frame = 1 n = 4
    >>> recursive frame = 2 n = 3
    >>> recursive frame = 3 n = 2
    >>> base case frame = 4 n = 1
    >>> recursive frame = 3 n = 2 r = 1
    >>> recursive frame = 2 n = 3 r = 3
    >>> recursive frame = 1 n = 4 r = 6
    >>> 10

By taking the returned result and binding it to a variable ``r``, we have the chance to not only print that value, but also get the correct frame. Also keep in mind that ``r`` represents the returned result from the previous, called frame - it is what is being added to ``n``, but is not what is being passed on. This is why you have to wait until the global frame to see the final answer.

This trick of breaking apart the return statement is extremely handy when you want to peek into what is being returned during the post-recursive portion of the cascade. I'll be using ``r`` (which stands for 'recursion', naturally) throughout this guide.

At first glance it may seem redundant to have two print statements in the ``else`` clause but it's not. As described at the end of :ref:`03 Frames`, the complete execution of any recursive function is split into at least two parts: what happens before the recursive call, and what happens afterwards. Here, all the print statements up to and including ``base case frame = 4 n = 1`` represent the first, pre-recursive part of the function. All the print statements *after* the spot where the ``return`` statement inserts its result represent the post-recursive part of the function’s top-to-bottom execution.

This is why we need to add ``frame -= 1`` after ``r = summ(n - 1)``. We want to capture our actions as we rewind our way back through the series of open frames. We already have  ``frame += 1`` at the top of the function, which counts every frame created on the way to the base case. If we'd continued to increment by inserting ``frame += 1`` after the recursive call, we would have gotten the total number of steps but we would have lost track of which frame we were in, hence ``frame -= 1``.

**Question:** What do you think happens if you remove ``frame -= 1`` from the code? Try it out. What's the reason for this behavior? Does it break the rules of a function's state? Why not?

By expanding the 'return + recursive call' one-liner, we can now access the complete narrative of the algorithm: the state of each frame as the recursive cascade proceeds both inward, from the global frame towards the base case frame, and back outward, as the paused computations in each open frame are completed. The code is decidedly less elegant than the original, but hopefully the printout makes it less mystifying. Knowing where to place print statements to answer specific questions will help you learn to read recursive algorithms more fluently.

Heuristics and Exercises
^^^^^^^^^^^^^^^^^^^^^^^^

♦ In recursion, counting frames is usually more important than counting steps.

♦ Being able to separate the pre-recursive and post-recursive state of a function (and the accompanying namespaces for variables) is essential to understanding how a recursive cascade unfolds.

♦ When a function's recursive call is part of the ``return`` statement, break the two apart by introducing an intermediate variable. This provides the opportunity to inspect the value actually being returned from the called frame.

**Exercise:** Revisit your factorial solution fom the previous section and apply the above techniques to get the full recursive narrative. What else can you print that would be useful information?

**Exercise:** Consider the following function ``decToBin()``, which recursively converts a decimal number to its binary equivalent. Add print statements and global variable to track frames so that you can see what the code is doing at each step.

.. code:: python

    def decToBin(n):
        if n == 0:
            return 0
        else:
            return n % 2 + 10 * decToBin(int(n / 2))

    print(decToBin(7))