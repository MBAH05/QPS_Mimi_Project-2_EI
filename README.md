# Quantum State Preparation with Optimal T-Count

A complete **Qiskit** implementation of arbitrary *n*-qubit quantum state preparation, based on the paper:

> **"Quantum State Preparation with Optimal T-Count"**  
> David Gosset, Robin Kothari, Kewen Wu  
> [arXiv:2411.04790v2](https://arxiv.org/abs/2411.04790) (2025)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/quantum-state-preparation/blob/main/Quantum_State_Preparation.ipynb)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/downloads/)
[![Qiskit](https://img.shields.io/badge/Qiskit-2.x-6929C4.svg)](https://qiskit.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Problem

Given a normalized complex vector $\phi \in \mathbb{C}^{2^n}$ with $\|\phi\|_2 = 1$, construct a quantum circuit $U$ such that:

$$U|0\rangle^{\otimes n} = \sum_{x=0}^{2^n-1} \phi_x |x\rangle$$

**Constraints:** The circuit may use any number of ancilla qubits (returned to $|0\rangle$), arbitrary single-qubit gates, and multi-controlled $R_Z$ gates. No classical bits or mid-circuit measurements are allowed.

## Quick Start

### Google Colab (recommended)

1. Open the notebook: [`Quantum_State_Preparation.ipynb`](Quantum_State_Preparation.ipynb)  
2. Run the first cell to install dependencies  
3. Step through each section

### Local Installation

```bash
pip install qiskit qiskit-aer pylatexenc
```

```python
from quantum_state_prep_qiskit import prepare_state, verify_state
import numpy as np

# Define any target state
target = np.array([1, 1j, -1, -1j], dtype=complex)
target = target / np.linalg.norm(target)

# Build the circuit
circuit = prepare_state(target, n=2)
circuit.draw("mpl")

# Verify correctness
result = verify_state(circuit, target)
print(f"Fidelity: {result['fidelity']:.10f}")  # → 1.0000000000
```

## Results

All test cases achieve **exact fidelity** (machine-precision errors only):

| State | *n* | Fidelity | L₂ Error |
|---|---|---|---|
| Random complex | 4 | 1.0000000000 | ~10⁻¹⁶ |
| GHZ state | 4 | 1.0000000000 | ~10⁻¹⁶ |
| W state | 4 | 1.0000000000 | ~10⁻¹⁶ |
| Gaussian × phase | 4 | 1.0000000000 | ~10⁻¹⁶ |

## How It Works

The implementation uses a two-stage pipeline:

### Stage 1 — Magnitude Preparation (Grover-Rudolph)

Recursively decomposes the target amplitudes into a binary tree of controlled rotations:

1. Rotate qubit 0 to match the marginal probability of $x_0 = 0$ vs $x_0 = 1$
2. Controlled on qubit 0, rotate qubit 1 to match conditional marginals
3. Repeat for all *n* qubits

Each node in the tree becomes a multi-controlled $R_Y$ gate with `ctrl_state` specifying whether each control qubit must be $|0\rangle$ or $|1\rangle$.

### Stage 2 — Phase Correction (Diagonal Unitary)

After the magnitudes are correct, apply a diagonal unitary $D|x\rangle = e^{i\theta_x}|x\rangle$ to fix the complex phases. The phases are decomposed via **multilinear polynomial expansion** with Möbius inversion:

$$\theta(x) = \sum_{S \subseteq [n]} c_S \prod_{j \in S} x_j$$

Each coefficient $c_S$ maps to a multi-controlled Phase gate.

### Paper's Optimal Approach (Theoretical)

The paper achieves a near-square-root improvement over Grover-Rudolph using:

- **Coarse approximation** via Boolean phase oracles and the Khintchine inequality
- **Iterative refinement** with exponentially decaying error
- **Linear combination of unitaries (LCU)** to coherently combine approximations
- **Exact amplitude amplification** (2 Grover rounds)

This yields the optimal T-count $O\!\left(\sqrt{2^n \log(1/\varepsilon)} + \log(1/\varepsilon)\right)$.

## Complexity Comparison

| Metric | Grover-Rudolph (implemented) | Paper Optimal (Theorem 1.1) |
|---|---|---|
| **T-count** | $O(2^n \cdot (n + \log(1/\varepsilon)))$ | $O(\sqrt{2^n \log(1/\varepsilon)} + \log(1/\varepsilon))$ |
| **Clifford count** | $O(2^n \cdot n)$ | $O(2^n \cdot \log(1/\varepsilon))$ |
| **Ancillas** | $O(n)$ | $O(\sqrt{2^n \log(1/\varepsilon)})$ |
| **Best for** | Small *n*, NISQ devices | Fault-tolerant regime |

## Repository Structure

```
├── README.md                             # This file
├── Quantum_State_Preparation.ipynb       # Google Colab notebook (24 cells)
├── quantum_state_prep_qiskit.py          # Standalone Python module
└── LICENSE                               # MIT License
```

## API Reference

### `prepare_state(target, n=None) → QuantumCircuit`

Build a circuit that prepares the target state from $|0\rangle^{\otimes n}$.

**Parameters:**
- `target` — Complex numpy array of length $2^n$ (auto-normalized)
- `n` — Number of qubits (auto-detected from array length if `None`)

**Returns:** Qiskit `QuantumCircuit`

### `verify_state(circuit, target) → dict`

Verify a state-preparation circuit using exact statevector simulation.

**Returns:** Dictionary with keys:
- `fidelity` — $|\langle\psi_{\text{target}}|\psi_{\text{prepared}}\rangle|^2$
- `l2_error` — $\|\psi_{\text{prepared}} - \psi_{\text{target}}\|_2$
- `l2_aligned` — L₂ error after removing global phase mismatch
- `prepared` — The prepared statevector as a numpy array

### `build_boolean_phase_oracle(signs, n) → QuantumCircuit`

Construct a diagonal $\pm 1$ unitary from a sign vector using ANF decomposition.

### `build_diagonal_unitary(phases, n) → QuantumCircuit`

Construct a diagonal unitary $D|x\rangle = e^{i\theta_x}|x\rangle$ using multilinear polynomial decomposition.

### `grover_rudolph_real(amplitudes, n) → QuantumCircuit`

Prepare a real non-negative amplitude state via recursive controlled rotations.

## Notebook Sections

The notebook is organized into 10 self-contained sections:

| # | Section | Description |
|---|---|---|
| 1 | **Problem Statement** | Mathematical formulation, allowed operations, constraints |
| 2 | **Constructive Strategy** | Overview of Grover-Rudolph and the paper's optimal approach |
| 3 | **Key Subroutines** | Boolean phase oracles, diagonal synthesis, amplitude amplification |
| 4 | **Real-Amplitude Prep** | Grover-Rudolph with iterative refinement concept |
| 5 | **Complex Extension** | Magnitude/phase separation with diagonal correction |
| 6 | **Code Architecture** | API design, function hierarchy, recursive structure |
| 7 | **Gate Decompositions** | CRz, CCRz decomposition rules with verification |
| 8 | **Proof Sketch** | Inductive correctness proof with circuit illustrations |
| 9 | **Complexity Analysis** | Gate counts, asymptotic scaling, comparison table |
| 10 | **n=4 Demo** | Four test states with full verification |

## Requirements

- Python ≥ 3.10
- Qiskit ≥ 2.0
- pylatexenc (for circuit visualization)
- NumPy

## Citation

If you use this code, please cite the original paper:

```bibtex
@article{gosset2024quantum,
  title   = {Quantum State Preparation with Optimal T-Count},
  author  = {Gosset, David and Kothari, Robin and Wu, Kewen},
  journal = {arXiv preprint arXiv:2411.04790},
  year    = {2024},
  note    = {v2, October 2025}
}
```

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

## Acknowledgments

- The algorithms are based on the construction by Gosset, Kothari, and Wu, which builds on prior work by Low, Kliuchnikov, and Schaeffer (LKS24), Grover and Rudolph (GR02), and Rosenthal (Ros24).
- Implementation uses IBM's [Qiskit](https://qiskit.org/) framework.
