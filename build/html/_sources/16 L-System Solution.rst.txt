.. _16 L-System Solution:

Solving L-System Recursion
==========================

Here are step-by-step solutions to both the exercises presented at the end of :ref:`15 L-System`.

Recursive version of a Koch curve L-system
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Exercise:** Modify the iterative code for a Koch curve L-system into recursive form. Make sure that your recursion preserves the original start and end points of the order 0 fractal - that is, if we have a Koch curve that begins at ``(-500, 0)`` and ends at ``(500, 0)``, then any order of the Koch curve should do the same.

**Solution:** The first thing to do is to start small. We can simplify the problem by putting aside the whole drawing business and just generating the string using a recursive mechanism. If we can get the right theorem, then we should be able to draw the shape without a problem. 

Let's go back to Lindenmayer's original system, where the rules of production state *A —> AB* and *B —> A*. Here is our original iterative code:

.. code:: python

    def lSysGenerate(s, iterations):
        for i in range(iterations):
            s = lSysCompute(s)
            print('s =', s)
        return s

    def lSysCompute(s):
        d = {'A': 'AB', 'B': 'A'}
        return ''.join([d.get(c) or c for c in s])

    axiom = 'A'
    iterations = 3
    print(lSysGenerate(axiom, iterations))

.. code-block:: text

    >>> s = AB
    >>> s = ABA
    >>> s = ABAAB
    >>> ABAAB

We can approach the problem intuitively and conservatively, by keeping the code that we know works and rewriting the code that no longer applies. If you examine the two functions, ``lSysGenerate()`` feeds the string ``s`` to ``lSysCompute()`` while ``lSysCompute()`` 'does the work'. You can already see the similarity to ``mergeSort()`` and ``sierpinski()`` where we put the recursive action in one function and fed its outputs to other functions that merged and sorted lists, or drew triangles. So let's begin by converting ``lSysGenerate()`` from an iterative to a recursive form. 

To do that, let's divide the problem into its pre- and post-recursive sections. Pre-recursively, it seems like there's really no work that needs to be done. We don't need to seed our frames with anything on the way to the base case. We just want to set up the right number of frames, so that we recurse  the correct number of times and get the correct final theorem. 

This implies that all the work should happen post-recursively. More specifically, this means that the base case returns the seed for all further computations, just as we returned ``[[]]`` for ``powerSet()``. With the base case in hand, every pass through the original code's ``for`` loop can be recursively restated as each successive frame's application of the rules of production to the base case/preceding frame.

If we've done it right, our final returned string will have accumulated all the substitutions needed for the *nth* order. (I'm taking advantage of the fact that, since the recursion is linear, 'frame' and 'order' mean pretty much the same thing as 'iterations' did in the iterative code. Therefore I'll substitute 'order' for 'iterations' from here on out.)

As for the base case, it's obviously the axiom itself. In the case of the original L-system, this is ``A``, which, along with ``order``, we pass to ``lSysGenerate()``. So with ``order`` as our counter, we keep decrementing until we get to the base case of ``order == 0``. Following our standard recursive template, we can assert:

