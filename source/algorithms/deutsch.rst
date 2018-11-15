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

.. _classical_oracle:
.. figure:: /_diagrams/deutsch/classical.png
    :scale: 35%
    :align: center
    :alt: Classical oracle for Deutsch problem

    Classical oracle executing the function :math:`f`.


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

Quantum solution
----------------

Quantum algorithms are a bit harder to figure out and harder to reason about concerning
their correctness. But we will do that here at the expense of explaining the oracles.

If you read the code for classical oracles, they are not hard to understand. But it is
not immediately obvious how they got translated to quantum oracles. No matter, it is not
our objective to construct the oracles, you are not supposed to peek into them anyway.
So we are going to focus on the algorithm itself.

Classical oracle to quantum oracle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To get started, we need to transform the way the classical oracle is called into a flow
the quantum algorithm can work with. We can't use the flow in :numref:`classical_oracle`
because it is not reversible. So we need to build an equivalent flow that has the same
effect but runnable on a quantum computer.

To make our oracles reversible, we use the following scheme, dubbing it *XOR encoding of boolean functions*.

.. admonition:: XOR encoding of boolean functions
    
    | Let :math:`f(x_1, x_2, \ldots, x_n):\mathbb{B}^n \to \mathbb{B}` be a boolean function.  
    | Define :math:`U_f(x_1, x_2, \ldots, x_n, y):\mathbb{B}^{n+1} \to \mathbb{B}` as :math:`U_f(x_1, x_2, \ldots, x_n, y) = (x_1, x_2, \ldots, x_n, y \oplus f(x_1, x_2, \ldots, x_n))`.
    | The function :math:`U_f:\mathbb{B}^{n+1} \to \mathbb{B}` is the XOR encoding of :math:`f:\mathbb{B}^n \to \mathbb{B}` and is equivalent to it up to the ancilla :math:`y`.


So we have transformed our classical function into a new function that is equivalent to it but with two important properties:

* The function :math:`U_f:\mathbb{B}^{n+1} \to \mathbb{B}` is reversible.
* The original function :math:`f(x_1, x_2, \ldots, x_n):\mathbb{B}^n \to \mathbb{B}` ouput can be recovered by taking :math:`y \oplus f(x_1, x_2, \ldots, x_n) \oplus y`.

The two properties above of the function :math:`U_f:\mathbb{B}^{n+1} \to \mathbb{B}` mean
that it is executable on a quantum computer and from the answer it provides we are able
to recover the original answer the classical function would have given.

As mentioned above, we won't see how to build oracles from :math:`U_f:\mathbb{B}^{n+1} \to \mathbb{B}`
but it is a good exercise if you want to try it. Neither are we going to actually find the output.
We are going to do the following though:

* See how to use the oracles from :math:`U_f:\mathbb{B}^{n+1} \to \mathbb{B}` in the algorithm.
* Understand how the algorithm solves Deutsch's problem.

To begin, we are going to simplify the XOR encoding and limit :math:`f(x_1, x_2, \ldots, x_n):\mathbb{B}^n \to \mathbb{B}` to :math:`f(x):\mathbb{B}^n \to \mathbb{B}`.
This means that its encoding is given by :math:`U_f(x, y):\mathbb{B}^{2} \to \mathbb{B}`.

Then we are going to shift to the bracket notation in order to simplify calculations and make :math:`U_f(x, y):\mathbb{B}^{2} \to \mathbb{B}` accept inputs of the form :math:`|x, y\rangle`.
For our satisfaction, let us show that :math:`U_f(|x, y\rangle):\mathbb{B}^{2} \to \mathbb{B}` is both reversible and :math:`f(x)` can be recovered from it.

Let us first look at a circuit similar to the one in :numref:`classical_oracle`.

.. _quantum_oracle:
.. figure:: /_diagrams/deutsch/quantum.png
    :scale: 35%
    :align: center
    :alt: Quantum oracle for Deutsch problem

    Quantum oracle executing the function :math:`f` using its encoding :math:`U_f(|x, y\rangle)=|x,y \oplus f(x)\rangle`.


The oracle is given two bits in the form :math:`|x, y\rangle` and produces output of the form :math:`|x, y \oplus f(x)\rangle`.
Looking at :numref:`quantum_oracle`, we can see how the quantum oracle is truly quantum and at the same time can be used to get back the classical oracle.

* To get back the original oracle from the output, we ignore :math:`|x\rangle` and XOR :math:`|y \oplus f(x)\rangle` with :math:`|y\rangle` resulting in :math:`|f(x)\rangle` which is the result of the classical oracle.
* To prove that the quantum oracle is truly quantum and therefore must be reversible we only need to show that executing the oracle passing it its own output gives back the original input.
  To show that, let :math:`z = y \oplus f(x)`. Thus the new input is :math:`|x, z\rangle`.
  Giving that input to the oracle, the expected output is :math:`|x, z \oplus f(x)\rangle`.
  This output is equivalent to :math:`|x, (y \oplus f(x)) \oplus f(x)\rangle`. Rearranging, we get :math:`|x, y \oplus (f(x) \oplus f(x))\rangle`.
  And finally eliminating :math:`f(x)` due to XOR, we get as final output :math:`|x, y\rangle`.
  And with that we have the original input! Therefore the quantum oracle is reversible.

Deutsch's algorithm
~~~~~~~~~~~~~~~~~~~

We are ready to tackle the quantum algorithm. We won't discuss how to derive it and
will limit ourselves to understanding how it works. Why it works is another matter
that is not presented and you are encouraged to read references given at the end of
this section.

We present a circuit description of the algorithm from which we shall derive the final
program.

.. _deutsch_algorithm:
.. figure:: /_diagrams/deutsch/algorithm.png
    :scale: 35%
    :align: center
    :alt: Deutsch algorithm

    Deutsch's algorithm as the quantum solution to Deutsch's problem.


Using the :numref:`deutsch_algorithm` as reference, we are going to analyze what it does
and how we find out if the oracle is balanced or constant from its output.



