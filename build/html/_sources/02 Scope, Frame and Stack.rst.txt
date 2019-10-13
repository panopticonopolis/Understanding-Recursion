.. _02 Scope, Frame and Stack:

Scope, Frame and Stack
======================

The most common way of introducing recursion to programmers involves two seemingly simple steps. Given a programming language that allows a function to call itself, we construct a recursive function once we do two things:

1) Identify the base case
2) Identify the recursive case

We formulate these two cases by successfully reducing the problem at hand to 'its simplest possible solution', whatever that means. 

This, as the saying goes, is necessary but not sufficient, and there's much that still needs to be unpacked. The purpose of this guide is to provide the reader with a toolbox of heuristics that can be used to quickly analyze a recursive algorithm. Once the reader is comfortable with analysis, the same set of heuristics can be applied to thinking about a problem recursively, and constructing a solution. 

Understanding how functions work
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Recursion is commonly introduced to students within the context of iterative procedures, especially ``for`` loops, 'but with functions'. This is the first source of misunderstanding. Recursion is better considered as a consequence of functional scope, frames and the call stack. Put differently, we understand recursion when we understand how functions prioritize, quarantine, and share values bound to variables and the evaluation of expressions (scope), and the way those spaces are created and then discarded (frames added to and subtracted from the stack).

This may make little sense in the abstract, so let's look at some code. Just about the simplest program we can write is:

.. code:: python

    hello = "Hello, World!"
    print(hello)

.. code-block:: text

    >>> Hello, World!

