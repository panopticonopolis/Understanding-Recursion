.. _03 Frames:

Recursion in Light of Frames
============================

My first recursive code
^^^^^^^^^^^^^^^^^^^^^^^

Now we are slightly better able to answer the question of what happens when a function calls itself. Given the preceding discussion, we can characterize recursion as a specialized case of frame creation and destruction - one where each successive frame and its associated state is created by a function as a slightly modified copy of itself.

Consider one of the simplest examples of recursion, a function ``summ()`` that sums a series of numbers. Starting from a given positive integer ``n``, this function should add to ``n`` every integer smaller than ``n``, all the way to ``1``. We would write this out as:

``summ(3) == 3 + 2 + 1 == 6``

Here is the recursive code, which we'll pick apart below. Don't panic.

.. code:: python

    def summ(n):
        if n == 1:
            return n
        else:
            return n + summ(n - 1)

    print(summ(3))

.. code-block:: text

    >>> 6

When our program (call it ``summ.py``) is first executed, we create the global frame. At the global frame, Python's interpreter defines ``summ()`` but does not  execute it. It basically just says, 'Here is a function called ``summ()`` that is expecting one argument. Gotcha.'

The next method executed is ``print()``, which calls ``summ()`` with an argument of ``3``. This creates frame 1 in the stack, which, if we could see it, would contain a very familiar-looking function:

.. code:: python

    def summ(3)
        if 3 == 1:
            return 3
        else:
            return 3 + summ(3 - 1)

Leveraging our discussion of namespace at the end of :ref:`02 Scope, Frame and Stack`, we would say that "the namespace of ``n`` in frame 1 is ``3``." Let's now execute the function ``summ(3)``.

Since ``3 != 1`` we skip the ``if`` clause and proceed to ``else``. There's only a ``return`` statement there, so let's try to evaluate it. We can get there partway, since we know ``n == 3``, but can't complete the return because the second part invokes ``summ()`` again, except now with the argument ``n - 1``. So we have to call ``summ(2)``. This opens frame 2. In the meantime, frame 1 remains open, still waiting to resolve ``3 + summ(2)``. Here's what frame 2 looks like:

.. code:: python

    def summ(2)
        if 2 == 1:
            return 2
        else:
            return 2 + summ(2 - 1)

In frame 2, the namespace of ``n`` is ``2`` - but keep in mind that in frame 1, it is still ``3``!. Regardless, frame 2 is in the same boat as frame 1: ``2 != 1`` so we skip the ``if`` clause and go to ``else``. At the ``return`` statement, we know that ``n == 2`` but still need to solve for ``summ(n - 1)``, ie, ``summ(2 - 1)`` or ``summ(1)``. So we invoke ``summ()`` once again, this time with an argument of ``1``. We leave frame 2 open, waiting to resolve ``2 + summ(1)``, and create frame 3 by calling ``summ(1)``.

.. code:: python

    def summ(1)
        if 1 == 1:
            return 1
        else:
            return 1 + summ(1 - 1)

Here in frame 3 we finally satisfy the ``if`` condition, since the namespace of ``n`` in this frame is indeed ``1``. We can return ``1`` and the ``else`` block goes untouched. Once we complete the return, the function in frame 3 will terminate and the frame will close. 

Now we begin moving back through the succession of open frames to conclude the computation. From frame 3, we are returning ``1`` to the frame that called it (frame 2). But where is it winding up in frame 2?

.. code-block:: python
   :emphasize-lines: 5

    def summ(2)
        if 2 == 1:
            return 2
        else:
            return 2 + summ(1)

Quite simply, it replaces ``summ(1)`` as the last term in the last line. This may seem obvious, but in a recursive setting, knowing where the returned value lands from the called function is essential to tracing the path a recursed value takes. 

Now that we have a definite value for ``summ(1)``, we add it to ``2``. With our ``3`` in hand, we are ready to return that value to frame 1, which is what's next in the call stack. Once we've returned ``3``, frame 2 is discarded.

.. code-block:: python
   :emphasize-lines: 5

    def summ(3)
        if 3 == 1:
            return 3
        else:
            return 3 + summ(2)

The same process applies: now that we know that ``summ(2) == 3``, we can add ``3 + 3`` and return ``6`` to the global frame. Once we've done this, the program ends.

You can see that there is a distinct resemblance between the program flow in this example and our trivial code from the last example in :ref:`02 Scope, Frame and Stack`.

.. code:: python

    def foo1(x=3):
        
        def foo2(y=2):
        
            def foo3(z=1):
                return z
            
            return foo3() + y
        
        return foo2() + x

    print(foo1())

