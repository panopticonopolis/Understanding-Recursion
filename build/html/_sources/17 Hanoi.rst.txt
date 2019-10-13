.. _17 Hanoi:

Boss Level: The Tower of Hanoi
==============================

Finally, we lay siege to the Tower of Hanoi. While I've studied recursion for only a brief time, I've become more and more surprised that many tutorials on the subject include this as only the third or fourth example (the other two are usually factorials and Fibonacci sequence). In this guide I've gone through dozens of algorithms, and I think the Tower problem is still very difficult to understand, partly because the problem and the solution are both easily stated. "The code is so short - how hard could it be?" are common Famous Last Words people say when they start digging into the puzzle.

The real learning, as I've tried to emphasize in every section of this guide to recursion, comes from an ability to work through the process by which we come to the solution. So for this puzzle we'll be deriving the answer from first principles, which is more beneficial (and perhaps easier) than reverse-engineering the solution. Then we'll spend some time understanding how that solution actually works - as we'll see, getting the answer and knowing how it works can be two very different things.

First, a little history. The problem was first posed by Édouard Lucas in 1883. Assume *n* number of disks are stacked by decreasing size on peg *A*. There are two other empty pegs, *B* and *C*. You have to move the disks from *A* to *C*, with the order of stacking preserved. There are two further restrictions:

1. You can only move one disk at a time
2. A larger disk can never be placed on top of a smaller disk

