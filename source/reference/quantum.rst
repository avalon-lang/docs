.. highlight:: none

Quantum gates
=============

Avalon currently only supports 1-Qubit gates. Multi-qubits gates can be created from those.
Control gates are parametrized by 1-Qubit gates.

1-Qubit gates
-------------

Any 1-Qubit unitary gate is of the form:

.. math::
    
    Gate\left(\theta,\phi,\lambda\right)=\left(\begin{array}{cc}
    e^{-i(\phi+\lambda)}\cos\left(\frac{\theta}{2}\right) & -e^{-i(\phi-\lambda)}\sin\left(\frac{\theta}{2}\right)\\
    e^{i(\phi-\lambda)}\sin\left(\frac{\theta}{2}\right) & e^{i(\phi+\lambda)}\cos\left(\frac{\theta}{2}\right)
    \end{array}\right)

This is the form that Avalon implements directly as :math:`Gate\left(\theta,\phi,\lambda\right)`.

Indeed, ``Gate(float, float, float)`` is a value constructor that constructs values of type instance ``gate``
which can be applied to qubits.

As an example, let us show how one creates the Hadamard gate:

.. code::
    
    -- import math since it contains pi
    import math

    -- and here we have the Hadamard gate
    val had_gate:gate = Gate(Math.PI / 2.0, 0.0, Math.PI)


And all other 1-Qubits gates are created the same way.


Controlled gates
~~~~~~~~~~~~~~~~

Controlled gates are parametrized by 1-Qubit gates. All controlled gates are of the form:

.. math::
    
    CGate\left(gate\right)=\left(\begin{array}{cccc}
    1 & 0 & 0 & 0\\
    0 & 1 & 0 & 0\\
    0 & 0 & Gate_{00} & Gate_{01}\\
    0 & 0 & Gate_{10} & Gate_{11}
    \end{array}\right)


Each :math:`Gate_{ij}` is an element from the ``Gate`` constructor. The form above is the value constructor
``CGate(gate)`` and constructs values of type instance ``cgate``.

For instance, to construct a controlled Hadamard gate, only simple does the following:

.. code::
    
    -- import math since it contains pi
    import math

    -- and here we have the Hadamard gate
    val had_gate:gate = Gate(Math.PI / 2.0, 0.0, Math.PI)

    -- we create a controlled hadamard gate
    val had_cgate:cgate = CGate(had_gate)


Appling gate to single qubits
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Applying gates to qubits is extremely simple. One simply calls the ``apply`` function, passing it the gate and the qubit(s) to apply the gate to.
For 1-Qubit gates, ``apply`` has the signature :code:`apply(g : gate, q : ref qubit) -> void` and for controlled gates it has the signature
:code:`apply(cg : cgate, control : ref qubit, target : ref qubit) -> void`.

Let us demonstrate with an example, reusing our previous code:

.. code::
    
    -- import math since it contains pi
    import math

    -- and here we have the Hadamard gate
    val had_gate = Gate(Math.PI / 2.0, 0.0, Math.PI)

    -- we create a controlled hadamard gate
    val had_cgate = CGate(had_gate)

    -- we create two qubits and we shall apply gates to them
    val q1 = 0q0, q2 = 0q1

    -- create the |+> state using the hadamard gate
    apply(had_gate, ref q1)

    -- apply the controlled hadamard gate using q2 as control and q1 as target
    apply(had_cgate, ref q2, ref q1)


.. note::
    The SDK that comes with the compiler has a few builtin gates that you can find in the table below.
    So you do not need to create them. Please see the table at the end of this section to see the list of those gates.


.. warning::
    Controlled gates and the swap gate require that their two qubit references arguments not be identical and the SDK will enforce it.
    Unfortunately, at the moment, the gate will simply not be applied if for instance references to control qubit and the target qubit are identical.
    No error message will be emitted. This will be corrected in future versions of the SDK.


Measuring single qubits
~~~~~~~~~~~~~~~~~~~~~~~

Once you have applied unitary transformations on your qubit(s), it often desirable to measure them.
This also is very easy, just use the ``measure`` function. On single qubit variables, the measure function returns a value of type instance ``bit``.

.. code::

    -- initialize q to |0>
    val q = 0q0

    -- measure it
    val b = measure(ref q)


.. note::
    You can use the ``cast`` operator to perform measurement as this is implemented internally for you.
    It is done as follows: :code:`val b = cast(ref q) -> bit`.


List of standard 1-Qubit gates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please find below a table of gates that come with the SDK, their names, signatures and example usage.
All standard gates live in the ``quant`` package and are bound to the ``Quant`` namespaces

.. csv-table:: Standard gates
    :header: "Gate name", "Signature", "Example"
    :widths: auto

    "Indentity", "id(q : ref qubit) -> void", "Quant.id(ref q)"
    "Controlled identity", "cid(control : ref qubit, target : ref qubit) -> void", "Quant.cid(ref q1, ref q2)"
    "Pauli X", "px(q : ref qubit) -> void", "Quant.px(ref q)"
    "Controlled X", "cx(control : ref qubit, target : ref qubit) -> void", "Quant.cx(ref q1, ref q2)"
    "Pauli Y", "py(q : ref qubit) -> void", "Quant.py(ref q)"
    "Controlled X", "cy(control : ref qubit, target : ref qubit) -> void", "Quant.cy(ref q1, ref q2)"
    "Pauli Z", "pz(q : ref qubit) -> void", "Quant.pz(ref q)"
    "Controlled X", "cz(control : ref qubit, target : ref qubit) -> void", "Quant.cz(ref q1, ref q2)"
    "Rotation about X", "rx(q : ref qubit, theta : float) -> void", "Quant.rx(ref q, Math.PI)"
    "Controlled rotation about X", "crx(control : ref qubit, target : ref qubit, val theta : float) -> void", "Quant.crx(ref q1, ref q2, 0.0)"
    "Rotation about Y", "ry(q : ref qubit, theta : float) -> void", "Quant.ry(ref q, Math.PI / 2.0)"
    "Controlled rotation about Y", "cry(control : ref qubit, target : ref qubit, val theta : float) -> void", "Quant.cry(ref q1, ref q2, Math.PI / 2.0)"
    "Rotation about Z", "rz(q : ref qubit, phi : float) -> void", "Quant.rz(ref q, 0.0)"
    "Controlled rotation about Z", "crz(control : ref qubit, target : ref qubit, val phi : float) -> void", "Quant.crz(ref q1, ref q2, Math.PI)"
    "Phase", "phase(q : ref qubit, lambda : float) -> void", "Quant.phase(ref q, Math.PI / 8.0)"
    "Controlled phase", "cphase(control : ref qubit, target : ref qubit, val lambda : float) -> void", "Quant.cphase(ref q1, ref q2, Math.PI / 8.0)"
    "S", "s(q : ref qubit) -> void", "Quant.s(ref q)"
    "Controlled S", "cs(control : ref qubit, target : ref qubit) -> void", "Quant.cs(ref q1, ref q2)"
    "T", "t(q : ref qubit) -> void", "Quant.t(ref q)"
    "Controlled T", "ct(control : ref qubit, target : ref qubit) -> void", "Quant.ct(ref q1, ref q2)"
    "Hadamard", "had(q : ref qubit) -> void", "Quant.had(ref q)"
    "Controlled hadamard", "chad(control : ref qubit, target : ref qubit) -> void", "Quant.chad(ref q1, ref q2)"
    "Swap", "swap(control : ref qubit, target : ref qubit) -> void", "Quant.swap(ref q1, ref q2)"