One of the elegant aspects of recursive code is that we don't need to create any additional functions beyond the one that we already specified to solve not only the specific problem, but the general case as well. In this case, we can now sum *any* positive integer ``n``. By invoking itself with a modified argument, the function handles as many steps are necessary - it just keeps decrementing until it gets to ``1``.

I want to point out two things before we leave ``summ()``. 

The first is about frames, state and namespaces. You'll note that, for every frame, the namespace of ``n`` was decremented by 1. Thus, even though ``summ()`` was pretty much a carbon copy in every frame, there was the crucial distinction of differing namespaces for ``n`` in each frame. Moreover, once each frame was seeded with those namespaces, the state persisted once we satisfied the ``if`` statement and we began returning values. 

In this case, we never changed the namespace for ``n`` - all the work was being done in the ``return`` statement, which picked up the existing value of ``n`` for that frame and just added it to what had been returned from the called frame/function. But fear not, we will see other examples where namespaces change for variables in frames.

The other thing worth mentioning is something you should always keep in mind when looking at recursive code: How much code gets executed on the way in to satisfying the ``if`` statement, and how much gets executed on the way back?

In this case, we carried out all of ``summ()``'s code in every frame *except* for the recursive call. Once we were able to ``return 1``, we could complete the computation for each open frame with ease. As an analogy, imagine you have to navigate a garden to get to, say a fruit tree. To do so, you make a path by laying down stones to a goal. Once you got to the tree and got the fruit, all you had to do to get back to your starting point was to retrace your steps along the path you had already laid. 

I'm being explicit about this because oftentimes the only examples of recursion offered are ones where the work is lopsided in exactly this way. People then get misled into thinking that there is some sort of special relationship between ``return`` and the recursive call, when there really isn't. In fact, we'll start pulling apart this syntax in the next sections, so that we can get the function itself to show us what's going on.

Understanding recursion as a process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now we can go back to the initial statement of what makes a function properly recursive:

1) Identify the base case
2) Identify the recursive case

Until now I've been illustrating the recursive case, which is another way of saying, in what way do we modify the argument the function passes to itself in order to break the problem down to its fundamental? But in order for recursion to work, we also have to recognize what is that fundamental state. This is the 'base case'. 