Lucas fancifully set the task to a group of monks in a temple, where they manually move a stack of 64 golden disks from *A* to *C*. Upon completing the task, their reward would be the end of the world. Now, executing the minimum number of moves for 64 disks, at one move per second, would take something like 585 billion years. Which means the world will be around for quite a while yet, at least if the disks have to be physically manipulated. It's a good thing they didn't use a computer to avoid any mistakes! (A good case for keeping monks away from computers is made in Arthur C. Clarke's short story `The Nine Billion Names of God <https://urbigenous.net/library/nine_billion_names_of_god.html>`_.)

Before we get started, we need to be more explicit about the rules for moving disks. Our first variation - let's call it ``simpleHanoi()`` - will allow us to jump from peg *A* to peg *C*, even if we couldn't land on peg *B* due to there being a smaller disk on *B* than the one we are moving. The second, stricter variant allows us to move disks only to a neighboring peg. We'll get to ``strictHanoi()`` after we've sorted out ``simpleHanoi()``.

Put the algorithm in your hands
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

While I'll be discussing the problem in Lucas's original terms of disks and pegs, you can physically model the Tower problem with any group of objects of increasing size - say, a stack of books. In fact, I strongly recommend that you do this right now. It's not often that you can hold the steps of an algorithm in your hands.

Let's start with a stack of *n = 3* disks as our example, using the ``simpleHanoi()`` variant. How many steps does it take? 14? 11? You should be able to get it down to 7 fairly quickly. Programmers are interested in efficiency, so we're really only concerned with the minimum number of moves.

So we can better keep track, let's also number the disks, starting with 1 for the smallest, top disk, and ending at *n* for the largest, bottom disk (the reason for doing it this way, as opposed to calling the smallest disk *n* and the largest disk 1, will become apparent when we get to the function itself). 

The first thing to notice about the 7-step solution is that, at step 4, we move disk 3 from peg *A* to peg *C*. This is the first disk that is permanently in the right spot. What's more, we only have to move it once! Also, the remaining disks are all on peg *B*. The next three moves simply stack the remaining disks on peg *C*, using *A* as a temporary peg. 

Moving on to a stack of *n = 4*, some practice should get you to a minimum number of 15 steps. Note that the same pattern obtains: at move 8, the (largest, bottom) disk 4 can jump from peg *A* to peg *C* because disks 1-3 are all on peg *B*. This allows us to hypothesize that, regardless of the size of *n*, at the midpoint of the minimum number of steps, we should be able to move disk *n* from *A* to *C*. When you think about it, the corollary that "all remaining disks must be stacked on peg *B*" is simply the logical pre-requisite to this. 

So we could say that, in order for us to move disk 4 from *A* to *C*, what we really want to do is build the stack of all disks but 1 on peg *B*. To achieve this, we have to clear the way for disk 3 to move from *A* to *B*. You can see there is a strategy slowly emerging here, or at least a way for us to know we're on the right track. 

But how do we know what the minimum number of steps is for any given *n*? Let's look at *n = 5* next. Instead of solving the entire puzzle for 5 disks, just determine how many steps you need to get to the theoretical midpoint, where disk 5 makes its only move from *A* to *C*. A bit more practice will reveal that this occurs on move 16. So by move 15, we have our stack of all remaining disks on peg *B* and disk 5 is ready to make its big jump. From there it's easy to conclude that the minimum number of moves for *n = 5* is 31. 

If we tabulate our observations for the non-strict variant of the Tower ``simpleHanoi()``, we have the following number of moves for the first few values of *n*:

.. code-block:: text

            midpoint  total
    n == 1      0        1
    n == 2      2        3
    n == 3      4        7
    n == 4      8       15
    n == 5     16       31

From this we can extrapolate that, for *n* disks, the minimum number of moves works out to ``(2**n) - 1``, and that we should hit our midpoint at ``(2**n) // 2``. It looks like we have a handle on the scope of the problem.

It's possible to extend our heuristical thinking a bit further: 

- We can't move the largest disk to peg *C* until it's the only disk on peg *A*, and peg *C* is empty
- In order for that to be true, all remaining disks must be stacked, in the correct order, on peg *B*
- In order to do that, the next-largest disk must be at the bottom of peg *B*
- To do this, we can only move that disk when it is at the top of peg *A* and peg *B* is empty
- Therefore all disks smaller than that disk must be stacked on peg *A*

You have probably already recognized this "I can't do 'x' until I do 'y', and I can't do 'y' until I do 'z'" sort of talk as a euphemism for recursion. But do we have any guidance for how the recursion might be structured? Well, we do have an important clue, which is that the minimum number of moves is ``(2**n) - 1``.

If you worked through the ``mergeSort()`` example in :ref:`12 Search and Sort`, you'll know that ``(2**n) - 1`` represents a binary tree structure. An initial root node generates two nodes of its own. In turn, each node creates two nodes of its own. This multiplication continues until something ends the tree, resulting in leaves (also known as terminal nodes).

The tree we saw in ``mergeSort()``  was created using the two recursive calls. Every time ``mergeSort()`` called itself, we would eventually open two frames, one for ``mergeSort(L[:mid])`` and another for ``mergeSort(L[mid:])``. Here's the tree and code for the example in the referenced section.

.. figure:: img/mergesort.jpg
   :scale: 50 %
   :alt: alternate text
   :align: center

   Figure 1. ``mergeSort()`` call tree for L = [8, 4, 1, 6, 5, 9, 2, 0, 3]

.. code:: python

    def mergeSort(L):
        if len(L) == 1:
            return L
        else:
            mid = len(L) // 2
            left = mergeSort(L[:mid])
            right = mergeSort(L[mid:])
            return merge(left, right)

In the case of ``mergeSort()`` the tree was created by halving the list recursively into left and right sublists, until a sublist had ``len(L) == 1``, which was the condition for 'leafiness'. With our solution to the Tower of Hanoi it will be a little more subtle. But this discussion shows that we have another piece of the puzzle: if ``simpleHanoi()`` can be modeled as a binary tree, then it will be represented in code as a function with two recursive cases.

Let's make a few more observations. The move of the next-largest (or *n - 1*) disk from peg *A* to peg *B* occurs at the midpoint between the start of the puzzle and the move where the largest disk jumps to *C* ('1 to *C*' is simply a restatement of the last move made overall).

.. code-block:: text

            n-1 to B    n to C    1 to C
    n = 1        0         0         1
    n = 2        1         2         3
    n = 3        2         4         7
    n = 4        4         8        15
    n = 5        8        16        31

And when it comes to taking disk 2 from peg *B* to its destination peg *C*, there is a similar symmetry at work:

.. code-block:: text

            n-1 to B    n to C    n-1 to C    1 to C
    n = 1        0         0           0         1
    n = 2        1         2           3         3
    n = 3        2         4           6         7
    n = 4        4         8          12        15
    n = 5        8        16          24        31

So far, we've established the minimum number of steps for a given *n* and abstracted it into a formula. We've also intuited some basic structural characteristics for a number of instances of the puzzle, all of which seem consistent. The next logical step is to look at the sequence of specific moves. We know that *n = 1* is a trivial example that can be stated as 'Move disk 1 from *A* to *C*'. Let's look at *n = 2*:

1. Move disk 1 from *A* to *B*    (disk n - 1 to *B*)
2. Move disk 2 from *A* to *C*    (disk n to *C*, midpoint)
3. Move disk 1 from *B* to *C*    (disk n - 1 to *C*)

The pattern becomes a bit clearer with *n = 3*:

1. Move disk 1 from *A* to *C*
2. Move disk 2 from *A* to *B*    (disk n - 1 to *B*)
3. Move disk 1 from *C* to *B*
4. Move disk 3 from *A* to *C*    (disk n to *C*, midpoint)
5. Move disk 1 from *B* to *A*
6. Move disk 2 from *B* to *C*    (disk n - 1 to *C*)
7. Move disk 1 from *A* to *C*

If we have *n = 4* we can see some complexity beginning to arise but our basic structure of midpoints holds:

1.  Move disk 1 from *A* to *B*
2.  Move disk 2 from *A* to *C*
3.  Move disk 1 from *B* to *C*
4.  Move disk 3 from *A* to *B*    (disk n - 1 to *B*)
5.  Move disk 1 from *C* to *A*
6.  Move disk 2 from *C* to *B*
7.  Move disk 1 from *A* to *B*
8.  Move disk 4 from *A* to *C*    (disk n to *C*, midpoint)
9.  Move disk 1 from *B* to *C*
10. Move disk 2 from *B* to *A*
11. Move disk 1 from *C* to *A*
12. Move disk 3 from *B* to *C*    (disk n - 1 to *B*)
13. Move disk 1 from *A* to *B*
14. Move disk 2 from *A* to *C*
15. Move disk 1 from *B* to *C*

We can see that there are other patterns at work here and really begin to appreciate how a ``(2**n) - 1`` tree expands. In this view, you can see that every successive value of *n* takes the previous sequence of *n - 1*, concatenates *n*, and then concatenates the *n - 1* sequence again:

.. code-block:: text

 n = 1 1
 n = 2 121
 n = 3 1213121
 n = 4 121312141213121
 n = 5 1213121412131215121312141213121

Another view shows how the series expands by always inserting a new element between every move of the preceding series. In turn, all existing values are incremented by 1:

.. code-block:: text

 n = 1                               1
 n = 2               1               2               1
 n = 3       1       2       1       3       1       2       1
 n = 4   1   2   1   3   1   2   1   4   1   2   1   3   1   2   1
 n = 5 1 2 1 3 1 2 1 4 1 2 1 3 1 2 1 5 1 2 1 3 1 2 1 4 1 2 1 3 1 2 1

Of course, these are just two different views of the same series expansion. But it helps to be able to think about things from several angles, and can also lead to insights about how to solve the problem differently.

Filling in the binary tree
^^^^^^^^^^^^^^^^^^^^^^^^^^

Since our hunch about the binary tree structure is bearing out, let's write out the steps in terms of that form. Also, by now you may have concluded that, if we stick to the minimum number of steps, there is only one possible sequence for any given *n*. This implies that there is only one correct way of populating the tree - another hint that our recursive approach will work out, since recursion is exhaustive by nature. 

As the comparison of steps shows, since 'Move disk *n* from A to C' will always occur at the midpoint, then that move occupies the 'root node' of the tree, with an equal number of moves to either side of it. Even though *n = 1* is our trivial example, it's always a good place to start:

.. figure:: img/hanoi01.jpg
   :scale: 50 %
   :alt: alternate text
   :align: center

   Figure 2. Call tree for *n = 1*

I'm using the circles to designate frames, and diamonds to show when the step actually gets executed. We know that a tree is populated from its initial, root node, and execution of a tree goes from left to right, so rendering *n = 2* isn't too difficult:

.. figure:: img/hanoi02.jpg
   :scale: 50 %
   :alt: alternate text
   :align: center

   Figure 3. Call tree for *n = 2*

For *n = 2*, both frame creation and step execution move in the same direction, so I guess you could draw this diagram as three boxes in a single line. However, things get complicated with *n = 3*, so it's more appropriate to consider the two disk 1 nodes as 'children' of the disk 2 'parent'. 

For *n = 3*, keep in mind that multiple recursion behaves in a depth-first fashion. So we can expect the first move to be represented by the left-most bottom leaf, and the final move to occupy the right-most bottom leaf.

.. figure:: img/hanoi03.jpg
   :scale: 50 %
   :alt: alternate text
   :align: center

   Figure 4. Call tree for *n = 3*

The *n = 3* tree clearly shows how the order of execution is developing. The further down the tree we go (ie, the smaller the disk, or the smaller the *n*), the more moves are required of it. The way each disk occupies its own 'level' of the tree is also consistent. This will become very handy when we finally look at designing our recursive calls.

Finally, the tree for *n = 4* really illustrates the 'rhythm' of expansion. We dip down to the leftmost leaf, and execute the triangle of calls of frames 3-5, then back to frame 2, then the trio of frames 6-8. Having completed the left side of the tree, we execute the root node, and then move on to the right side.

.. figure:: img/hanoi04.jpg
   :scale: 30 %
   :alt: alternate text
   :align: center

   Figure 5. Call tree for *n = 4*

We're almost ready to design our recursive calls. But before we do that, I want to point out one difference that you may have noticed between ``simpleHanoi()`` and ``mergeSort()``: the order of execution is altered. In ``mergeSort()``, for example, frame 2 (where ``L == [8, 4, 1, 6]``) splits its list into ``[8, 4]`` for frame 3, and ``[1, 6]`` for frame 6. The two called frames are executed first, followed by the calling frame. In ``simpleHanoi()``, we execute the first called frame, then the calling frame, and lastly the second called frame - another thing we'll certainly have to take this into account.

The other thing to consider is exactly what kind of data we are providing as our solution. I've been presenting the puzzle in a very text-heavy way (eg, 'Move disk 3 from A to B'). This is because most implementations present the print statements *as the solution*. Once the program is finished, all that you have to show for it is what's on the screen. Admittedly, this is a little bizarre. With the exception of our drawn fractals, we have so far worked with algorithms that provided us with results that we could then send to other functions. I'll continue to develop the Tower of Hanoi example as it's commonly done, but it's certainly possible to repurpose the code so that it isn't quite so self-contained. 

Designing the recursive calls
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that we've successfully built a model of how the puzzle works, we want to 
think about how a function could produce an output that matches the order of steps in our binary tree. How do we fit the recursive calls to the model? 

What do we have to cook with? Not much, it seems - *n*, *A*, *B* and *C*. But it's enough. Also, thanks to our construction of both the binary tree and the sequence of moves, we know ``simpleHanoi()`` will have:

1) Two recursive calls
2) A way to decrement *n* so that we can address all 'levels' of *n*
3) The order of execution must be 'called frame/calling frame/called frame', versus the 'called frame/called frame/calling frame' implementation we had in ``mergeSort()``
4) 'Move disk *n* from *A* to *C*' must occur at the midpoint of program execution

