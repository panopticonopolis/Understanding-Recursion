.. _06 Palindromes:

Palindromes and Recursion-as-Evaluation
=======================================

Applying recursion to strings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Recursion can also be used to simply evaluate the truth of an argument that is passed to the recursive function. With the base cases for ``summ()`` and ``factorial()``, we identified the simplest possible state of the problem, eg: the sum of all numbers from 1 to 1 is 1, and 1! == 1. When evaluating for truth, we restate our base case to answer the question: what is the most basic state of the problem that would return ``True``? On the way to the base case, we evaluate ever-smaller subsets of the problem using recursive calls. As long as each call evaluates to ``True``, we keep reaching for the base case. If we can get to the most basic solution of that problem and return ``True`` from the base case, this is thanks to the fact that every previous reduction of the problem has also been true.

For example, consider the problem of palindromicity: if a string can be read both forwards and backwards, that string is said to be a palindrome. We might say that the simplest palindrome is a single letter, for example ``a``, since ``a == a`` is certainly true. In general for humans, shorter words are quite easy to evaluate, such as ``kayak``. As a foil, we could bring up Python's ``reversed()`` method, which is effective for individual words but is quite slow, performance-wise.

.. code:: python

    'redder'  ==  ''.join(reversed('redder')  
    'driver'  !=  ''.join(reversed('driver')  

However, entire phrases such as 'Go hang a salami; I'm a lasagna hog' may be more difficult to evaluate.

So far we have been practicing recursion using numbers, which lend themselves quite well to operations like incrementing, decrementing, and hitting limits. It's a bit of a shift to focus on strings, but Python treats letters much like numbers. String slicing and indexing is a great way to isolate subsequences or individual letters in a string, and is much faster than ``reversed()`` above. So let's consider how these string methods could be used in a recursive solution.

First we decide how to apply a reductive strategy to the matter of palindromes. This is a good example of recognizing that, when considering how to design the recursive case, what you are really asking is, "How can I restate the problem so that every time I recursively call the function, the problem is smaller, and eventually I am guaranteed to reach the base case?" You're just doing it with strings and not numbers - the approach remains the same. 

For example, if our string is ``racecar`` we know that the first and last letters are the same, as are the second and second-to-last letters, etc, all the way until we get to the midpoint of the string. Since ``e`` is at the midpoint and we know that all single letters show palindromicity, we know that ``e == e`` and we're done. This is going to be easy! Recalling string index positions:

.. code-block:: text

        0 1 2 3 4 5 6
    s = r a c e c a r

Put another way, 

.. code-block:: text

    s[0] == s[6] is the same as 'r' == 'r'
    s[1] == s[5] is the same as 'a' == 'a'
    s[2] == s[4] is the same as 'c' == 'c'
    s[3] == s[3] is the same as 'e' == 'e'

We can re-write the above to take advantage of Python's indexing: increasing negative numbers count downwards from the end of the string. This way we don't care how long the string is, and we know that if we keep doing it for long enough we'll get to the midpoint:

.. code-block:: text

    s[0] == s[-1]
    s[1] == s[-2]
    s[2] == s[-3]
    s[3] == s[-4]

Already, a glimpse of the recursive case is emerging. If we can just evaluate each pair in the sequence in order, we should be able to establish a word's palindromicity. We can write out some pseudo-code to get started:

    1) Take the first and last letters of a string
    2) If they are equal proceed to the next pair of letters
    3) If they are not, return ``False``
    4) If there is only one letter left, return ``True``

How can we represent this in code? Well, if ``s[0] == s[-1]`` then what's left over? You may think that it is ``s[1:-2]`` but Python doesn't count the character at index position ``-2``, so in fact the portion of the substring still to be evaluated is ``s[1:-1]``. Note that we are *not* altering the string in any way - the immutability of strings as a data type prevents that. Nor are we making a new copy of the string. We are just looking at a sub-section of the original string, and narrowing our view two characters at a time:

.. code-block:: text

    1) s[0] == s[-1]        r     ?     r
    2) s[1] == s[-2]          a   ?   a 
    3) s[2] == s[-3]            c ? c
    4) len(s) == 1                e

Clearly, ``e`` is the midpoint, and we’ll want to stop once there, otherwise we'll just be repeating computations. If we know to stop when there is only one character that we are testing, we can re-state the limit as ``len(s) == 1``. This is all very sensible, but if we aren't mutating ``s`` how can we ever assert that ``len(s) == 1``? In the case of ``racecar``, ``len(s) == 7`` always and forever. Except our limit is literally at ``len(s[3:-4]) == 1``, so somehow we have to persuade Python that that's what we're talking about when we're talking about ``s``. This is where the recursive call works to our advantage, as we can pass slices as our argument:

.. code:: python

    def pal(s):
        if # some condition:
            # do something
        else:
            return pal(s[1:-1])

In other words, each frame only knows what it is passed to it as an argument. Let's begin with ``racecar`` in the first frame. The argument for the first recursive call is ``s[1:-1]``, so the namespace for string ``s`` in frame 2 is ``aceca``. The second recursive call opens a third frame, where the namespace for ``s`` is ``cec``, etc. This means that, if you were deep inside the recursion and expected to access the full representation of ``s == 'racecar'`` you'd be out of luck - the function in that frame only knows what's passed to it.

I'm going over this again because it's one of the tricky things about recursion. It seems at first a bit much to reconcile the statement that 1) "We aren't mutating or copying ``s``" with 2) "Each function state has its own ``s``, depending on how the recursive call modifies its passed argument." In fact, thanks to the rules of scope and frame creation, both can and must be true. There may be a bit of a cognitive leap when thinking this way about strings as opposed to numbers, but this is no different than the state of ``n`` in any given frame of ``summ()``. It may have been ``6`` in the global frame, and ``5`` in the first recursed frame, but in the next recursed frame, it was passed as ``4`` and that's what it is for that frame. 

Next, triggering the recursive call is always dependent on the truth of the condition ``if s[0] == s[-1]``. If ``s[0] != s[-1]`` at any point in our investigation, we can simply stop what we're doing and return the bad news, as there is no reason to go any further. (As a side note, a good deal of effective programming is knowing when to stop.) So let's build this branch into our code:

.. code:: python

    def pal(s):
        if # some condition:
            # do something
        else:
            if s[0] == s[-1]:
                return pal(s[1:-1])
            else:
                return False

There's another bit of elegance made possible by recursion here. Because the next ``s`` that is being passed is always snipped of its first and last elements, ``if s[0] == s[-1]`` is *always the right test* for that particular frame. There's no reason to muck about with evaluating ``if s[0] == s[-1]`` and then ``if s[1] == s[-2]`` and then ``if s[2] == s[-3]`` - the recursive call design handles everything for us correctly.

Multiple base cases
^^^^^^^^^^^^^^^^^^^

That's great, but what about the base case? Based on our discussion, it would be intuitive to say:

.. code:: python

    def pal(s):
        if len(s) == 1:
            return True
    …

This certainly works for ``racecar``, but if we try ``redder`` we get a ``IndexError: string index out of range`` error. As it turns out, strings, whether palindromic or not, come in two flavors: those that have an odd number of letters, and those with an even number. Since ``redder`` belongs to the latter, does this mean we have to change our recursive call? 

Not at all. We are far better off defining an additional base case. We know that slicing a character off either end of an odd-numbered string will leave us with a final string of 1. By the same logic, a string with an even number of characters will end up with no elements in it once the final comparison and slicing is executed:

.. code-block:: text

    1) s[0] == s[-1]        r     ?     r
    2) s[1] == s[-2]          e   ?   e 
    3) s[2] == s[-3]            d ? d
    4) len(s) == 0

That is, if ``s == 'dd'`` at step 3, then ``s[1:-1] == ''``, and ``len(s) == 0`` for that final recursive call. As you know, an empty string is a perfectly legal thing to have. Here is the final code, accounting for both base cases:

.. code:: python

    def pal(s):
        if len(s) == 1 or len(s) == 0:
            return True
        else:
            if s[0] == s[-1]:
                return pal(s[1:-1])
            else:
                return False

If we trick out our code with print statements we can see the recursion in action:

.. code:: python

    frame = 0

    def pal(s):
        global frame
        frame += 1

        if len(s) == 1 or len(s) == 0:
            print('frame =', frame, 'BASE; \t  s =', s)
            return True
        else:
            if s[0] == s[-1]:
                print('frame =', frame, 'TRUE; \t  s =', s)
                r = pal(s[1:-1])
                frame -= 1
                print('frame =', frame, 'BACK; r =', r, 's =', s)
                return r
            else:
                print('frame =', frame, 'FALSE; \t  s =', s)
                return False

If we set ``s`` to ``racecar`` we get the following output:

.. code-block:: text

    >>> frame = 1 TRUE;          s = racecar
    >>> frame = 2 TRUE;          s = aceca
    >>> frame = 3 TRUE;          s = cec
    >>> frame = 4 BASE;          s = e
    >>> frame = 3 BACK; r = True s = cec
    >>> frame = 2 BACK; r = True s = aceca
    >>> frame = 1 BACK; r = True s = racecar
    >>> True

