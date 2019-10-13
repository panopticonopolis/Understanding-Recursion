.. _12 Search and Sort:

Recursive Approaches To Searching And Sorting
=============================================

Programmers spend a lot of time looking for some things and sorting other things. We'll first look at how recursion can help us to find an item in a list. After that, we'll explore one of the better ways of taking a list and sorting it. This may seem backwards - after all, doesn't it make sense to sort a collection of values before you search for it? Just bear with me.

Searching by bisecting the search space
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Say I gave you a list of items ``L`` and assured you that ``L`` was sorted (ie, in ascending order). Next, I may well ask you if a given element ``e`` was in that list, and if so, where. One way is to interrogate the list item by item (ie, ``for e in L``). While this sort of exhaustive enumeration might work well for small collections, if our list had several billion items we would wind up wasting a lot of time. In the worst case - and we are always interested in the worst case - the sought-after item would occupy the final index position, or be entirely absent from the list, forcing us to examine every item in the list. 

Let's instead use the sorted nature of the list to our advantage. Assuming that ``e`` is in our sorted list ``L``, we know it must be in one of three places:

    1. In the first half of ``L``
    2. At the midpoint of ``L``, which we'll call ``mid``
    3. In the second half of ``L``

If we get lucky and find that in fact ``L[mid] == e``, then all we have to do is return ``True``, or the index position of ``e`` (that is, ``L[mid]``), depending on what's asked of us. If ``L[mid] != e``, however, the sorted nature of the list will tell us if our guess ``e`` was too low (``L[mid] > e``) or too high (``L[mid] < e``). We can then use this information to narrow our search by half again. Repeating this strategy allows us to quickly zero in on where ``e`` might be hiding (or if it's not present at all). This powerful technique is known as bisection or binary search, and belongs to the class of 'divide-and-conquer' algorithms. Since recursion is itself a form of divid-and-conquer, you can see where this is going.

To get us started, here is an iterative function, ``binSearch()``, that does the job:

.. code:: python

    def binSearch(L, e):
        high = len(L) - 1
        low = 0
        while high >= low:  
            mid = (high + low) // 2
            if L[mid] == e:
                return True
            elif L[mid] > e:
                high = mid - 1
            else:
                low = mid + 1
        return False

The ``while`` loop ensures that our comparison stops when ``high`` and ``low`` meet. Once there is no daylight between the two values, we know that ``e`` isn't in the list, and we jump out to ``return False``. Otherwise we continue to reduce the search space by half with every iteration. 

**Question:** If our search space (ie, ``len(L)``) were 100 items long, how many bisections are needed to conclude that ``e`` is not in ``L``? What if ``len(L) == 1000``? Can you abstract this into a general measurement for the efficiency of binary search, versus exhaustive enumeration? This is a good insight into how computational complexity can be measured, independently of hardware or other variables.

To think about binary search recursively, let's begin with the base case: as successive halvings of the list cause ``len(L)`` to approach zero, we will either find or not find ``e``. So we have two base cases:

.. code:: python

    def binSearchRecur1(L, e):
        if L == []:
            return False
        elif len(L) == 1:
            return L[0] == e
        else:
            # lots of code

This approach elegantly handles two edge cases: if we reach an empty list, we know we haven't found ``e``. On the other hand, if we are passed a list with only item in it, ``return L[0] == e`` returns a boolean that tells us whether ``L[0]`` is the ``e`` we're looking for. 

Let's now consider how to handle all other situations, that is, where ``len(L) > 1``. As with the iterative version, we want to capture the bisection and test it against ``e``. We base our recursive call on this evaluation, sending the halved list back to the function until we have reached one of the base cases:

.. code-block:: python
   :emphasize-lines: 6-11

    def binSearchRecur1(L, e):
        if L == []:
            return False
        elif len(L) == 1:
            return L[0] == e
        else:
            half = len(L) // 2
            if L[half] > e:
                return binSearchRecur1(L[:half], e)
            else:
                return binSearchRecur1(L[half:], e)

