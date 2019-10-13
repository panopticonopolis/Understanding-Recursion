.. _15 L-System:

Lindenmayer Systems
===================

A different way of generating fractals
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Also known as 'rewrite' or 'L-systems', this approach was first developed in 1968 by Aristid Lindenmayer, who used L-systems to describe the behaviour of plant cells, and to model the growth processes of plant development. L-systems can create realistic-looking branching structures, as well as generate fractals in a surprisingly simple way. L-systems are also a gateway to understanding systems of propositional logic. We'll stay away from propositional logic, and just use the vector-based drawing approach to explore some of these shapes, which is perhaps more immediately gratifying. 

An L-system has three components:

1. An alphabet, consisting of two type of symbols: variables (which can be replaced) and constants (which cannot)
2. An axiom, consisting of symbols from the alphabet, which provides the starting point for replacement
3. Rule(s) of production, which show us the way(s) in which we can transform each replaceable symbol with said rule(s)

There is one more requirement for all L-systems: each step (eg, an application of the rules of production) must be comprehensive before the next step is applied. Let's use Lindenmayer's original L-system as our first example:

.. code-block:: text

    alphabet: 
        variables: A, B
        constants: none
    axiom: A
    rules: A —> AB
           B —> A

We begin with our axiom *A* at step 0. Our first iteration yields *AB* by the rule *A —> AB*. Our second iteration involves taking every replaceable element in *AB* and applying the rules of production. Since *A —> AB* and *B —> A*, our next transformation yields *AB —> ABA*. You can see that for every iteration of *n* we generate an ever-increasing string of letters:

.. code-block:: text

    n == 0: A
    n == 1: AB
    n == 2: ABA
    n == 3: ABAAB
    n == 4: ABAABABA
    n == 5: ABAABABAABAAB
    n == 6: ABAABABAABAABABAABABA
    n == 7: ABAABABAABAABABAABABAABAABABAABAAB

We can represent this in Python with a fairly trivial solution:

.. code:: python

    def lSysGenerate(s, n):
        for i in range(n):
            s = lSys(s)
        return s

    def lSysCompute(s):
        new = ''
        for c in s:
            if c == 'A':
                new += 'AB'
            elif c == 'B':
                new += 'A'
        return new

    axiom = 'A'
    n = 3
    print(lSysGenerate(axiom, n))

.. code-block:: text

    >>> ABAAB

**Question:** Instead of returning the *nth* value of the L-system, modify the return statement so that it gathers up the length of each string in a list. Can you identify the sequence? If you go out to ``n == 8`` that should be enough. Derive a formula that will generate the sequence of lengths up to any given ``n``.

So how can we use this system to generate fractals? If we feed it into our turtle program, we can say that every element in the string is a cue for the turtle to draw a segment. But if we can't turn the turtle then all we'll have is a very long, very straight line. This is where the alphabet's constants come in - we'll designate them as our angles. So if we wanted to turn right by a certain angle, we'll include a ``+``, and if we wanted to go left, we'll designate it as ``-``. Since these constants are not subject to substitution, they need to be written into the rules of production in order for them to propagate through the string.

Because we're already familiar with it, let's go back to the Koch curve. Recall how we trisect a line of length ``n``: 

1. Draw a line of length ``n // 3`` 
2. Turn left 60°
3. Draw a line of length ``n // 3``
4. Turn right 120°
5. Draw a line of length ``n // 3``
6. Turn left 60°
7. Draw a line of length ``n // 3``

We can use an L-system to go from order 0 to order 1 simply by translating this into symbols, axioms and rules of production.

For the axiom, all we need is a line, as represented by ``A``. This, by definition, is our order 0 fractal. To derive our sole rule of production, we rewrite the entire set of instructions above as ``A-A++A-A``, which is also what's required to go from order 0 to order 1. 

This implies that our alphabet consists of three symbols: the variable ``A`` and the constants ``+`` and ``-``. Finally, we write a bit of code that takes ``A``, ``-`` and ``+``, and turns them into commands that the turtle can follow:

.. code:: python

    def draw(t, s, n):
        for c in s:
            if c == 'A':
                t.fd(n)
            elif c == '-':
                t.left(60)
            elif c == '+':
                t.right(60)

Of course, we want to be able to extend this to create as many iterations of ``A`` as we like.

.. code-block:: text

    n == 0: A
    n == 1: A-A++A-A
    n == 2: A-A++A-A-A-A++A-A++A-A++A-A-A-A++A-A
    n == 3: A-A++A-A-A-A++A-A++A-A++A-A-A-A++A-A-A-A++A-A-A-A++A-A++A-A++A-A-A-A++A-A++A-A++A-A-A-A++A-A++A-A++A-A-A-A++A-A-A-A++A-A-A-A++A-A++A-A++A-A-A-A++A-A