In the example of ``summ()``, the simplest possible number that is the sum of whole numbers preceding it is, well, 1. In mathematical notation, f(1) == 1. (We could also specify the base case as ``if n == 0`` but that would not add to the sum, and hence only represents an additional, unnecessary computational step.) So for ``summ()`` the base case is ``n == 1``. In this case we can also either return ``1`` or ``n``, as the value at the base case is the same as the variable that eventually led us there (this won't always be true, either).

The larger point is that the base and recursive cases must work in tandem: the recursive cases provides an incremental way to get to the base case, and the base case provides the fundamental answer that can then be returned back through the preceding frames to generate the final answer. At every step in the return journey, the returned value is computed against the *state of each frame*, until we are back where we began, solution in hand.

This process of 'seeding' our frames on the way to the base case is essential. Simply identifying the base case is necessary but insufficient, because if we don't arrive at the base case recursively, there is nothing against which the base case can be successively computed during the return journey. In ``summ()``, this took the form of adding ``n``'s namespace to the returned value of every recursive call. It is this unbroken chain of as-yet undetermined computation that is fulfilled when the base case begins its trip back to the original, global frame. In terms of frame we can narrate the program as saying:

    **1**   Global frame asks, "What is ``print(summ(3))``?""

    **2**   Frame 1 says, "``summ(3)`` is ``3 + summ(2)`` but I don't know what ``summ(2)`` is."

    **3**   Frame 2 says, "``summ(2)`` is ``2 + summ(1)`` but I don't know what ``summ(1)`` is."

    **4**   Frame 3 says, "I know that ``summ(1) == 1`` so I can now return ``1``"

    **3**   Frame 2 says, "``summ(2)`` is now ``2 + 1`` so I can now return ``3``"

    **2**   Frame 1 says, "``summ(3)`` is now ``3 + 3`` so I can now return ``6``"

    **1**   Global frame says, "``print(summ(3)) == 6``"

If the numbering looks unusual it's because I'm counting by frames and not steps. Steps may be useful for iterative procedures - for example, how many times are you looping through a particular iterable? Counting by frames emphasizes recursion's unique mechanism of first setting up the series of undetermined computations, and then, once the base case has been reached, completing those computations. As the numbering implies, this run of completions *always* happens in the reverse order.

Once you understand that each frame has its own state and that that state persists while it waits for the return to occur, there is no mystery in how each recursive step 'knows' exactly what to add to the returned result before passing it on. Sometimes recursion can be complex, but if you hold on to this axiom, you should be able to trace the flow of values, no matter how convoluted.

One way of visualizing this is to think of computation within a function as happening 'vertically', in the sense that a function generally executes its statements and expressions from the first line to the last line (taking into account, of course, branching and loops and such). On the other hand, recursive calls and their returns from frames thus generated are 'horizontal', in the sense that they stop and restart the generating function at the exact point where the function calls itself. 

I imagine it as a left-to-right process, but however you imagine it, the thing to keep in mind is that, at the moment the recursive call happens, all computation in the originating frame pauses. Similarly, at the moment that the 'horizontal' insertion occurs (ie, a result is returned to the originating frame), the 'vertical' computation restarts, with the state of the function in the originating frame re-engaged.

Another benefit of conceptually separating the 'horizontality' of what is being returned from the 'verticality' of a specific frame's computation is that it reminds us that *functions compute until they are done*, which overwhelmingly means when the function reaches a ``return`` statement. Of course, this doesn't mean that every line is executed. In ``summ()``'s base case, the ``else`` clause is never triggered. But for every frame the function executes until it reaches a return statement it can completely evaluate - or it reaches another recursive call.

In the interest of thinking about this more clearly, allow me to suggest some non-standard terminology. Let's call the entire recursive process a 'recursive cascade'. I suggest this because once a function calls itself, you're pretty much stuck with letting the entire process work itself out. Recursion is unlike iterative code where it's possible to break out of a loop once a condition is met. In recursion, you either get to the base case (and out again), or you die trying. Ok, not really, but you'll be stuck in an infinite loop and your program will request more memory than your machine has, and the program will crash. We'll look at a few exceptions to this, but in general recursion is intended to be exhaustive in its application.

The other two terms are related to the notion of 'horizontality' and 'verticality'. A great way to understand recursive code (and how to build it) is to recognize what needs to happen on the way to the base case, and what needs to happen on the way back. In this sense, I think of all the code that precedes the recursive call as being 'pre-recursive', and everything that happens after it as 'post-recursive'. 

Pre-recursively, we are generally 'seeding' each frame with the values we want to be available to the function. In the case of ``summ()``, that was making sure that each namespace of ``n`` had a value that had been decremented by 1. Once the base case was achieved, we flipped into post-recursive mode, where we revisited each calling frame (in LIFO order), and computed the base case's returned value with the value of ``n`` in each frame. 

I realize that you may squint at ``summ()`` and struggle to see why I should be so overdetermined in my nomenclature. Suffice to say that it will come in very handy as our examples become more complex.

Of course, you may not like these terms at all. For instance, you could argue that there is no such thing as 'pre-recursive' and 'post-recursive' becase once a function calls itself, it's all recursive. You could think about recursion using the garden path metaphor I tossed out above, or using something else entirely. I'm just trying to present a number of ways in which you can think about this technique, so please use whatever feels best to you, and let me know if you have any suggestions.

Heuristics and Exercises
^^^^^^^^^^^^^^^^^^^^^^^^

Since I like my way of framing recursion, I'll use it to begin defining some heuristics.

♦ The recursive cascade is the recursive mechanism viewed as a whole. Just as functions execute until they are done (ie, reach a ``return`` statement), a recursive function, once called, will 'cascade' to the base case and back to the frame that originally called it.

♦ Every recursive algorithm is divided into (at least) two parts - the pre-recursive and post-recursive. The pre-recursive part of the function is executed before the recursive call, and post-recursive is the part that is executed after the recursive call has returned its results.

♦ The pre-recursive portion of function seeds the frames with the desired namespaces until the base case has been reached. 

♦ Once the base case has been attained, the post-recursive portion of the function computes the value returned by the base case against the namespaces of selected variables, as pre-recursively seeded in each frame.

♦ The pre-recursive portion of the algorithm happens in the order that frames are created, whereas the post-recursive always happens in the reverse order.

**Question**: Am I just hedging by saying 'Every recursive algorithm is divided into (at least) two parts'? In what circumstances might there be more parts?

**Exercise:** Write a recursive solution ``factorial()`` to find the factorial of a given positive integer ``n``.

To solve this problem, first consider how factorials are computed. How can this formula then be applied to a recursive context? How can you use what we developed in ``summ()`` to characterize both the base and recursive cases?

Since we're interested in not just solving the problem but understanding how this solutions works, build into your program a functionality to keep track of what is going on. Can you keep track of each argument as it is passed recursively? Can you count the steps? Can you count the frames? How can you tell which is which?