This looks pretty good: we have successfully covered all of our cases and are guaranteed an answer. The important stuff happens pre-recursively, in the sense that we do all the hard work on the way to the base case. Whether the base case yields ``True`` or ``False``, that's what we bring back through the recursive process, without any further alterations. Just like ``pal()``, we only want to know if the hypothesis of our original proposition (in this case, 'is ``e`` in ``L``?') holds up through the reductive process of recursion.

However, our algorithm is not as efficient as it could be. Note that in the recursive call we are sending a *copy* of the list into the next frame (any time you see a ``:`` in a list method, it means a copy has been made). Even if it's just half the size of the original list, it's still an expense that we could do without. If our initial list has a billion elements, the recursive halving to half of a billion is still 500 million elements. 

Since ``L`` is not being altered in any way, we would prefer to operate only on the original list itself. To do this, we can use variables to specify the index positions that will form the boundaries of the search space. Every time we call the function, we'll just change those index positions. In this way, we only need to keep one copy of the list in memory.

Let's add to our recursive call additional parameters for those index positions. To do this, we'll rewrite our algorithm using a helper function. You'll recall from our discussion in :ref:`02 Scope, Frame and Stack` that functions can be nested inside of other functions. These are known as inner (or helper) functions, and, by the rules of scope, all values in the enclosing function are available to the inner function. This turns out to be very useful for our current situation. Here is the complete code:

.. code:: python

    def binSearchRecur2(L, e):

        def binSearchHelp(L, e, low, high):
            if high == low:
                return L[low] == e

            mid = (low + high) // 2

            if L[mid] == e:
                return True
            elif L[mid] > e:
                if low == mid:
                    return False
                else:
                    return binSearchHelp(L, e, low, mid - 1)
            else:
                return binSearchHelp(L, e, mid + 1, high)

        if L == []:
            return False
        else:
            return binSearchHelp(L, e, 0, len(L) - 1)

Let's look at the wrapper function first. We see the same base case for the empty list as above - it should return ``False``. 

**Question:** ``elif len(L) == 1: return L[0] == e`` seems to have disappeared. Why? Can you find this functionality in the new code?

If the list isn't empty, we call the helper function ``binSearchHelp()`` and pass it ``L`` and ``e``, as well as arguments representing the bottom and top bounds of the list's range. This call only happens once, and is *not* a recursive call! As soon as we're inside the inner function, recursion takes care of the rest.

Within the helper function, the ``if high == low`` block is equivalent to the second base case in ``binSearchRecur1()``: if there is one item in the list, return a boolean evaluating ``L[low] == e`` as ``True`` or ``False``. Otherwise, we set up the midpoint ``mid`` and evaluate where ``e`` sits in relation to ``mid``. And we keep bisecting recursively until ``L[mid] == e``, or we run out of search space.

You'll notice that we've borrowed a few lines from the original iterative code. However, in the new version we don't have a ``while`` loop - we keep the program spinning by recursively calling ``binSearchHelp()`` with new arguments for ``low`` and ``high``. This is the code's most important feature: we are *not* making copies of a list but just looking at a smaller subset of the original list (``low, mid - 1`` vs ``[half:]`` and ``mid + 1, high`` vs ``[:half]``). 

While the readability of ``binSearchRecur2()`` may be more challenging in comparison to ``binSearchRecur1()``, running these two algorithms against a large, randomly generated list yields significant differences in execution. 

As with the previous section, let's use ``timeit()`` to accurately measure code execution speed. Before we can run ``timeit()``, we need to generate a big list and randomly choose an ``e`` for our algorithm to find. 

Our complete code for testing both algorithms looks like this:

.. code:: python

    import timeit
    import random

    def binSearchRecur1(L, e):
        if L == []:
            return False
        elif len(L) == 1:
            return L[0] == e
        else:
            half = len(L) // 2
            if L[half] > e:
                return binSearchRecur1(L[:half], e)
            else:
                return binSearchRecur1(L[half:], e)


    def binSearchRecur2(L, e):

        def binSearchHelp(L, e, low, high):
            if high == low:
                return L[low] == e
            mid = (low + high) // 2
            if L[mid] == e:
                return True
            elif L[mid] > e:
                if low == mid:
                    return False
                else:
                    return binSearchHelp(L, e, low, mid - 1)
            else:
                return binSearchHelp(L, e, mid + 1, high)

        if L == []:
            return False
        else:
            return binSearchHelp(L, e, 0, len(L) - 1)

    L = [i for i in range(1000000000)]   # creates list of random items
    e = random.choice(L)                 # selects one element from L

    print(timeit.timeit('binSearchRecur1(L, e)', globals=globals(), number=1), 'seconds')
    print(timeit.timeit('binSearchRecur2(L, e)', globals=globals(), number=1), 'seconds')

.. code-block:: text

    >>> 36.74825… seconds
    >>> 00.00047… seconds

While this disparity may not be as disastrous as what we saw with the Fibonacci sequence, it certainly highlights the importance of optimizing code wherever possible.

Sorting things out: ``mergeSort()``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sorting has always been a programming challenge. It seems to have fewer options than searching, and the computational costs tend to be higher and less negotiable, partly due to the fact that sorting requires you to examine every item in a list at least once. If you haven't looked at sorting algorithms yet, take some time to familiarize yourself with `bubble sort <https://runestone.academy/runestone/books/published/pythonds/SortSearch/TheBubbleSort.html>`_ and `select sort <https://runestone.academy/runestone/books/published/pythonds/SortSearch/TheSelectionSort.html>`_. That will give you a better context for the following discussion.

I'm only going to discuss merge sort, which is the algorithm that lends itself most naturally to recursion. As with the binary search approach taken above, merge sort falls into the category of divide-and-conquer algorithms. 

Once again, let's start with the simplest statement of the problem: When can we be sure that, without even inspecting it, a list is sorted? Similar to binary search's base cases, if the list is empty or has only one element, it must, by definition, already be in a sorted state. 

.. code:: python

    def mergeSort(L):
        if len(L) < 2:
            return L
        else:
            # a whole bunch of code

For all other lists where ``len(L) > 1``, we'll employ the following strategy:

1. Keep splitting the list in half (recursively) until all our sublists contain one element.

2. Merge pairs of sublists by comparing the first element of each pair, first appending the smaller element to a new, sorted list, followed by the larger one. We'll do this in a separate function.

3. If during merging one sublist of a pair runs out of elements, add all the remaining elements in the other sublist to the sorted list.

The first step shows us how to recursively arrive at our base cases. The merging will happen after the recursive call, and will do the work of actually sorting the list. The third step takes care of the edge case where a pair of (now-sorted) lists are of unequal length. What would this look like in terms of actual code? 

Let's begin at the beginning: breaking down the target list into single-item sublists. As with our searching algorithms, we want the base case to trigger when ``len(L) < 2``, at which point we return that reduced list. Here is the code so far, where we recursively split our target list into ``left`` and ``right`` halves.

.. code:: python

    def mergeSort(L):
        if len(L) < 2:
            return L
        else:
            mid = len(L) // 2
            left = mergeSort(L[:mid])
            right = mergeSort(L[mid:])
            return merge(left, right)

The return statement implies that some function ``merge()`` will do the actual merging and sorting, so let's put that aside for the moment, and better understand the recursive mechanism, and at what point sublists get sent from ``mergeSort()`` to ``merge()``.

The first thing that stands out with ``mergeSort()`` is its multiple recursive calls. As with Fibonacci, this means we have a branching call structure. Using ``L = [8, 4, 1, 6, 5, 9, 2, 0, 3]`` as our target list, here's a diagram that shows what happens on the trips towards, and back from, the base case. Note that Fibonacci was a bit lopsided, since ``fib(n - 1) + fib(n - 2)`` always meant that the left side of the tree (``fib(n - 1)``) would have more nodes and leaves than the right side (``fib(n - 2)``). ``mergeSort()`` is our first example of a symmetrical binary tree.

.. figure:: img/mergesort.jpg
   :scale: 50 %
   :alt: alternate text
   :align: center

   Figure 1. ``mergeSort()`` call tree for L = [8, 4, 1, 6, 5, 9, 2, 0, 3]


