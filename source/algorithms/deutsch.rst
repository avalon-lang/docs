.. highlight:: none

Deutsch's algorithm
===================

Deutsch's algorithm will be our first quantum algorithm to look into.
As will be the style throughout the documentation, we will try to keep the mathematics to 
minimum and focus on the implementation.  
This doesn't mean that we won't try to understand why and how the algorithm works but
we will seek to only use the mathematics necessary to get us to our goal.
If it is your wish to study the algorithm in depth, references are given at the end of the section.

Here is how this section is organized: first, we will we will set up the problem with all
the information necessary to understand the problem we are trying to solve.
Then we will state the problem and comment on it. Afterwards we shall present a classical
solution to the problem and comment on the solution. Following the classical solution
we will present a quantum solution. Then we shall close with some concluding remarks.

Introduction
------------

Imagine the following: you are given a function :math:`f:\mathbb{B} \to \mathbb{B}`
where :math:`\mathbb{B}=\{0, 1\}`. So this function takes zero or one and returns
zero or one.

What are the possible input/output combinations of this function?

* First combination: :math:`f_{0}(0) \to 1` and :math:`f_{0}(1) \to 1`.
* Second combination: :math:`f_{1}(0) \to 0` and :math:`f_{1}(1) \to 1`.
* Third combination: :math:`f_{2}(0) \to 1` and :math:`f_{2}(1) \to 0`.
* Fourth combination: :math:`f_{3}(0) \to 0` and :math:`f_{3}(1) \to 0`.

Notice that for the first and the fourth combinations, the output doesn't change
irrespective of the input. The output is constant; hence the function is said to be
constant in both combinations.  
The second and the third combinations though are different: the number of zeros in
the output is the same as the number of ones. There is a balance in the output; hence
we call it balanced in both combinations.

.. admonition:: Balanced and constant functions
    
    A function :math:`f:\mathbb{B} \to \mathbb{B}` is said to be constant if :math:`f(0)=f(1)`.
    It is said to be balanced if :math:`f(0) \neq f(1)`.


There are lots of interesting questions we can ask about this function (yes, actually a lot).
But here is one we are going to try to solve: suppose we are given one of the four combinations
randomly. Without being told which one or being allowed to look inside,
we are asked if it is balanced or constant. This is called Deutsch's problem after David Deutsch
who first proposed a quantum solution to this problem in 1985 (and mind you, his first solution worked
only half the time). When we are given a function which implementation we don't know,
we call it a black box or more formally an oracle.

.. admonition:: Deutsch's problem
    
    Let :math:`f:\mathbb{B} \to \mathbb{B}` be an oracle. Is :math:`f` balanced or constant?


One remark though, however obvious, is that we are going to need to call the oracle and pass it
our input(s). Note that nothing is said about reading the output. Read on and find out why.

.. note::
    Throughout the section, we are going to assume that we are given combinations as oracles
    of the form :math:`f_{i}(n)` for :math:`i \in \{0, 1, 2, 3\}` and :math:`n \in \mathbb{B}`. 

Classical solution
------------------

So we are given the oracle, now we need to find out if it is balanced or constant.
As programmers, part our business is finding out how to solve problems efficiently.
Many things may go into the final solution but right now we do know that we would like to reduce
the number of times we call the oracle. The more we call it the longer it will take to
make our decision of whether we have a balanced or a constant oracle.

We can visualize the classical algorithm running as shown below.

.. _classical_solution:
.. figure:: /_diagrams/deutsch/classical.png
    :scale: 40%
    :align: center
    :alt: Classical solution to Deutsch problem

    Classical algorithm executing the oracle :math:`f`.


So how many times exactly do we need to call the oracle here?

If we call the oracle passing it :math:`0`, the first combination will return :math:`1`.
But the third combination will also return :math:`1`. Also, the second combination
will return :math:`0` when given :math:`0` as input. We cannot hope to know if the oracle is balanced
or constant by passing it :math:`0` alone. The same argument applies when :math:`1` is given as input.

But note that if we call the oracle with :math:`0` then with :math:`1`, we can make some progress.
If :math:`f(0)` and :math:`f(1)` are equal then we know the oracle is constant by definition.
The same argument can be used by definition of a balanced function.

Therefore we need to call the oracle once for :math:`f(0)` and once for :math:`f(1)` to solve
the problem. Two calls to the oracle are *necessary* (we can't make a decision with only one call)
and *sufficient* (we learn nothing new by a third call).

Here is the classical code that solves Deutsch's problem.
You can find it in the ``deutsch`` folder in the algorithms repository at the `Classical and Quantum Algorithms in Avalon <https://github.com/avalon-lang/algorithms/tree/master/deutsch/>`_.

.. code::
    
    import io
    import oracles.classical


    def __main__ = (val args : [string]) -> void:
        -- step 1 : call the oracle with 0
        val left = Oracle.f0(0b0)

        -- step 2 : call the oracle again but with 1
        val right = Oracle.f0(0b1)

        -- step 3 : compare both results to determine if the oracle is constant or balanced
        if left == right:
            Io.println("The oracle contains a constant function.")
        else:
            Io.println("The oracle contains a balanced function.")

        -- we are done
        return


Notice that we are calling the oracle twice, first in step 1 then in step 2. Therefore,
any algorithm that allows us to solve the exact same problem in less than two calls 
(that is in one call) is better than the current classical algorithm. And coming right next up
is that solution, first due to David Deutsch.

Quantum solution: Deutsch's algorithm
-------------------------------------

Quantum algorithms are a bit harder to figure out and harder to reason about concerning
their correctness. But we will do that here at the expense of explaining the oracles.

If you read the code for classical oracles, they are not hard to understand. But it is
not immediately obvious how they got translated to quantum oracles. No matter, it is not
our objective to construct the oracles, you are not supposed to peek into them anyway.
So we are going to focus on the algorithm itself.

To get started, we need to transform the way the classical oracle is called into a flow
the quantum algorithm can work with. We can't use the flow in :numref:`classical_solution`
because it is not reversible. So we need to build an equivalent flow that has the same
effect but runnable on a quantum computer.

To make our oracles reversible, we use the following scheme: let :math:`f(x_1, x_2, \ldots, x_n):\mathbb{B}^n \to \mathbb{B}`
be a boolean function. We create a new function :math:`F(x_1, x_2, \ldots, x_n, z):\mathbb{B}^{n+1} \to \mathbb{B}`
mapped as shown below.

.. admonition:: XOR encoding of boolean functions

    F(x_1, x_2, \ldots, x_n, z) = (x_1, x_2, \ldots, x_n, z \oplus f(x_1, x_2, \ldots, x_n))


So we have transformed our classical function into a new function that takes booleans and returns a pair of booleans.