At this point, let's see if we can use these guidelines to simply just recreate the text output for *n = 2*:

.. code-block:: text

    Move disk 1 from A to B
    Move disk 2 from A to C
    Move disk 1 from B to C

To get this result, we could draft a totally fake, non-recursive function:

.. code:: python

    def simpleHanoi(n):
        print('Move disk', n - 1, 'from A to B')
        print('Move disk', n, 'from A to C')
        print('Move disk', n - 1, 'from B to C')

    n = 2

    print(simpleHanoi(n))

.. code-block:: text

    >>> Move disk 1 from A to B
    >>> Move disk 2 from A to C
    >>> Move disk 1 from B to C

This addresses most of the four points above: 

1. We have two recursive calls.
2. ``n - 1`` implies where our recursive calls should be placed.
3. By sandwiching the ``print`` statement between the two calls, we guarantee that we will follow the 'called frame/calling frame/called frame' order of execution. 
4. This sandwiching also ensures that the move of disk *n* from *A* to *C* occurs at the midpoint. And by recursion, we can assume that, at greater values of *n*, this symmetry will hold, since the first recursive call represents the left side of the tree, and the second the right. 

Obviously this code, in addition to not being recursive, doesn't really work for anything other than *n = 2*. In fact, if recursion is to do all the work, the *only* print statement we can have is the one in the middle. The recursive statements will have to arrange the function's variables - ``A``, ``B``, ``C`` - so that the correct print statement is executed for *n*, *n - 1*, …, until ``n == 0``. And all in the correct order! Obviously, this means we cannot hard-code the string ``from A to C``, because that will be correct for only a tiny number of moves. How can we create the flexibility we need?

