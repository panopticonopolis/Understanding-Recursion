.. _08 Power Set:

Using Recursion to Make More: The Power Set
===========================================

Thinking before coding
^^^^^^^^^^^^^^^^^^^^^^

So far, we have been using recursion reductively: we get an input and distill it down to a sum, product, integer or even a boolean. We simplified a nested list into a flat list, and, in the last exercise, pulled out a minimum value from any nested list, regardless of the depth of nesting in that list. It makes sense to apply recursion to these sorts of problems, as recursion itself is based on reducing a problem down to smaller and smaller subproblems until we get to the simplest possible statement of the problem. There is a certain cognitive harmony in knowing the answer you need to get from an input, and the method used to do it, work in the same 'direction'. But what if we wanted to take an input and make *more* out of it?

Our example here is the power set, or the set of all subsets of a given set. The power set is an essential gateway towards understanding a large family of problems revolving around combinatorial optimization. As a point of clarification, in Python, a ``set`` is a collection of unique elements, whereas a ``list`` may contain duplicates. We'll use lists for our power set discussion, since a set that generates a power set doesn't have to consist of unique elements (for instance, 'apple, banana, banana, mango' just means you're working with two bananas). Also, we can leverage some of our previous experience with lists to better approach the problem. 

One thing we can establish right away is the base case. By definition, any set ``L``, is itself a member of the power set. The empty set is also always a member of the power set. Therefore we know that the power set of the empty set contains the empty set as its only member:

.. code-block:: text

    powerSet([]) == [[]]

Using our basic template, this give us the first outlines of our code:

.. code:: python

    def powerSet(L):
        if len(L) == 0:
            return [[]]
        else:
            # a bunch of code

It's always good to see how a problem behaves, independently of how we might try to solve it. If you spend some time working out a few iterations or instances of a problem, you may be able to make observations or identify patterns that will help greatly with the algorithm design. If you can write a quick snippet of code, great - but sometimes grabbing a pencil and paper is even better. Being able to reason your way to a solution from first principles is one of the best ways to learn algorithmic thinking. So before we get to the rest of the code, let's see what we can learn by developing the power set 'in real life'.

We already know that an empty set (or list - I'll use the terms interchangeably here) will always return one element - itself - as its power set. After that, we get some pretty serious expansion:

.. code-block:: text

    []                   == [[]]
    ['a']                == [[], ['a']]
    ['a', 'b']           == [[], ['a'], ['b'], ['a', 'b']]
    ['a', 'b', 'c']      == [[], ['a'], ['b'], ['a', 'b'], ['c'], ['a', 'c'], ['b', 'c'], ['a', 'b', 'c']]
    ['a', 'b', 'c', 'd'] == [[], ['a'], ['b'], ['a', 'b'], ['c'], ['a', 'c'], ['b', 'c'], ['a', 'b', 'c'], ['d'], ['a', 'd'], ['b', 'd'], ['a', 'b', 'd'], ['c', 'd'], ['a', 'c', 'd'], ['b', 'c', 'd'], ['a', 'b', 'c', 'd']]

Counting the members (sublists) of each, we see that the power set shows exponential growth, or:

.. code-block:: text

    len(powerSet([]))                   ==  1
    len(powerSet(['a']))                ==  2
    len(powerSet(['a', 'b']))           ==  4
    len(powerSet(['a', 'b', 'c']))      ==  8
    len(powerSet(['a', 'b', 'c', 'd'])) == 16
    len(powerSet(L))                    == 2**len(L)

Another interesting conclusion we can draw from the above few instances is that the power set is *additive*. If ``L == ['a', 'b', 'c']``, then its power set will contain the power set of ``L == ['a', 'b']``. Put another way, we don't know what the power set of ``L == ['a', 'b', 'c']`` is until we know the power set of ``L == ['a', 'b']``. If we asked ``powerSet()`` to compute ``L == ['a', 'b', 'c']`` we might get the following response:

1. I can't compute the power set of ``L == ['a', 'b', 'c']`` because I first need to compute the power set of ``L == ['a', 'b']``
2. I can't compute the power set of ``L == ['a', 'b']`` because I first need to compute the power set of ``L == ['a']``
3. I can't compute the power set of ``L == ['a']`` because I first need to compute the power set of ``L == []``

In other words, "I can't do 'x' until I've done 'y' and I can't do 'y' until I've done 'z'." Whenever you find yourself in a situation like this, the chances for a recursive solution are pretty good.

In that spirit, we want to reduce ``L`` until we've gotten it down to the empty set. As we did with our palindrome algorithm, we can use slices to send an ever-smaller list as an argument for each recursive call. This time, we apply slices not to the string, but to the list. For our purposes, the result is the same:

.. code:: python

    def powerSet(L):
        if len(L) == 0:
            return [[]]
        else:
            return powerSet(L[:-1])

    L = ['a', 'b', 'c', 'd']
    print('L =', L)
    print(powerSet(L))

Once at ``len(L) == 0``, we'll have the smallest statement of the problem in hand: ``[[]]``. On the way, we'll also have seeded each frame with decremented instances of ``L``. It looks like we've got a handle on at least the pre-recursive portion of the algorithm.

**Question:** Look closely at what gets returned from the base case. Is ``[[]]`` the same as ``L[:-1]`` at ``len(L) == 0``? What difference does it make? 

Let's add our usual print-tracing so we can start tracking frames. Also, it would be good to see the namespace for ``L`` in each frame:

.. code:: python

    frame = 0

    def powerSet(L):
        global frame
        frame += 1
        if len(L) == 0:
            print('\nbase case, frame', frame)
            return [[]]
        else:
            print('\npre-recursive, frame', frame)
            print('L =', L)
            return powerSet(L[:-1])

    print('global frame =', frame)
    L = ['a', 'b', 'c', 'd']
    print('L =', L)
    print(powerSet(L))

.. code-block:: text

    >>> global frame 0
    >>> L = ['a', 'b', 'c', 'd']

    >>> pre-recursive, frame 1
    >>> L = ['a', 'b', 'c', 'd']

    >>> pre-recursive, frame 2
    >>> L = ['a', 'b', 'c']

    >>> pre-recursive, frame 3
    >>> L = ['a', 'b']

    >>> pre-recursive, frame 4
    >>> L = ['a']

    >>> base case, frame 5
    >>> [[]]

The only value returned throughout the post-recursive cascade at the moment is ``[[]]``, so we'll for now we'll omit printing those returns. Nevertheless, this code should be sufficient to get us going. We now have a structure that recognizes the additive nature of the power set, but is couched in recursive terms. 

The right seed in the right frame
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's now think about what we expect to happen in each frame, and what post-recursive communication of values between frames might look like. 

Starting from the base case, we return ``[[]]``. In our example, we know that in frame 4, the namespace of ``L`` is ``['a']``. Post-recursively, we can try to create the power set for a set of two elements, ``[[]]`` and ``['a']``. If we can take that result and return it back to frame 3, we can do the same for that frame, and repeat until we have the complete set. 

Put another way, each frame has one, unique element that operates on multiple existing elements to create the permutations needed for that frame. Let's call what's coming from the called frame the ``base`` and what's waiting for it the ``operator``:

.. code-block:: text

    in frame 5:
        result   == [[]]

    in frame 4:
        base     == [[]]
        operator == ['a']
        result   == [[], ['a']]

    in frame 3:
        base     == [[], ['a']]
        operator == ['b']
        result   == [[], ['a'], ['b'], ['a', 'b']]

    in frame 2:
        base     == [[], ['a'], ['b'], ['a', 'b']]
        operator == ['c']
        result   == [[], ['a'], ['b'], ['a', 'b'], ['c'], ['a', 'c'], ['b', 'c'], ['a', 'b', 'c']]

It's clear that the result of each frame becomes the ``base`` of the frame that called it (remember, we're moving away from the base case now). So we know that ``base`` should store the results of the recursive call ``powerSet(L[:-1])``. Now we have to compute the interaction of ``base`` and ``operator``, add those results to ``base``, and return the whole thing to the next frame. 

First we'll solve for the interaction between ``base`` and ``operator``, simulating the state found in frame 4:

.. code:: python

    base = [[]]
    operator = ['a']
    next_base = base[:]
    for b in base:
        next_base.append(b + operator)

    print(next_base)

.. code-block:: text

    >>> [[[]], ['a']]

Plugging this into our existing draft of ``powerSet()``:

.. code:: python

    def powerSet(L):
        if len(L) == 0:
            return [[]]
        else:
            base = powerSet(L[:-1])
            operator = ['a']
            next_base = base[:]
            for b in base:
                next_base.append(b + operator)
            return next_base

If you run this code for ``L = [[]]``, you'll get the right answer, but for any other ``L`` it doesn't quite work. We're still missing one last piece. Recall what we left in each frame on the way to the base case:

.. code-block:: text

    >>> pre-recursive, frame 1
    >>> L = ['a', 'b', 'c', 'd']

    >>> pre-recursive, frame 2
    >>> L = ['a', 'b', 'c']

    >>> pre-recursive, frame 3
    >>> L = ['a', 'b']

    >>> pre-recursive, frame 4
    >>> L = ['a']

You can see that the last item in the list is our ``operator``. And it's easy to specify the last item in any list with ``L[-1:]``. So all we have to do is replace ``operator = ['a']`` with ``operator = L[:-1]`` to create the generalized case.

A good takeaway here is how we got to ``operator``. Decrementing the original list by slices had two purposes: getting to the base case, and setting up a situation where ``L[-1:]`` would give us the correct value for ``operator`` for every frame. In fact, it would be difficult to disentangle the two.

This indeed works, but note that we can also rewrite the ``for`` loop as a much more concise list comprehension, and insert it directly into the return statement. Once you know how the code works, it makes it much more readable:

.. code:: python

    def powerSet(L):
        if len(L) == 0:
            return [[]]
        else:
            base = powerSet(L[:-1])
            operator = L[-1:]
            return base + [(b + operator) for b in base]

    L = ['a', 'b', 'c', 'd']
    print(powerSet(L))

**Question:** Why did we write ``L[-1:]``? Isn't ``L[-1]`` sufficient?

**Question:** Can we set ``operator = L[-1:]`` before the recursive call? Why or why not?

And here is the version with complete print-tracing:

.. code:: python

    frame = 0

    def powerSet(L):
        global frame
        frame += 1
        if len(L) == 0:
            print('\nbase case, frame is', frame)
            print('returning [[]]')
            return [[]]
        else:
            print('\npre-recursive, frame', frame)
            print('list in this frame is ', L)
            print('operator in this frame is ', L[-1:])
            base = powerSet(L[:-1])
            operator = L[-1:]
            frame -= 1
            print('\npost-recursive, frame', frame)
            print('base in this frame is', base)
            print('operated on by', operator)
            r = base + [(b + operator) for b in base]
            print('returning', r)
            return r

    print('global frame is', frame)
    L = ['a', 'b', 'c', 'd']
    print('L =', L)
    print(powerSet(L))

As with most algorithms, there are many ways to go about generating the power set. I've chosen this one because I think it's a good example for how we can think recursively through the problem, step by step. But it's always instructive to look at other solutions.

From Wikipedia:

.. code:: python

    def powerSet2(L):
        if L == []:
            return [L]
        else:
            e = L[0]
            t = L[1:]
            pt = powerSet2(t)
            fept = [x + [e] for x in pt]
        return pt + fept

**Question:** How is this different in terms of the order in which the power set is constructed? What causes the difference?

There are also several iterative solutions. You may be disappointed to realize that they are quite similar to the recursive ones. One thing that the iterative solution suggests is that ``[[]]`` is a sort of mathematical MacGuffin, an excuse to get the recursive process going, but in the end it's really nothing. You wouldn't be wrong.

.. code:: python

    def powerSet3(L):
        result = [[]]
        for l in range(len(L)):
            for k in range(len(result)):
                result.append(result[k] + L[l:l + 1])
        return result

.. code:: python

    def powerSet4(L):
        result = [[]]
        for e in L:
            result.extend([subset + [e] for subset in result])
        return result

**Question:** What is the difference between the use of ``append()`` and ``extend()`` list methods? Can you rewrite ``powerSet3()`` to use ``extend()``, or ``powerSet4()`` to use ``append()``? Why or why not?

Heuristics and Exercises
^^^^^^^^^^^^^^^^^^^^^^^^

♦ Recursion doesn't have to be reductive; it can be used to multiply, expand or further elaborate an input.

♦ Before thinking about a problem in recursive terms, it can be helpful to simulate desired inputs and outputs by creating small 'modules' that will produce the desired results, and then using that to specify the return statement and computation that is internal to each frame. If you just work from the base case, the next steps may not be apparent.

♦ Pre-recursive seeding of frames has more functionality than simply decrementing to the base; it can also provide essential inputs for in-frame computation.

**Exercise:** Write a recursive function ``powerbin()`` that, given a list of unique elements, returns an additional list of binary representations of each subset of that list. For example, if the input list was:

.. code-block:: text

    L = ['a', 'b', 'c', 'd']
 
Then the following sample of subsets would be returned as:

.. code-block:: text

    ['a']           == ['1000']
    ['a', 'c', 'd'] == ['1011']
    ['b', 'd']      == ['0101']

What does this tell us about the power set and its relationship to binary counting?