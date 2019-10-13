.. _05 Swapped:

Recursion and Swapped Arguments
===============================

So far we've looked at a very simple recursive function, in the sense that ``summ()`` only had one argument that decremented regularly until we reached the base case. In a sense, this linear decrementing of ``n`` made it its own counter. We'll see this counter functionality appear consistently as a great way to manage the recursive cascade. 

Of course, functions can take multiple arguments, and we can recursively modify those arguments to reach the answer quickly, skipping many steps along the way. The next algorithm we'll look at does exactly this, in a very clever way.

Python is agnostic about how arguments can be passed to functions. As long as the function receives the correct number of arguments and can run its computation without throwing any errors, the interpreter will be happy. So depending on what your function is trying to do, you can pass an integer in place of a float, or a tuple or even a string in place of a list. This is both a strength and a weakness of the language, but here we'll only use it to our advantage.

One way to benefit from this agnostic attitude is by writing recursive calls that re-arrange the order of arguments. We will use this to great effect later on as we draw fractals and solve for the Towers of Hanoi, but let's begin with a simpler example: finding the greatest common divisor of two numbers.

The problem is simple: given two positive, non-zero numbers, what is the largest number that can divide into both? For example, given the pair (20, 12), the greatest common divisor is 4. 

We can solve for this iteratively by exhaustive enumeration: 

.. code:: python

    def gcdIter1(a, b):
        for x in range(a, -1, -1): 
            if a % x == 0 and b % x == 0: 
                return x

    print(gcdIter1(a, b))

The algorithm simply begins from ``a`` and decrements until ``x`` divides both ``a`` and ``b`` evenly. But this takes too many steps. If, for example, our pair was (2012314, 1234234) we would have to start from 2,012,234 and check each value until we reached the answer 2!

Doing it Euclid's way
^^^^^^^^^^^^^^^^^^^^^

Far better to use Euclid's algorithm, which `asserts <https://en.wikipedia.org/wiki/Euclidean_algorithm>`_ that "the greatest common divisor of two numbers does not change if the larger number is replaced by its difference with the smaller number". Using the pair (20, 12), our sequence of substitutions looks like this:

.. code-block:: text

    20 - 12 = 8
    12 -  8 = 4
     8 -  4 = 4
     4 -  4 = 0

     4

Or, for the pair (25, 20):

.. code-block:: text

    25 - 20 =  5
    20 -  5 = 15
    15 -  5 = 10
    10 -  5 =  5
     5 -  5 =  0

     5

As we iterate through the algorithm, we get our answer when the last remainder becomes equal to the largest remaining number. This final remainder is the greatest common divisor (or highest common factor, if you prefer).

In its classic form, Euclid's algorithm asks us to find the larger number so we can order the subtraction correctly - we want to avoid being left with a negative number. We can avoid this by dividing the two numbers and selecting for the remainder, using the modulo operator ``%``. In the worst case, the modulo will be the smaller number of the two. Consider the pair (12, 20):

.. code-block:: text

    12 % 20 = 12

Since 12 divided by 20 has no other remainder than itself, we flip the values of the two variables and repeat the process:

.. code-block:: text

    20 % 12 = 8
    12 %  8 = 4
     8 %  4 = 0

As with the original algorithm, we return the last number that got us to 0. Iteratively, the code is very simple:

.. code:: python

    def gcdIter2(a, b):
      while b:
        a, b = b, a % b
      return a

    print(gcdIter2(a, b))

Inserting a step counter and print statements before and after the statement inside the while loop allows us to inspect the progression of the algorithm:

.. code-block:: text

    >>> step = 1
    >>> a = 12 b = 20 a % b = 12
    >>> a is now 20 and b is now 12

    >>> step = 2 
    >>> a = 20 b = 12 a % b = 8
    >>> a is now 12 and b is now 8

    >>> step = 3
    >>> a = 12 b = 8 a % b = 4
    >>> a is now 8 and b is now 4

    >>> step = 4
    >>> a = 8 b = 4 a % b = 0
    >>> a is now 4 and b is now 0

    >>> 4    

Plugging in the pair (2012314, 1234234) shows that we can get to 2 in 13 steps, which, to put it mildly, is a vast improvement. 