In this program, there is only one frame in the stack, known as the global frame. We could add a few more variables and statements and what not, and everything would still belong to this global frame. All variables and their associated values would be just as available to any further statement we chose to write (although variables used for iterating over loops don't have this sort of permanence).

This changes when we introduce functions. Every time we call (as opposed to merely define) a function, a new frame is created, and is added to the stack. At the same time, the flow of control passes to that frame. The usual analogy is a stack of cafeteria trays: the global frame is the first tray. When we call a function from the global frame, the new frame is placed on top of it. When that function completes its computation, it returns its results, and the frame is discarded. In terms of our analogy, we can put what we want on the tray, then remove it from the top of the stack. The tray beneath it is now available for plating food, or whatever it is we want to do with it. This kind of ordering is commonly known as LIFO, or Last In First Out. 

Of course, if the function calls another function and is waiting for results, then its frame remains open or 'on the stack'. But as mentioned, while there may be numerous frames open at any given point during a program's runtime, the Python interpreter is always executing the current computation within the context of a specific frame. This will be extremely important to keep in mind as we begin to understand recursion.

Part of what makes frames valuable is that they regulate the accessibility of a function's variables, otherwise known as scope. For example, consider this trivial function:

.. code:: python

    def foo(x):
        return x

    x = 12

    print(foo(x))

.. code-block:: text

    >>> 12

Easy enough. But consider that we also get ``12`` if we write:

.. code:: python

    def foo(y):
        return y

    x = 12
    
    print(foo(x))

Or even if we write:

.. code:: python

    def foo(y):
        return x

    x = 12

    print(foo(x))

This is scope at work. When we first specify ``x = 12`` we do in at the global frame of the program. When we pass ``x`` to ``foo()``, the function's parameter doesn't care if the argument in ``print(foo(_))`` is ``x`` or ``q`` or whatever, but only that it is of the correct type. Otherwise the function can't perform its computation. But once the argument is passed to the function, we *must* refer to it within the function as it was named in the function definition, ie, ``foo(y)``. 

Once ``foo()`` is invoked, it has its own frame created on the stack, one that now has its own, 'local' scope. (Note that the function has to be called or invoked - simply defining a function doesn't create a new frame). Within that frame, ``y`` is a local variable that exists *only* within the function. When the function's work is finished, ``y`` and all other local variables are destroyed - you can verify this in the second snippet of code, by attempting to ``print(y)`` outside of the function. Even if you invoke ``print(y)`` after the function has returned ``y``, you'll get an error. So why does ``return x`` in the last code snippet not throw an error?

When we declared ``x = 12`` we did so in the global frame. As such, it is a global variable. This means that ``x`` will be available to all statements and expressions in that frame (such as ``print(foo(x))``), but also to any frame that originates from the global frame, like the one generated when we called ``foo()``. In other words, if Python is executing ``foo()`` and comes across a variable that is not stated inside ``foo()``, it will look outside the scope and, if it finds ``x``, it will use it (of course, if it doesn't find it, it will throw an error). 

Take to the extreme, we could use this to forego passing any argument at all, although that doesn't seem very useful:

.. code:: python

    def foo():
        return x

    x = 12

    print(foo())

.. code-block:: text

    >>> 12

It stands to reason, then, that global and local variables retain their separate identities, even if they share a name:

.. code:: python

    def foo(f):
        f = 5
        print('in foo(), f =', f)
        
    f = 12
    print(foo(f))
    print('in global frame, f =', f)

.. code-block:: text

    >>> in foo(), f = 5
    >>> in global frame, f = 12

Here, the ``f`` printed inside the function is the local variable. Python won't look any further than it has to, and is happy to print ``5`` for ``print('in foo(), f =', f)``. At the same time, ``print('in global frame, f =', f)`` gives us ``12`` in the global frame, even though we are also using ``f`` as the argument when we call ``foo()``. Having two variables with the same name doesn't bother the interpreter, as they are separated from one another by the rules of scope.

.. note:: Many of the programs we'll see in this guide will be single functions, so *function* and *frame* may seem to be used interchangeably, in the sense that each frame will contain the function that is the entirety of the program. In a program with multiple functions, a separate frame opens for each function, and doesn't take anything else with it. So there is an important difference, but one we won't encounter very often.


Nested functions are nested frames
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You may have already seen functions defined - or 'nested' - within other functions. As you may suspect, when such functions are invoked, the frames that contain them are also nested. Here, ``foo2()`` is a function defined within the function ``foo()``. As a result, ``foo2()`` can also be called a 'helper function':

.. code:: python

    def foo():
        f = "I'm enclosed"
        print('in foo(), f says', f)

        def foo2():
            f = "I'm local"
            print('in foo2(), f says,', f)

        print(foo2())


    f = "I'm global"

    print('in global frame, f says,', f)
    print(foo())

.. code-block:: text

    >>> in global frame, f says, I'm global
    >>> in foo(), f says I'm enclosed
    >>> in foo2(), f says, I'm local

Here, ``f`` in ``foo()`` is an 'enclosed' variable, since ``foo()`` is a wrapper for another function. The separation of local vs enclosed vs global allows each ``f`` to be printed correctly. 

Just as we can reference a global variable from inside a local frame, we can also reference an enclosed variable from inside a local frame, as long as the function in the local frame is wrapped into the function in the enclosed frame:

.. code:: python

    def foo():
        g = "I'm enclosed"
        print('in foo(), g says', g)

        def foo2():
            h = "I'm local"
            print('in foo2(), g says,', g)
            print('from foo2(), f still says,', f)

        print(foo2())

    f = "I'm global"

    print('in global frame, f says,', f)
    print(foo())

.. code-block:: text

    >>> in global frame, f says, I'm global
    >>> in foo(), g says I'm enclosed
    >>> in foo2(), g says, I'm enclosed
    >>> from foo2(), f still says, I'm global

The last print statement also shows that we can access the global frame, no matter how deeply nested we are. This is true for any frame, as long as there is a direct line of succession through which we can travel. 

It's important to note in the above example how ``foo2()`` is called. The call doesn't happen in the global frame, but from inside ``foo()``'s frame. On the other hand, try this code and see what happens:

.. code:: python

    def foo():
        g = "I'm enclosed"
        print('in foo(), g says', g)

        def foo2():
            h = "I'm local"
            print('in foo2(), g says,', g)
            print('from foo2(), f still says,', f)

    f = "I'm global"

    print('in global frame, f says,', f)
    print(foo())
    print(foo2())

In order for ``foo2()`` to be accessible, we have to call it from the frame that contains its definition. This makes sense, because what's the difference between asking for ``foo2()`` from the global frame, and asking for the variable ``g`` in ``foo()`` from the same, global frame? None; both ``g`` and ``foo2()`` are locked away inside the scope that can only be accessed by invoking ``foo()``. 

By the same logic, whatever ``foo2()`` returns will be returned *to the frame that called it*. This seems obvious but is actually another key point to keep in mind. I'm belaboring these points because frames and scopes are implied and as such can be somewhat invisible. If you're careless with your variable names or call order you may experience unintended consequences. But it's also impossible to understand recursion without understanding - and keeping track of - frame and scope. 

It's also pretty much impossible to design recursive code if you can't visualize the flow of values as they are passed from one frame to another. Let's simulate what that looks like with an an extremely artificial example, a series of nested functions:

.. code:: python

    def foo1(x=3):
        
        def foo2(y=2):
        
            def foo3(z=1):
                return z
            
            return foo3() + y
        
        return foo2() + x

    print(foo1())

.. code-block:: text

    >>> 6

There are four successively nested frames here: 

1) The global frame, containing ``print(foo1())`` and the definition of ``foo1()``

2) The first frame, containing ``foo1()`` and all of its variables and methods, including the definition of ``foo2()``

3) The second frame, containing ``foo2()`` and all of its variables and methods, including the definition of ``foo3()``

