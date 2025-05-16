# Quantum Random Token Generator

A Python-based quantum random token generator using Qiskit, designed to demonstrate quantum randomness, enable secure alphanumeric token generation, and serve as an educational resource on quantum circuit design.

## 🔍 Overview

This project creates high-entropy, quantum-generated random tokens using a multi-layer quantum circuit and simulates results with Qiskit's Aer simulator. It also evaluates the statistical quality of the randomness using standard tests.

## ✨ Features

- 🧠 **Quantum Circuit Design**: Utilizes Hadamard gates, entanglement via RZZ and CNOT gates, Toffoli gates, and random single-qubit rotations.
- 🔐 **Alphanumeric Token Output**: Produces tokens with characters from A-Z, a-z, and 0-9.
- 🧪 **Statistical Testing**: Built-in Frequency and Runs tests evaluate bitstring randomness.
- ⚙️ **Customizable**: Adjust token length, number of quantum shots, and circuit depth via parameters.

## 📦 Installation

Install the required Python packages with:

```bash
pip install qiskit qiskit-aer numpy scipy
```

## 🚀 Usage

Here's a full example that generates 10 random tokens (32 characters each) and evaluates their randomness:

```python
from qiskit import QuantumCircuit
from qiskit_aer import Aer
from qiskit.circuit.library import RZZGate
import numpy as np
from scipy.stats import chi2, norm

# Character set and helper
CHAR_SET = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
CHAR_COUNT = len(CHAR_SET)

def bits_to_char(bits):
    value = int(bits, 2) % CHAR_COUNT
    return CHAR_SET[value]

def generate_random_token(num_chars=32, shots=10, qubits_per_batch=12):
    bits_per_char = 6
    total_bits = num_chars * bits_per_char
    bits_per_run = qubits_per_batch
    num_runs = (total_bits + bits_per_run - 1) // bits_per_run
    tokens, all_bitstrings = [], []

    for shot in range(shots):
        token_bits = ''
        for _ in range(num_runs):
            qc = QuantumCircuit(qubits_per_batch, qubits_per_batch)
            for i in range(qubits_per_batch): qc.h(i)
            for layer in range(2):
                for i in range(0, qubits_per_batch - 1, 2):
                    qc.append(RZZGate(theta=np.pi / (2 ** layer)), [i, i+1])
                for i in range(0, qubits_per_batch - 1):
                    qc.cx(i, (i + 1) % qubits_per_batch)
                for i in range(qubits_per_batch):
                    qc.rx(np.pi * np.random.random(), i)
                    qc.ry(np.pi * np.random.random(), i)
            for i in range(0, qubits_per_batch - 2, 3):
                qc.ccx(i, i+1, i+2)
            for i in range(qubits_per_batch): qc.h(i)
            qc.measure(range(qubits_per_batch), range(qubits_per_batch))
            simulator = Aer.get_backend('qasm_simulator')
            result = simulator.run(qc, shots=1).result()
            token_bits += list(result.get_counts().keys())[0]

        token_bits = token_bits[:total_bits]
        all_bitstrings.append(token_bits)
        token = ''.join(bits_to_char(token_bits[i:i+bits_per_char])
                        for i in range(0, total_bits, bits_per_char))
        tokens.append(token[:num_chars])
    return tokens, all_bitstrings

def frequency_test(bitstring):
    n = len(bitstring)
    ones = bitstring.count('1')
    zeros = n - ones
    expected = n / 2
    chi_square = ((ones - expected) ** 2 + (zeros - expected) ** 2) / expected
    p_value = 1 - chi2.cdf(chi_square, df=1)
    return p_value > 0.05, p_value

def runs_test(bitstring):
    n = len(bitstring)
    runs = sum(bitstring[i] != bitstring[i-1] for i in range(1, n)) + 1
    ones = bitstring.count('1')
    zeros = n - ones
    if ones == 0 or zeros == 0: return False, 0.0
    expected_runs = 2 * ones * zeros / n + 1
    variance = (2 * ones * zeros * (2 * ones * zeros - n)) / (n ** 2 * (n - 1))
    z = abs(runs - expected_runs) / np.sqrt(variance)
    p_value = 2 * (1 - norm.cdf(z))
    return p_value > 0.05, p_value

# Run example
tokens, bitstrings = generate_random_token()
print("\nGenerated Tokens:")
for i, token in enumerate(tokens):
    print(f"Token {i+1}: {token}")

print("\nRandomness Quality Tests:")
for i, bits in enumerate(bitstrings):
    freq_pass, freq_p = frequency_test(bits)
    runs_pass, runs_p = runs_test(bits)
    print(f"Bitstring {i+1}:")
    print(f"  Frequency Test: {'Pass' if freq_pass else 'Fail'} (p-value: {freq_p:.4f})")
    print(f"  Runs Test: {'Pass' if runs_pass else 'Fail'} (p-value: {runs_p:.4f})")
```

## 📝 License

This project is licensed under the MIT License.