While we were deriving the steps needed to move the disks in the correct sequence, you may have noticed that any given step always only uses two pegs. This implies that each recursive call should specify both the peg that holds the disk, and the peg that is the disk's destination. The third peg doesn't need to be specified as it's not at all part of the move. Staying with *n = 2*, it's simple to restate our fake code to include our peg variable names:

.. code:: python

    def simpleHanoi(n, A, C, B):
        if n > 0:
            simpleHanoi(n - 1, A, B, C)
            print('Move disk', n, 'from', A, 'to', C)
            simpleHanoi(n - 1, B, C, A)

    n = 2

    print(simpleHanoi(n, 'A', 'C', 'B'))

.. code-block:: text

    >>> Move disk 1 from A to B
    >>> Move disk 2 from A to C
    >>> Move disk 1 from B to C

I've made only four modifications to the code: 

1. I've *literally translated* the first and third print statements into recursive calls (keep in mind that we still need to pass the correct number arguments originally defined in the function, so even though they're not mentioned in the print statements, we still have to include ``C`` and ``A`` as arguments in their respective calls). 

2. In order to make this literal translation possible, the initial parameters in the function definition (and the arguments passed when ``simpleHanoi()`` is first invoked) are ``(n, A, C, B)``

3. Instead of hard-coded text, the middle print statement now calls parameters ``A`` and ``C``, whose values are, unsurprisingly, *A* and *C*.