.. code:: python

    def lSysGenerate(axiom, order):
        if order == 0:
            return axiom
        else:
            return lSysGenerate(axiom, order - 1)

    order = 3
    print(lSysGenerate('A', order)

.. code-block:: text

    >>> A

Since the function doesn't have multiple recursive calls, we know that we're dealing with a fairly simple structure here. Nevertheless, let's throw in some print-tracing to keep track of frames, along with splitting the recursive call from the ``return`` statement with ``r``:

.. code:: python

    frame = 0

    def lSysGenerate(axiom, order):
        global frame
        if order == 0:
            frame += 1
            print('base case, frame =', frame)
            print('returning', axiom)
            return axiom
        else:
            frame += 1
            print('frame =', frame)
            r = lSysGenerate(axiom, order - 1)
            frame -= 1
            print('frame =', frame)
            print('r =', r)
            return r

    print(lSysGenerate('A', 3)

.. code-block:: text

    >>> frame = 1
    >>> frame = 2
    >>> frame = 3
    >>> base case, frame = 4
    >>> returning A
    >>> frame = 3
    >>> r = A
    >>> frame = 2
    >>> r = A
    >>> frame = 1
    >>> r = A
    >>> A

We next want the opportunity to apply the rules of production to the string ``axiom`` that is being passed back to us from the base case. To do this, all we need is to call ``lSysCompute()``, and the most concise way to do it is in the return statement itself. (Calling the external function at the ``return`` statement was exactly what we did when we called the ``merge()`` function in our ``mergeSort()`` program.) 

So far we have:

.. code:: python

    def lSysGenerate(axiom, order):
        if order == 0:
            return axiom
        else:
            return lSysCompute(lSysGenerate(axiom, order - 1))

    def lSysCompute(s):
        d = {'A': 'AB', 'B': 'A'}
        return ''.join([d.get(c) or c for c in s])

    axiom = 'A'
    order = 3
    print(lSysGenerate(axiom, order))

.. code-block:: text

    >>> ABAAB

Ok, then. What about drawing? The fact is that we cannot draw the shape until we have the final string in hand, and this doesn't happen until the recursive has completed. Since the iterative code doesn't draw anything until the final theorem is generated, there is no change in the reationship between the ``draw()`` and ``lSysGenerate()`` functions.

Implementing absolute distance
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

But we are still missing a crucial part of the solution, which is the ability to draw at a scale that preserves the absolute distance described by the 0th order. With ``koch()`` and ``sierpinski()``, we linked the variable tracking the fractal's order with the length of the line that would be drawn. The higher the order, the shorter the line (or the smaller the shape). 

In the current case, we can't do anything at the base case except retrieve the axiom, and we can't draw the complete figure until we've exited the recursion. So it looks like we're asking two separate things of our code: figure out the smallest line length we need, and compute the final iteration of the L-system string. We already know (and pretty much don't have a choice) about how the latter works. The trick is to figure out where to compute the former, and how to extract it from the recursive mechanism.

If we want two values from our recursion, it makes sense to include more than just the string in the recursive function's return statement. This means that we will also have two variables in the base case's return statement. Since we'll have to return two values through the recursive cases, we'll have to be careful about what we're subjecting to computation. Recall from our expansion of Pascal's triangle, we went from wanting to pass a single list (ie, the *nth* layer of the triangle) to a list of lists (ie, all layers of the triangle up to and including the *nth* layer). The trick is that, while we wanted the entire list of lists, we only wanted to recursively operate on the last returned sublist. We'll be doing something similar here. 

So far, we've got the string-generation part down, and there's no need to mess with that. To find the smallest line length ``length``, we know that for every order, ``length`` will be recomputed successively as ``length /= 3``  . So it makes sense to conduct the division pre-recursively. Once we get down to the base case, we'll have the correct minimum value for ``length``. Now all we need to do is pass that value of ``length`` - *unchanged* - back through the recursive process and hand it to the frame that initially called the recursive function.

Here's the final code for ``lSysGenerate()``, which I'll walk through below:

.. code:: python

    def lSysGenerate(axiom, order, length):  
        if order == 0:
            return [axiom, length]
        else:
            length /= 3
            r = lSysGenerate(axiom, order - 1, length)
            return [lSysCompute(r[0]), r[1]]

We first need to include ``length`` as an argument when defining (and calling) ``lSysGenerate()``. Assuming that ``order > 0``, we skip the ``if`` block and enter the ``else`` block. There, we modify the pre-recursive portion of the else block to compute ``length /= 3``. This gets us to the point where we recursively call the function. 

Now, the point of the recursive call is to get us to the base case, with the additional requirement that we bring the successively subdivided variable ``length`` along with us. Since our function currently has the parameters ``lSysGenerate(axiom, order, length)``, we decrement ``order - 1`` and the latest value of ``length`` travels with it. 

For the base case ``order == 0``, we don't need to recompute ``length``, but we do need to capture it as part of our return statement. So we'll recast our return statement as a list, ``[axiom, length]``. In the case of the Koch curve, if we wanted order 2 and had an initial length of 1000, what should be returned is ``['A', 111]``. 

But returned as what? Here I've been a bit more explicit, as I want to emphasize the fact that we're passing a list back to the calling frame. As I've done throughout this guide, I use the variable name ``r`` as a simple placeholder, which gives us a more intuitive view of the return statement. Since ``r`` is a list that consists of the string and the minimum length, we want to apply the rules of production (by calling ``lSysCompute()``) to the first index item but leave the second one untouched. 

In the interest of compact code, you could also write:

.. code:: python

    return [lSysCompute(lSysGenerate(axiom, order - 1, length)[0]), lSysGenerate(axiom, order - 1, length)[1]]

But this is difficult to read, and also implies an additional and unnecessary computation. Be nice to other people, and make your code easy to read.

The last modification that we need to make is in the arguments of the ``draw()`` function, since we are now accessing two index items from a list - ``r[0] == theorem`` and ``r[1] == length``. Here is the complete code:

.. code:: python

    import string
    import turtle

    def lSysGenerate(axiom, order, length):  
        if order == 0:
            return [axiom, length]
        else:
            length //= 3
            r = lSysGenerate(axiom, order - 1, length)
            return [lSysCompute(r[0]), r[1]]

    def lSysCompute(theorem):
        d = {'A': 'A-A++A-A'}
        return ''.join([d.get(c) or c for c in lString])

    def draw(t, theorem, length, angle):
        for c in theorem:
            if c in string.ascii_letters:
                t.fd(length)
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
        t.setpos(-500, 0)
        t.pendown()
        t.speed(0)

        axiom = 'A'
        length = 1000
        angle = 60
        order = 4

        r = lSysGenerate(axiom, order, length)
        draw(t, r[0], r[1], angle)

        wn.exitonclick()

    main()

You can quickly test this by writing a little ``for`` loop outside of ``main()`` that overlaps each order of the curve on top of the last, using a different pen color:

.. code:: python

    def main(order):
        t = turtle.Turtle()
        wn = turtle.Screen()
        wn.bgcolor('black')

        colormap = ['red', 'orange', 'yellow', 'green', 'blue', 'violet', 'white']

        t.color(colormap[i])
        t.pensize(3)
        t.penup()
        t.setpos(-1250, -600)
        t.pendown()
        t.speed(0)

        axiom = 'A'
        length = 1000
        angle = 60

        r = lSysGenerate(axiom, order, length)
        draw(t, r[0], r[1], angle)

    order = 7
    for i in range(order):
        main(i)

You'll see that the code applies to any combination of alphabet, axiom and rules of production, although with varying and sometimes surprising results. 


Recursively finding a target string
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Exercise:** For a given L-system, find if a string ``target`` exists as part of the system after a certain number of iterations. Write a recursive solution that returns the order where ``target`` appears, as well as its index position in the string as it exists in that order.

For example, say that we want to find at what point ``target = 'BAABAABA'`` *first* appears in Lindenmayer's original system. After running the function the output should print ``target BAABAABA is at index position 6 at order 7``. If it doesn't, output should be ``target BAABAABA not found``.

**Solution:** Given the generative nature of L-systems, keep in mind that ``target`` may show up in one order but, thanks to the next application of the rules of production, be substituted out at the next! By the same token, you have to write your solution so that what's returned is the first time ``target`` is found, not the most recent.

I mention this because recursion tends to favor retrieval of values at two points: at the base case, and at the end of the entire recursive cascade. It's more difficult to capture values during recursion. One way we did this was with ``mergeSort()``, where we called an outside function for input that was then passed into the recursive cascade. We used this same technique for the solution to the first exercise above, where we called ``lSysCompute()`` at every return statement of ``lSysGenerate()``.

Here is a somewhat different approach, where we don't have to invoke another function. Instead, we'll store the values as additional parameters of the recursive function itself. Let's start with the solution we came up with for the previous exercise. Since we're only interested in finding a string in the theorem, we can use Lindenmayer's original system and dispense with a drawing component.

.. code:: python

    def lSysGenerate (axiom, order): 
        if order == 0:
            return axiom
        else:
            return lSysCompute(lSysGenerate(axiom, order - 1))
    
    def lSysCompute(lString):
        d = {'A': 'AB', 'B': 'A'}
        return ''.join([d.get(c) or c for c in lString])

    def main():
        axiom = 'A'
        order = 8
        print(lSysGenerate(axiom, order))

    main()

.. code-block:: text

    >>> ABAABABAABAABABAABABAABAABABAABAABABAABABAABAABABAABABA

Great. We already know that we have to pass another argument to ``lSysGenerate()`` - the string that we're looking for. Let's call it ``target``. 

.. code-block:: python
   :emphasize-lines: 1, 5, 15

    def lSysGenerate (axiom, order, target): 
        if order == 0:
            return axiom
        else:
            return lSysCompute(lSysGenerate(axiom, order - 1, target))

    def lSysCompute(lString):
        d = {'A': 'AB', 'B': 'A'}
        return ''.join([d.get(c) or c for c in lString])

    def main():
        axiom = 'A'
        order = 8
        target = 'BAABAABA'
        print(lSysGenerate(axiom, order, target))

We also have to unpack the ``else`` block in ``lSysGenerate()`` so that we can test for the presence of ``target``. If it's found, we want to record that, so we'll need a variable ``hit`` that we'll initially set to ``None``. This variable is also added as a parameter to the function, so that it's carried along with everything else.

.. code:: python

    def lSysGenerate (axiom, order, target):  
        hit = None
        if order == 0:
            return [axiom, hit]
        else:
            r = lSysGenerate(axiom, order - 1, target)
            if target in r[0]:
                r[1] = r[0].find(target)
            return [lSysCompute(r[0]), r[1]]

At the base case, we return a list, initially ``['A', None]``, so we know that ``r[0]`` is the string, and, if we find ``target in r[0]`` is ``True``, then the index position gets recorded in the second list item, ``r[1]``. But we're still missing the exact order (or frame, or iteration) at which point this happens. So we'll add another item to our base case's returned list, ``order``.

.. code:: python

    def lSysGenerate (axiom, order, target):  
        hit = None
        if order == 0:
            return [axiom, hit, order]
        else:
            r = lSysGenerate(axiom, order - 1, target)
            if target in r[0]:
                r[1] = r[0].find(target)
                r[2] = order
            print(r)
            return [lSysCompute(r[0]), r[1], r[2]]

I snuck in a ``print(r)`` statement to see what's going on here:

.. code-block:: text

    >>> ['A', None, 0]
    >>> ['AB', None, 0]
    >>> ['ABA', None, 0]
    >>> ['ABAAB', None, 0]
    >>> ['ABAABABA', None, 0]
    >>> ['ABAABABAABAAB', None, 0]
    >>> ['ABAABABAABAABABAABABA', 6, 7]
    >>> ['ABAABABAABAABABAABABAABAABABAABAAB', 6, 8]
    >>> ['ABAABABAABAABABAABABAABAABABAABAABABAABABAABAABABAABABA', 6, 8]

Hmmm, so we're getting the correct index position for ``target`` but the value for ``order`` isn't sticking. This is because every frame re-checks to see whether ``target`` shows up within the namespace of ``r[0]`` for that frame. We need to be able to set things up so that, once ``target`` is found, we can stop looking, and preserve both the index position and ``order`` *as they were in that frame*. There is no other way, since, unlike a loop, we can't just ``break`` out of the recursion.

To do this, we set up a variable ``flag``. Initially set to ``False``, once ``target`` is found, we re-set ``flag`` to ``True`` and integrate it as a condition for the ``if`` block. Once ``flag == True`` then the loop is never triggered again, even if ``target in r[0]`` continues to be ``True`` for succeeding frames. Here's our code so far:

.. code:: python

    def lSysGenerate (axiom, order, target, flag):  
        hit = None
        if order == 0:
            return [axiom, hit, order, flag]
        else:
            r = lSysGenerate(axiom, order - 1, target, flag)
            if target in r[0] and r[3] == False:
                r[1] = r[0].find(target)
                r[2] = order
                r[3] = True
            print(r)
            return [lSysCompute(r[0]), r[1], r[2], r[3]]

This looks pretty good! We are, however, missing one last piece. It's always important to consider edge cases - what if ``target`` is part of the axiom itself? As we have written the code so far, we will only return ``True`` for ``order == 1`` even if ``target`` is present in the 0th order. So we have to add a check at the base case. Here is the final, complete code:

.. code-block:: python

    def lSysGenerate (axiom, order, target, flag):  
        hit = None
        if order == 0:
            if target in axiom:
                hit = axiom.find(target)
                flag = True
            return [axiom, hit, order, flag]
        else:
            r = lSysGenerate(axiom, order - 1, target, flag)
            if target in r[0] and r[3] == False:
                r[1] = r[0].find(target)
                r[2] = order
                r[3] = True
            return [lSysCompute(r[0]), r[1], r[2], r[3]]

    def lSysCompute(lString):
        d = {'A': 'AB', 'B': 'A'}
        return ''.join([d.get(c) or c for c in lString])

    def main():
        axiom = 'A'
        order = 8
        target = 'BAABAABA'
        flag = False
        r = lSysGenerate(axiom, order, target, flag)
        if r[1] != None:
            print(``target``, target, 'is at index position', r[1], 'at order', r[2])
        else:
            print(``target``, target, 'not found')

    main()

.. code-block:: text

    >>> target BAABAABA is at index position 6 at order 7

This code may not be as elegant or compact as many of the high-level recursive examples. On the other hand, it shows how you can retrieve values while the recursive cascade is still unfolding. Also, by gradually building up the code, I hope I made it a little less intimidating than if I'd simply introduced the final solution straight away.