And for ``redder`` we see:

.. code-block:: text

    >>> frame = 1 TRUE;          s = redder
    >>> frame = 2 TRUE;          s = edde
    >>> frame = 3 TRUE;          s = dd
    >>> frame = 4 BASE;          s = 
    >>> frame = 3 BACK; r = True s = dd 
    >>> frame = 2 BACK; r = True s = edde
    >>> frame = 1 BACK; r = True s = redder
    >>> True

Note that nothing prints for ``s`` in the base case frame, as ``s`` is empty by that point.

Finally, to show an example where the test fails, let's try ``ricercar``:

.. code-block:: text

    >>> frame = 1 TRUE;           s = ricercar
    >>> frame = 2 FALSE;          s = icerca
    >>> frame = 1 BACK; r = False s = ricercar
    >>> False

Notice how the recursion 'turns back' as soon as the conditional isn’t met. Thanks to the branching we put into the ``else`` block, we're not forced to go all the way to the base case, once we know we don't have to (although we still have to retrace our way through all the frames we opened pre-recursively). This is a very useful trick to know.

Our algorithm is pretty good at handling words of any length or parity, but what about phrases? As an addendum, here is some code that will clean up most text and make it legible to our pal ``pal()``. In general this is good programming practice - you want to break up functions into discrete areas of responsibility. It makes it much easier to debug, but also to reuse your code.

.. code:: python

    def clean(s):
        s = s.lower().replace(' ', '')
        s_list = list(s)
        return ''.join([c for c in s if c not in '!@#$ %^&*()_+;:,.<>?/\'\"'])

    def pal(s):
        if len(s) == 1 or len(s) == 0:
            return True
        else:
            if s[0] == s[-1]:
                return pal(s[1:-1])
            else:
                return False

    print(pal(clean('Go hang a salami; I\'m a lasagna hog.')))


There are two takeaways from this example. The first is that you may not be able to restrict yourself to a single base case. If there are two (or more) states that satisfy the most fundamental statement of the problem, then each should absolutely be part of the base case design. We will see multiple base cases play a vital role in the recursive approach to the Fibonacci sequence later on.

The second takeaway is our opportunistic approach to using recursion to get what we want. In ``summ()``, once we hit the base case, we traced our way back down the stack, adding up the numbers that were deposited in each frame, until we reached the global frame. We always knew we would reach an answer, and based on our initial ``n`` we also knew how many steps it would take. 

Similarly, in ``gcdRecur()`` we knew that, given non-zero inputs, we would always get an answer, even if we didn't know how many steps it might take. But when we got to the base case and found our answer, we simply pulled it back through the stack, all the way to the global frame, with no additional computation necessary.

With ``pal()``, we don't know if the base case is attainable at all. Indeed, our print-tracing for the third, failed case shows that once a comparison of two letters was ``False``, all we had to do was return that boolean through the stack. Obviously, that boolean could not come from the base case, so the option had to be created in the form of a conditional branch that would stop the recursion. 

If you are just beginning to work with functions, an 8-line program with no less than 3 return statements may seem like overkill. But the program flow must take into account every possibility of how a function should terminate. A function *must* exit when it encounters a return statement, and it must exit with the returned value. Therefore, using return statements is the most legible way of recognizing and tracing a function's output. However, having said all that, we still have to thread our way back through the call stack, even if there are no additional computations to be made. To see this, use ``racercar`` as an input.

Heuristics and Exercises
^^^^^^^^^^^^^^^^^^^^^^^^

Testing for palindromicity using recursion suggests to us the following heuristics: 

♦ If there is more than one most basic form of the problem, then there is more than one base case that needs to be incorporated into the code.

♦ If the purpose of the program is to evaluate the truth of an input, the base case may not be attainable, in which case there must be another way of stopping the recursive process and returning the desired value to the global frame.

**Exercise:** In the last exercise, we implemented a recursive algorithm ``itos()`` that converted an integer to a string. Now, implement a recursive function ``stoi()``, which takes a string and returns the *sum* of integer values of each character of that string.

Think carefully about how you identify your base case. What is the minimum possible value of a string, and what does it say about the string? How do you arrive at the base case? And what would you add to the base case as it works its way back through the recursed calls? Again, this problem divides neatly into base case, pre-recursive and post-recursive parts.

As before, annotate your solution with print statements that show, at each frame, the state of the function, specifying what is being passed and what is being returned, along with a counter to keep track of frames.