Recall that our only variable is ``A`` - we want to parse our string in such a way that constants are unaffected, otherwise we will muck up our angles and who knows where the turtle will go. So we'll rewrite ``lSysCompute()`` using the following crisp construction:

.. code:: python

    def lSysCompute(s):
        d = {'A': 'A-A++A-A'}
        return ''.join([d.get(c) or c for c in s])

The first line in the function declares a dictionary ``d`` that has the target variable as the key, and the rule of production as its value. This is very handy, as in the future we can just substitute any dictionary we like for ``d``. If we have multiple variables, each with its own rule of production, then all we do is add a new key-value pair.

The second line is a list comprehension: if the character ``c`` in ``s`` is in the dictionary as a key, then add its value (ie, rule) as the next list item; if not, just add ``c`` as it is. Once the list is completed, it's converted to a string and returned to ``lSysGenerate()``.

Here is our complete code for generating L-system Koch curves:

.. code:: python

    import string
    import turtle

    def lSysGenerate(s, order):
        for i in range(order):
            s = lSys(s)
        return s

    def lSysCompute(s):
        d = {'A': 'A-A++A-A'}
        return ''.join([d.get(c) or c for c in s])

    def draw(t, s, length, angle):
        for c in s:
            if c in string.ascii_letters:
                t.forward(length)
            elif c == '-':
                t.left(angle)
            elif c == '+':
                t.right(angle)

    def main():
        t = turtle.Turtle()
        wn = turtle.Screen()
        wn.bgcolor('black')

        t.color('orange')
        t.pensize(1)
        t.penup()
        t.setpos(-250, -250)
        t.pendown()
        t.speed(0)

        axiom = 'A'
        length = 10
        angle = 60
        iterations = 3

        draw(t, lSysGenerate(axiom, iterations), length, angle)

        wn.exitonclick()

    main()

The only other modification to the code is in ``draw()``, where I've changed the ``if`` block to pick up on the presence of any letter, not just ``A`` - another functionality we'll need if we're to parse strings with more than one variable. (NB: don't forget to import the ``string`` module at the top of your code, otherwise Python won't know how to interpret ``string.ascii_letters``.)

Now we can generate any L-system fractal we like - it's just a matter of plugging in the right axiom, angle, length and rules of production. Try, for example, the Gosper Curve, whose whole L-system can be characterized as:

.. code-block:: text

    alphabet: 
        variables: A, B
        constants: +, -
    angle: 60°
    axiom: A
    rules: A —> A-B--B+A++AA+B-
           B —> +A-BB--B-A++A+B

Our old friend the Sierpinski triangle can be represented as well. You'll note that the axiom is, once again, the base case that we used in ``sierpinski()``:

.. code-block:: text

    alphabet: 
        variables: A, B
        constants: +, -
    angle: 120°
    axiom: A-B-B
    rules: A —> A-B--B+A++AA+B-
           B —> +A-BB--B-A++A+B


A particularly interesting shape is the Sierpinski arrowhead, which has the following traits:

.. code-block:: text

    alphabet: 
        variables: A, B
        constants: +, -
    angle: 60°
    axiom: A
    rules: A —> B+A+B
           B —> A-B-A

As you can see, the Sierpinski arrowhead has a modest beginning. While it shares the same order 0 shape as the Koch curve, it diverges quite quickly - and only looks 'right' when ``order % 2 == 0`` (that is, when ``order`` is even).

Despite the fact that the rules of production are quite simple compared to the Gosper curve and Sierpinski's triangle, if you run the arrowhead algorithm with a substantial number of iterations (eg, 8), the shape that it implies will seem quite familiar.

But hang on, you say. This may be very interesting and all, but so far there hasn't been any mention of recursion! Also, you may have noticed that, to see some of these drawings in their entirety, you've also had to revise your ``length`` and ``setpos()`` variables, as the length of an iteration doesn't scale down nicely as it has done with our implementations of ``koch()`` and ``sierpinski()``. 

I'm not trying to distract you with pretty pictures. By now you should have enough of a grasp on recursion that you can take the above code and figure out how to make it recursive - and have that recursive solution conserve the original length or perimeter of the fractal. Along the way, see if you can discover your own heuristics.

Exercises
^^^^^^^^^

**Exercise:** Modify the above code into a recursive form, using the Koch curve as an example. Make sure that your recursion preserves the original start and end points of the order 0 fractal - that is, if we have a Koch curve that begins at ``(-500, 0)`` and ends at ``(500, 0)``, then any order of the Koch curve should do the same.

**Exercise:** For a given L-system, find if a string ``target`` exists as part of the system after a given number of iterations. Write a recursive solution that returns the order where ``target`` appears, as well as the index position at which it appears.

For example, say that we want to find at what point ``target = 'BAABAABA'`` *first* appears in Lindenmayer's original system. After running the function the output should print ``target BAABAABA is at index position 6 at order 7``. If it doesn't, output should be ``target BAABAABA not found``.