4. The function's contents is now inside an ``if`` block. Otherwise the first recursive call will keep decrementing ``n`` into negative numbers until the maximum recursion depth is exceeded, and we won't ever see a single print statement.

Ok, so we've translated a fake function into something that provides the same output for the simplest possible case that can use recursion. However, here is the remarkable thing: run this version of ``simpleHanoi()`` for any value of *n* and you'll see that it yields the correct sequence. Somehow, we've solved the problem. How the hell did this happen?

Why does it work?
^^^^^^^^^^^^^^^^^

The short answer is thanks to recursion. That is, if it works for the simplest possible binary tree, it should work for a binary tree of *any* size. This isn't an exclusive property of binary trees, though. Recall with ``sierpinski()``, once we'd solved the base case and the minimally recursive case, we'd also solved the problem for any order fractal we wanted.

You may find this explanation unsatisfying, so let's look more closely at how ``simpleHanoi()`` generates all the right calls, in the right order.

Another perspective might be helpful. Up until this point, I haven't at all mentioned the base case. Is there one? Of course, but not where you might think it to be. Go back to the section on the Sierpinski triangle, and you'll see a similar construction:

.. code:: python

    def sierpinski(t, order, p):
        draw(t, p)
        if order > 0:
            # insert recursive magic here    

With ``sierpinski()`` we wanted to be sure that we drew the triangle at the given coordinates ``p`` even if the ``if`` block never triggered. This always gave us a result, even if ``order == 0``. In this sense, the base case for ``sierpinski()`` is the triangle described by ``p``. It's the simplest statement of the problem. 