Using this iterative solution as a blueprint, a recursive version is fairly straightforward. As long as ``b != 0``, swap the values ``a, b`` with ``b, a % b``, else ``return a``. This provides us with both the base and recursive cases:

.. code:: python

    def gcdRecur(a, b):
        if b == 0:
            return a
        else:
            return gcdRecur(b, a % b)

If we add print-tracing as we did with ``summ()``, we can precisely track the recursion:

.. code:: python

    frame = 0
    print('at global frame =', frame, 'a =', a, 'b =', b)
    def gcdRecur(a, b):
        global frame
        frame += 1
        if b == 0:
            print('base case frame =', frame, 'a =', a, 'b =', b)
            return a
        else:
            print('recursive frame =', frame, 'a =', a, 'b =', b)
            r = gcdRecur(b, a % b)
            frame -= 1
            print('recursive frame =', frame, 'a =', a, 'b =', b, 'gcdRecur(b, a % b) =', r)
            return r

    print(gcdRecur(a, b))

.. code-block:: text

    >>> at global frame = 0 a = 12 b = 20
    >>> recursive frame = 1 a = 12 b = 20
    >>> recursive frame = 2 a = 20 b = 12
    >>> recursive frame = 3 a = 12 b = 8
    >>> recursive frame = 4 a = 8 b = 4
    >>> base case frame = 5 a = 4 b = 0
    >>> recursive frame = 4 a = 8 b = 4 gcdRecur(b, a % b) = 4
    >>> recursive frame = 3 a = 12 b = 8 gcdRecur(b, a % b) = 4
    >>> recursive frame = 2 a = 20 b = 12 gcdRecur(b, a % b) = 4 
    >>> recursive frame = 1 a = 12 b = 20 gcdRecur(b, a % b) = 4
    >>> 4

This yields an interesting observation: once we have gotten to the base case, the returned value of ``r == 4`` never changes. Look at the way the 'return + recursive call' was originally written: 

            ``return gcdRecur(b, a % b)``

How is this different from ``summ()``, or, for that matter, your implementation of ``factorial()``? Simply put, there is no further computation occurring within the return statement. With ``return n + summ(n - 1)`` we still needed to add ``n`` from each frame, but here we just want the value of ``a`` that happened to be there the instant when ``b == 0`` became ``True``. Sometimes all we really want is what the base case tells us. In that case, no post-recursive computation is needed.

You can further see the effect of this because, in the post-recursive portion of the readout, there is no change to the namespaces of ``a`` or ``b`` in any of the frames. They remain exactly as they were seeded during the pre-recursive portion of the cascade.

Obviously, when compared with the iterative solution there is no real gain in terms of the number of steps. Both versions resolve (2012314, 1234234) in the same number of steps, if one were to equate the number of loop iterations with the number of function calls. And to be honest, from a performance point of view, the iterative example is faster. But we will see other examples where recursion is faster, and in some cases, the only possible solution.

Heuristics and Exercises
^^^^^^^^^^^^^^^^^^^^^^^^

We can add several heuristics for thinking recursively from this example. 

♦ Learn to recognize an effective formula and how to translate it into code. Sometimes it is easier to first translate the formula into an iterative format, test and understand it, and use it as a guide to implementing the recursive solution. 

♦ In addition to performing operations on arguments during the recursive call (eg, ``n - 1``, ``a % b``, we can swap those arguments as well.

♦ If the answer you need is captured by reaching the base case, there is no need to perform further computations on the return statement. Just return the desired value from the base case, and make sure that the return statement for the recursive call only carries that value back to the global frame. 

**Exercise:** Implement a recursive algorithm ``itos()`` that converts a number, digit by digit, to a string. Don't convert the entire integer to a string and return it - that's cheating! Also, the final returned result should be a single string representing the entire number. For example, if we passed the integer ``1234`` to ``itos()``, the function would return ``'1234'`` such that ``type('1234') == str``.

You can break this problem down into three parts. 

1) How do you identify your base case? 
2) The pre-recursive work: How do you get to that base case? How do you need to seed your frames on the way to the base case? 
3) The post-recursive work: What would you add to the base case as it works its way back through the recursed calls? Does the order of what is returned and what is added matter?

Annotate your solution with print statements that show, at each frame, the state of the function, specifying what is being passed and what is being returned, along with a counter that tracks the frames as they are opened and closed.