In keeping with the depth-first nature of recursion, the subdivision of the list happens until we hit the leftmost leaf in the tree. In this case:

.. code-block:: text

    frame 1 [8, 4, 1, 6, 5, 9, 2, 0, 3]
    frame 2 [8, 4, 1, 6]
    frame 3 [8, 4]

Note that this is all driven by the *first* recursive call, ``left = mergeSort(L[:mid])``. We get to the base case for the first time in frame 4, where ``len(L) < 2`` and the return assigns ``left = 8`` in frame 3. Since the namespace for ``L`` in frame 3 is ``[8, 4]``, our second recursive call, ``right = mergeSort(L[mid:])``, creates frame 5, which immediately goes to the base case, returning ``4``. Now back in frame 3, we have completed the function and are ready to send ``8`` and ``4`` as arguments to ``merge()``.

Before we do so, keep in mind that we are still deep in ``mergeSort()``'s recursive structure. Unlike the programs we have seen so far, we aren't returning values generated within ``mergeSort()`` back to ``mergeSort()``, but rather calling another function. This doesn't mean that we are done with ``mergeSort()`` - a quick glance at the tree shows that we have a long way to go yet.

For its part, ``merge()`` handles the work of putting elements of two lists - of any length - into a third list, all in the right order:

.. code:: python

    def merge(left, right):
        result = []
        i, j = 0, 0
        while i < len(left) and j < len(right):
            if left[i] < right[j]:
                result.append(left[i])
                i += 1
            else:
                result.append(right[j])
                j += 1
        while i < len(left):
            result.append(left[i])
            i += 1
        while j < len(right):
            result.append(right[j])
            j += 1
        return result

In our current case, ``[8]`` and ``[4]`` have been passed as arguments to the ``left`` and ``right`` parameters. What gets passed back to frame 3 of ``mergeSort()`` is ``result == [4, 8]``. Now that frame 3 has everything it needs, it returns this result to frame 2, which originally called it. Where does this returned list wind up? Here:

.. code:: python

    left = mergeSort(L[:mid])

Recall that in frame 2, the namespace for ``L`` is ``[8, 4, 1, 6]``. Now we have ``left = [4, 8]`` so we move on to the next line, which is ``right = mergeSort(L[mid:])``. This call creates frame 6 to handle the second half of ``L``, which is ``[1, 6]``. Frames 7 and 8 provide us with base case singleton lists, and from frame 6 we send ``[1]`` and ``[6]`` to ``merge()`` as arguments for ``left`` and ``right``. The combined and sorted list is returned to frame 6 and immediately passed up to frame 2, where it is bound to ``right`` in the statement

.. code:: python

    right = mergeSort(L[mid:])

Now we have two sorted lists for frame 2:

.. code:: python

    left = [4, 8]
    right = [1, 6]

Having reached the bottom of ``mergeSort()`` in frame 2, we repeat the same return, calling ``merge()`` with the above values for ``left`` and ``right``. 

Finally, the sorted list ``[1, 4, 6, 8]`` is passed up to frame 1, where it is assigned to that frame's namespace for ``left``. We've now completed the left side of the recursive call structure!

Of course, the right side is executed in exactly the same way. We continue to drill down to the leftmost leaf of that side, which is ``5``, sorting it against ``9``. The only additional wrinkle is when we get to frame 13 and its odd number of list items. Since ``[2, 0, 3]`` splits into ``[2]`` and ``[0, 3]``, the latter has to undergo recursion one more time. It's ok, frame 13 will wait while this gets resolved. 

Similar to our example of summing a list of lists, recursion is good at handling these sorts of situations - it will drive on until the base case is triggered. This is another reason why ``merge()`` was designed to handle lists with different lengths - if we were only expecting to receive arguments where ``len(left) == len(right)`` we could easily get either an ``Index Error`` or, worse, a digit that is omitted or returned in the wrong order.

Finally, the right side of the tree has returned its portion of the list. So now we are back at frame 1, where

.. code:: python

    left = [1, 4, 6, 8]
    right = [0, 2, 3, 5, 9]