Similarly, the simplest statement of the problem for ``simpleHanoi()`` is 'Move disk 1 from *A* to *C*', where *n = 1*. We enforce this by making the execution of the entire function subject to the condition ``n > 0``. In the case of *n = 1*, both recursive calls send ``n == 0`` to new frames. Since neither new frame triggers the ``if`` block, the program exits these frames without any further action. All we're left with is ``print('Move disk', n, 'from', A, 'to', C)``. From this, we can generalize that the base case of any recursive treatment of a binary tree is that tree's root node. 

For all other cases where ``n > 1``, we trigger recursive calls that have to do actual work. If ``n > 1`` and ``Move n from A to C`` is always the midpoint, then the left side of the tree finishes when all disks are stacked on peg *B*, and all calls on the right side of the tree are dedicated to getting the rest of the disks from *B* to the destination peg, *C*.

The technique behind these calls should look familiar to you. If it doesn't, go back to the example of the greatest common divisor in the section :ref:`05 Swapped`. But while ``gcdRecur()`` swapped two parameters in a recursively linear context, ``simpleHanoi()`` is more complex, swapping three parameters over two recursive calls. 

This swapping is *by far* the most difficult thing to understand about ``simpleHanoi()``, so let's take a closer look at how it works. Let's begin with frame 1 for *n = 2*. There's a lot to keep track of, but keep in mind one of our heuristics: a frame holds the values of all the parameters and variables *for that frame*. Those values don't change, either, unless we explicitly bind another value to one of those variables. What *does* get changed is the content/arrangement of the arguments for each recursive call. 

So far we have mostly been breaking down (or augmenting) lists, passing on boolean values, etc. Here all we're doing is taking the parameters of the calling function and swapping them to form the arguments for the called function.

That's the trick: once the recursive call is made from a given frame, the swapped values then become the order for the newly called frame. When we return to the calling frame, we *re-use* the values as they exist *in the calling frame* to populate the swapped arguments for the second call. In the abstract this makes sense, but in practice it gets tricky rather quickly, so I'll modify the code to label pegs *A*, *B* and *C* with source (``src``), temp (``tmp``) and destination (``dst``): 

.. code:: python

    def simpleHanoi(n, src, dst, tmp):
        if n > 0:
            simpleHanoi(n - 1, src, tmp, dst)
            print('Move disk', n, 'from', src, 'to', dst)
            simpleHanoi(n - 1, tmp, dst, src)

    n = 2

    print(simpleHanoi(n, 'A', 'C', 'B'))

It's easy to follow the arguments ``A``, ``C`` and ``B`` as they're passed to the function and assigned to parameters ``src``, ``dst`` and ``tmp``. These, along with ``n``, are used explicitly in the print statement. For the recursive statements, the swapping alters which term is bound to a given peg:

.. code-block:: text

    frame 1 function definition:
         1) src = 'A'
         2) dst = 'C'
         3) tmp = 'B'

       1st recursive call:
         1) src —> src or 'A' —> 'A'
         2) dst —> tmp or 'C' —> 'B'
         3) tmp —> dst or 'B' —> 'C'

             frame 2 function definition:
                 1) src = 'A'
                 2) dst = 'B'
                 3) tmp = 'C'

       2nd recursive call:
         1) src —> tmp or 'A' —> 'B'
         2) dst —> dst or 'C' —> 'C'
         3) tmp —> src or 'B' —> 'A'

             frame 3 function definition:
                 1) src = 'B'
                 2) dst = 'C'
                 3) tmp = 'A'

