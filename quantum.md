## Challenge
We are simulating a Quantum satellite that can exchange keys using qubits implementing BB84. You must POST the qubits and basis of measurement to `/qkd/qubits` and decode our satellite response, you can then derive the shared key and decrypt the flag. Send 512 qubits and basis to generate enough key bits.

### This is the server's code:

```
import random
import numpy

from math import sqrt
from flask import current_app

def rotate_45(qubit):
  return qubit * complex(0.707, -0.707)

def measure(rx_qubits, basis):
  measured_bits = ""
  for q, b in zip(rx_qubits, basis):
    if b == 'x':
      q = rotate_45(q)
    probability_zero = round(pow(q.real, 2), 1)
    probability_one = round(pow(q.imag, 2), 1)
    measured_bits += str(numpy.random.choice(numpy.arange(0, 2), p=[probability_zero, probability_one]))
  return measured_bits

def compare_bases_and_generate_key(tx_bases, rx_bases, measure):
  """Compares TX and RX bases and return the selected bits."""
  if not (len(tx_bases) == len(rx_bases) == len(measure)):
    return None, "tx_bases(%d), rx_bases(%d) and measure(%d) must have the same length." % (len(tx_bases), len(rx_bases), len(measure))
  ret = ''
  for bit, tx_base, rx_base in zip(measure, tx_bases, rx_bases):
    if tx_base == rx_base:
      ret += bit
  return ret, None

def unmarshal(qubits):
  return [complex(q['real'], q['imag']) for q in qubits]

# Receive user's qubits and basis, return the derived key and our basis.
def perform(rx_qubits, rx_basis):
  random.seed()
  # Multiply the amount of bits in the encryption key by 4 to obtain the amount of basis.
  sat_basis = [random.choice('+x') for _ in range(len(current_app.config['ENCRYPTION_KEY'])*16)]
  measured_bits = measure(unmarshal(rx_qubits), sat_basis)
  binary_key, err = compare_bases_and_generate_key(rx_basis, sat_basis, measured_bits)
  if err:
    return None, None, err
  # ENCRYPTION_KEY is in hex, so multiply by 4 to get the bit length.
  binary_key = binary_key[:len(current_app.config['ENCRYPTION_KEY'])*4]
  if len(binary_key) < (len(current_app.config['ENCRYPTION_KEY'])*4):
    return None, sat_basis, "not enough bits to create shared key: %d want: %d" % (len(binary_key), len(current_app.config['ENCRYPTION_KEY']))
  return binary_key, sat_basis, None
```  
  
### How to send qubits
POST your qubits in JSON format the following way:

basis: List of '+' and 'x' which represents the axis of measurement. Each basis measures one qubit:
+: Normal axis of measurement.
x: π/4 rotated axis of measurement.
qubits: List of qubits represented by a dict containing the following keys:
real: The real part of the complex number (int or float).
imag: The imaginary part of the complex number (int or float).

The satellite responds:

basis: List of '+' and 'x' used by the satellite.
announcement: Shared key (in hex), the encryption key is encoded within this key.

### Example decryption with hex key 404c368bf890dd10abc3f4209437fcbb:
echo "404c368bf890dd10abc3f4209437fcbb" > /tmp/plain.key; xxd -r -p /tmp/plain.key > /tmp/enc.key

echo "U2FsdGVkX182ynnLNxv9RdNdB44BtwkjHJpTcsWU+NFj2RfQIOpHKYk1RX5i+jKO" | openssl enc -d -aes-256-cbc -pbkdf2 -md sha1 -base64 --pass file:/tmp/enc.key

Flag (base64)
U2FsdGVkX19OI2T2J9zJbjMrmI0YSTS+zJ7fnxu1YcGftgkeyVMMwa+NNMG6fGgjROM/hUvvUxUGhctU8fqH4titwti7HbwNMxFxfIR+lR4=


# Problems faced
For `echo "U2FsdGVkX182ynnLNxv9RdNdB44BtwkjHJpTcsWU+NFj2RfQIOpHKYk1RX5i+jKO" | openssl enc -d -aes-256-cbc -pbkdf2 -md sha1 -base64 --pass file:/tmp/enc.key`, I could not parse the openssl on my Ubuntu on GCP. Probably we need a proper crypto linux machine. 
`--pass` changed to `-pass` makes that part ok, but I couldn't figure out how to adapt `-pbkdf2`. Google search did not show helpful results.

Without having this to work, I went to other question. It seems that this question is considered easy because it is about following instructions.