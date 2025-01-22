from qiskit import QuantumCircuit, QuantumRegister, ClassicalRegister
from qiskit_aer import Aer  # Updated import
from qiskit.visualization import plot_histogram
from qiskit.compiler import transpile, assemble
import numpy as np

# Create quantum registers (2 qubits)
q = QuantumRegister(2, 'q')
# Create classical registers (2 bits) to store measurements
c = ClassicalRegister(2, 'c')

# Create quantum circuit
circuit = QuantumCircuit(q, c)

# Create an entangled pair (Bell state)
# First, put the first qubit in superposition
circuit.h(q[0])
# Then, CNOT gate to entangle both qubits
circuit.cx(q[0], q[1])

# Add measurements
circuit.measure(q, c)

# Use Aer's qasm_simulator
simulator = Aer.get_backend('qasm_simulator')

# Execute the circuit on the simulator
# Updated execution method
transpiled_circuit = transpile(circuit, simulator)
job = simulator.run(transpiled_circuit, shots=1000)
result = job.result()

# Get the measurement results
counts = result.get_counts(circuit)

def generate_key(measurement_results, basis_alice, basis_bob):
    """
    Generate a shared key from measurement results using matching basis
    """
    key = ''
    for i in range(len(measurement_results)):
        # Only keep results where Alice and Bob used the same basis
        if basis_alice[i] == basis_bob[i]:
            key += measurement_results[i]
    return key

# Simulate quantum key distribution
def quantum_key_distribution(n_bits):
    # Generate random basis choices for Alice and Bob
    alice_basis = np.random.randint(2, size=n_bits)  # 0 for Z basis, 1 for X basis
    bob_basis = np.random.randint(2, size=n_bits)    # 0 for Z basis, 1 for X basis
    
    # Generate random bits for Alice
    alice_bits = np.random.randint(2, size=n_bits)
    
    # Create a circuit for each bit
    shared_key = []
    for i in range(n_bits):
        qc = QuantumCircuit(2, 2)
        
        # Prepare Alice's qubit
        if alice_bits[i]:
            qc.x(0)
        if alice_basis[i]:
            qc.h(0)
            
        # Entangle the qubits
        qc.cx(0, 1)
        
        # Bob's measurement basis
        if bob_basis[i]:
            qc.h(1)
            
        # Measure
        qc.measure_all()
        
        # Execute with updated method
        transpiled_qc = transpile(qc, simulator)
        result = simulator.run(transpiled_qc, shots=1).result()
        measurement = list(result.get_counts().keys())[0]
        shared_key.append(measurement[1])  # Keep Bob's measurement
        
    return shared_key, alice_basis, bob_basis

# Example usage
n_bits = 20
results, alice_basis, bob_basis = quantum_key_distribution(n_bits)
shared_key = generate_key(results, alice_basis, bob_basis)

print(f"Circuit measurement counts: {counts}")
print(f"\nQuantum Key Distribution Results:")
print(f"Number of bits generated: {n_bits}")
print(f"Alice's basis: {alice_basis}")
print(f"Bob's basis: {bob_basis}")
print(f"Shared key: {shared_key}")