Here is an expanded version of our *n = 2* call diagram, where I've written in the functions' actual values into the function definitions and recursive calls:

.. figure:: img/hanoi05.jpg
   :scale: 30 %
   :alt: alternate text
   :align: center

   Figure 6. Call tree for *n = 2* with complete function definitions

You can now see clearly how frame 1's first recursive call…

.. code-block:: text

    simpleHanoi(1, src='A', tmp='B', dst='C')

…sets the parameters for frame 2's function definition:

.. code-block:: python

    def simpleHanoi(1, src='A', dst='B', tmp='C'):
        # frame 2 stuff

By the same token, frame 1's second recursive call…

.. code-block:: text

    simpleHanoi(1, tmp='B', dst='C', src='A')

…sets the parameters for frame 3's function definition:

.. code-block:: python

    def simpleHanoi(1, src='B', dst='C', tmp='A'):
        # frame 3 stuff

Since the frames where ``n == 0`` don't do any work, I don't include them in the diagram. You could say they are 'ghost frames' because they just open and close, returning program control to the calling frame. Still, these frames are vital, because it's at this point that the recursion 'turns around'. In a sense, these frames exert the pressure the program needs to bubble back up the call stack.

You can see that a diagram that lists the complete function with all variables, parameters and arguments will get quite big quite quickly. So if you still don't trust my explanation, I invite you to expand it out to cover *n = 3* or even *n = 4*. Eventually, I believe you'll agree with me.

In the next section [forthcoming], we'll look at another way of solving the Tower of Hanoi problem. We'll use the strict version, which won't allow us to jump over the middle peg. It will be a little more laborious, but will also lead to some very surprising results.

Heuristics and Exercises
^^^^^^^^^^^^^^^^^^^^^^^^

♦ Gathering as much information about a problem can have multiple benefits. As you develop your knowledge you can begin to see patterns and clues, which can inform the design of a recursive (or any other) solution.

♦ If a recursive solution works for a binary tree of the smallest non-trivial size, there's a good chance it will work for a binary tree of any size.

♦ Swapping arguments is a powerful method for covering all contingencies of a recursive scenario - as long as you can keep track of what is going on.

♦ You can control the order in which nodes are executed by changing the order of statements in the recursive function (ie, where the recursive calls are, in relation to the node's statements)

♦ We can write recursive functions in such a way that, at the leaf level of a binary tree, nothing happens (no statement is executed). However, this is very useful for when you just need to 'turn around' the recursive cascade.

♦ The base case of a binary tree is always the root node.

**Exercise:** You've shown ``simpleHanoi()`` to your friends but they still don't believe you, because everything is "just print statements" and there's "no real data". Following the usual rules, modify (or re-write) ``simpleHanoi()`` so that you begin with three lists…

.. code-block:: python

    A = [4, 3, 2, 1]
    B = []
    C = []

…and end up with…

.. code-block:: python

    A = []
    B = []
    C = [4, 3, 2, 1]

Your (recursive!) solution should provide for a way to store each step in some sort of data structure that will be returned by the function when it finishes executing.

Now try to modify your solution to use a dictionary that's initially defined as…

.. code-block:: python

    d = {A: [4, 3, 2, 1], B: [], C: []}

…and ends up in the following state:

.. code-block:: python

    d = {A: [], B: [], C: [4, 3, 2, 1]}

**Exercise:** Can you write a recursive solution to the Tower of Hanoi as a variation on our expansion of Pascal's triangle? How about as an L-system? For this exercise, only reproduce which disk is moving for each step. (Hint: refer back to the part where I discuss the expansion of the series for various values of *n*).