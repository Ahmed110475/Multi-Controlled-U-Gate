# Multi-Controlled-U-Gate
Decomposition of an n-controlled unitary gate using a linear Toffoli cascade structure in Qiskit.

> The goal is to write a Qiskit function that takes two inputs: a positive integer $n$ and a matrix $U \in \text{U}(2)$ and outputs a quantum circuit on $n + 1$ qubits, possibly with further ancillas, that implements a multi-controlled $U$ gate, $C^n U$, that is
> $$C^n U |x\rangle_n |y\rangle_1 = \begin{cases} |x\rangle_n U|y\rangle_1, & \text{if } x = (1, 1, \dots, 1), \\ |x\rangle_n |y\rangle_1, & \text{otherwise.} \end{cases}$$
> 
> 
> The construction may use any number of ancillas, arbitrary 1-qubit gates, $CX$, and Toffoli gates. No classical bit and measurements allowed.

---

## 1. Algorithmic Strategy & Method

To implement the $C^n U$ operation using the allowed gates, this project utilizes a **Toffoli cascade (V-chain)** method. It splits the logic into three consecutive phases to compute the state, apply the gate, and safely clean up the workspace:

1. **Phase A: Compute (The Cascade):** A sequential chain of Toffoli ($CCX$) gates progressively evaluates the condition $x = (1, 1, \dots, 1)$ by checking pairs of control qubits and storing their running logical **AND** product into $n-1$ helper ancilla qubits.
2. **Phase B: Action:** The final ancilla qubit (`ancilla_{n-2}`) functions as a master switch that triggers a 1-controlled version of your custom matrix $U$ on the target qubit $|y\rangle_1$.
3. **Phase C: Uncompute:** The exact same Toffoli sequence is executed in absolute reverse order. This resets all ancillas back to their initial state, successfully avoiding phase kickback and preventing unwanted background entanglement.

---

## 2. Resource & Complexity Analysis

The resource requirements for this V-chain decomposition method scale dynamically at a strictly linear rate as a function of the number of control qubits $n$:

| Metric | Complexity | Description |
| --- | --- | --- |
| **Total Qubits** | $2n$ | Requires $n$ control qubits, $1$ target qubit, and $n-1$ ancillas (for $n \ge 2$). |
| **Gate Count** | $O(n)$ | Synthesizes exactly $2(n-1)$ Toffoli gates plus $1$ single-controlled $U$ gate. |
| **Circuit Depth** | $O(n)$ | Because each Toffoli gate relies sequentially on the previous ancilla's calculation, they cannot run in parallel. Depth directly mirrors the gate count. |

---

## 3. Script Structure & Validation

The implementation is a lightweight, standalone Python script designed to execute seamlessly in a single Jupyter Notebook cell:

* **Configuration Interface:** Placed at the very top of the script for direct adjustment of the controls variable (`num_controls`) and the unitary target array (`U_matrix`).
* **Automated Sanity Testing:** Prior to constructing the circuit, the function runs two algorithmic checks using `numpy`:
1. Dimension validation (ensures matrix is a $2 \times 2$ shape).
2. Unitarity validation (uses `np.allclose` to confirm $U U^\dagger = I$).


* **Metrics Dashboard:** Leverages Qiskit’s native `circuit.depth()` and `circuit.count_ops()` tools to output exact metrics and draw a clean, text-based map of your circuit.