We do the final call to ``merge()`` and, since there are no more frames, return the final sorted list to the global frame.

Here is the entire code, along with a random list generator and the ``timeit`` module:

.. code:: python

    import timeit
    import random

    def merge(left, right):
        result = []
        i, j = 0, 0
        while i < len(left) and j < len(right):
            if left[i] < right[j]:
                result.append(left[i])
                i += 1
            else:
                result.append(right[j])
                j += 1
        while i < len(left):
            result.append(left[i])
            i += 1
        while j < len(right):
            result.append(right[j])
            j += 1
        return result

    def mergeSort(L):
        if len(L) == 1:
            return L
        else:
            mid = len(L) // 2
            left = mergeSort(L[:mid])
            right = mergeSort(L[mid:])
            return merge(left, right)

    L = [random.randrange(1, 100) for x in range(1000)]

    print(timeit.timeit('mergeSort(L)', globals=globals(), number=1))

This seems like an awful lot of work for sorting a list, but it's actually more efficient than most other sorting algorithms. More importantly, sorting a collection makes searching it much more efficient. Returning to my initial remarks at the top of this guide, it's especially true if you have a large collection that you know will be searched many times but only needs to be sorted once. In that sense, the cost of sorting can be 'amortized' over the number of searches performed on the sorted set. Given a large enough number of searches, the investment in sorting becomes trivial. So, taking the time to sort may well be wortwhile.

Heuristics and Exercises
^^^^^^^^^^^^^^^^^^^^^^^^

♦ Using slices to drive recursion may be clearer to visualize but it creates copies of lists (or strings, tuples, ranges or bytes) for each frame, which can become computationally burdensome. It's preferable to allow a recursive process to repeatedly access the same, single data structure, than replicate a piece of that data structure for every recursive call. If you write a recursive solution that is executing slowly over a large collection, see if there is a way to restate the boundaries of the search space without creating unnecessary copies. 

♦ A recursive function doesn't have to be completely self-contained. We can place recursion within more complex program structures. For example, we can separate the recursive function from functions that do other, significant computational work. In the case of ``mergeSort()``, we offloaded a fairly complex merge/sort process into a separate function that we called from the return statement of the recursive function. However, keep in mind that calls to and returns from other functions are still embedded in the recursive process, which must entirely run its course.

**Exercise:** Rewrite ``binSearchRecur2`` as a single function. What are the advantages, if any, of using a helper function in this case?

**Exercise:** Here is a bubble sort algorithm, ``bubble()``. Convert it into a recursive form, ``bubbleRecur()``.

.. code:: python

    def bubble(L):

        for i in range(len(L)-1, 0, -1):

            for j in range(i):

                if L[j] > L[j + 1]:
                    temp = L[j]
                    L[j] = L[j + 1]
                    L[j + 1] = temp

        return L

**Exercise:** Here is a select sort algorithm, ``select()``. Convert it into a recursive form, ``selectRecur()``.

.. code:: python

    def select(L):

        for i in range(len(L)):
           mini = i

           for j in range(i + 1, len(L)):
               if L[mini] > L[j]:
                   mini = j
                    
           temp = L[i]
           L[i] = L[mini]
           L[mini] = temp

        return L

**Exercise:** We implemented binary search using slices, and then made it more efficient by using index positions. Merge sort also uses slices, so apply this strategy to write a function ``mergeSort2()`` that will show dramatically improved performance. 

**Exercise:** We now have six sorting algorithms:

 - ``bubble()``
 - ``bubbleRecur()``
 - ``select()``
 - ``selectRecur()``
 - ``mergeSort()`` 
 - ``mergeSort2()`` 

In a single program, run these algorithms against the same randomly generated list of numbers (your list should be pretty big - say at least 1 million items). Use ``timeit`` to get a handle on the differences and improvements in the various algorithms' execution times.

**Credit:** Much of this material adapted from pp152-164 of John Guttag's `Introduction to Computation and Programming Using Python, 2nd Ed. <https://mitpress.mit.edu/books/introduction-computation-and-programming-using-python-second-edition>`_