4) The third frame, containing ``foo3()`` and all of its variables and methods

Executing each function call in the above order invokes the next function (and therefore the next frame). You could say it took us four steps to get to the innermost frame, where ``foo3()`` reigns supreme. What happens then? 

5) ``foo3()`` returns the value ``z == 1`` to ``foo2()``. The third frame is now closed.

6) In the second frame, ``1`` is added to ``y``, which is ``2``. The sum ``3`` is then returned to ``foo1()``, and the second frame is closed.

7) In the first frame, ``3`` is added to ``x``, which is ``3``. The sum ``6`` is then returned to the global frame, and the first frame is closed.

8) Finally, the global frame receives the results from ``foo1()`` and prints ``6``.

Of course we would never write code like this for such a simple computation, but take a moment to make sure you understand the flow of how functions were called and values were passed back, frame by frame. This is not dissimilar from what happens in the course of recursion. The difference is that recursion won't use a cascade of nested functions, but one function calling itself repeatedly.

Functional state and namespaces
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As we've discussed, frames fulfill a crucial role in setting boundaries around computation by creating privacy by the rules of scope. This implies that each function has its own *state*, which includes the values assigned to variables, the items that happen to be in a list, etc. Of course, at the moment a function is invoked, it will have certain arguments passed to it, and those values may impact the function's state. In turn, the state of the calling function, which receives the results returned from the called function, may (or may not) change thanks to those results. Recursion uses these same principles. 

Let's look at some code that demonstrates this. 

.. code:: python

    def foo(x):
        print('state of x before calling foo2() is', x)

        def foo2(y):
            y = 7
            print('state of "x" passed to foo2() is', y)
            return y

        x = foo2(x)
        print('state of x after calling foo2() is', x)

        return x

    print(foo(12))

.. code-block:: text

    >>> state of x in foo() before calling foo2() is 12
    >>> state of "x" passed to foo2() is 7
    >>> state of x in foo() after calling foo2() is 7
    >>> 7

Here we changed the state of ``x`` in ``foo()`` by assigning ``x = foo2(x)``. But we have to be explicit about it - if we had just written ``foo2(x)`` without the ``x =`` then the state of ``x`` in ``foo()`` would have remained ``12``. In other words, the *namespace* of ``x`` in ``foo()`` would not have changed.

Generally speaking, the originating frame - in this case ``foo()`` *persists in its own state* until the results of the called function are returned to the *exact place* where the call originally occurred. It's only at that point that the originating frame carries on with its own computation, which may (or may not) involve a change in its state. This may be difficult to visualize at the moment, but it will become clearer as we begin looking at actual recursive code.

This is also why the comparisons between procedural/iterative and recursive solutions are at best limited, and at worst thoroughly confusing. Of course, in many cases it's possible to write both a recursive and an iterative solution to the same problem, and we'll even see solutions that combine both techniques. But the dependence of iteration on frame/scope, at least within a function's internal workings, is fairly minimal, whereas frame/scope is essential to understanding the mechanism of recursion.

Heuristics
^^^^^^^^^^

Here are some important keywords that will help us navigate recursion:

♦ Frame: Whenever a function is called, a new frame is opened in which the function performs its computation. Once the computation is finished, the results are returned to the calling function and the frame, along with the function and its contents, is discarded.

♦ Stack: Also known as the call stack, it conceptualizes the order in which frames are created and discarded. This occurs in a last-in-first-out order (LIFO). Any frame that is open (ie, has not concluded its computation) remains on the stack, even though the program may not be executing anything in that frame. 

♦ State: The inventory of variables, methods and associated values constituting a function at a given moment in the program's execution. A function can change its state thanks to internal computation, or when a result is returned from a function called by it.

♦ Namespace: The value bound to a variable within a function, at a specific frame. Usually written as 'the namespace of ``x`` in ``foo()`` at frame 2 is ``3``.'