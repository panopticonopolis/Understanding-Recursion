.. _01 Introduction:

Introduction: Another Guide To Recursion? And Why Is It So Long?
================================================================

Oftentimes algorithms are presented to beginning programmers as recipes or to-do lists. You may have a to-do list that starts with 'pick up the kids from school', then realize that, you first must put gas in the car. But first of all, you need to find your car keys. You can't do the last step unless you've done all the other ones. If you're baking a cake, the final step may be to frost the cake, but not before you have taken the cake out of the oven, which of course requires you to make the cake in the first place, in addition to preparing the frosting, etc. 

Of course, there is nothing particularly recursive about this. Breaking a problem down into more manageable subproblems is true of algorithmic design in general. What makes recursion unique in algorithmic thinking is the fact that it's about designing a solution in such a way that, simply by running the same function over and over with slightly modified inputs, you can have your cake and eat it too.

The following guide to recursion is undoubtedly long. But that is because most material currently available online is simply too short. Recursion is an extremely compact method. While this allows for a presentation that is usually considered 'elegant', the practical consequence is that much of the program execution takes place implicitly. 

For example, it's not terribly difficult to understand the principle of recursive action, whether generally or for a specific piece of code. But, as with any algorithm, the ability to write out the actual process of computation on a line-by-line basis is where the rubber meets the road. As a result, much of the length of this document is due to the fact that I have annotated what actually happens when a function undergoes recursive computation, using print statements to trace inputs, transformations and outputs. 

It's also long because the usual ways of presenting problems and their recursive solutions is inadequate. Usually, the code is simply given *ex cathedra*, or perhaps accompanied with a brief commentary on its salient aspects. With recursion, there is an abiding mystery of *just how* we got to that solution. Although it's not true for every algorithm I've chosen to present here, in this guide I prefer to describe how to think about the problem, and then how to convert that critical thinking into code. 

If you understand recursion on an intuitive level, this guide is most likely not for you. It will be boring and obvious. But for many others, it's my hope that the explanations of how recursive calls and returns work will be time well spent.

The following text uses Python 3.6 but the concepts should be broadly applicable to many languages. A solid working knowledge of Python is preferred, including the basic data types and their associated methods. (Virtually) no familiarity with object-oriented techniques is needed.