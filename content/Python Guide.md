# Bits

Python has a number of built in types : numerics, sequences, mappings. Interestingly, integers in Python3 are unbounded - the maximum integer representable is a function of the available memory.

We can use the `sys.maxsize` variable to determine what is the maximum value integers that can be stored on a 64-bit machine.

Bit operations are 

 - `&` : This compares the last bit
 - `|` : This compares each of the coresponding bits
```
eg. 4 | 1 

0100
0001
----
0101
```

- `^` : This is a XOR operation and returns 1 for a bit if only one of the corresponding bits is equal to 1. Else it returns 0.

```
eg. 4 ^ 1 = 5

0100
0001
----
0101 = 5
```

- `<< n`: This shifts the bits of the left operand on the left by `n` bits and adds `n` 0 paddings. This results in a larger number.

```py
x = 2
x <<= 1 # This will return 4

0010 # Initial Value
0100 # Final Value
```

- `>> n` : Shifts the bits of the left operand to the right by `n` positions. This results in a smaller number


We can do some interesting stuff with this

## Determine the parity of a bit

```py
def parity(x: int) -> int:
    bit_count = 0
    while x:
        # If we find a 1, we flip the bit
        bit_count ^= x & 1
        x >>= 1
    return bit_count
```

Note here that if bit_count is 0, encountering a 1 will flip the bit to a 1. If bit count is 1, then encountering a 1 will flip the bit to a 0 since XOR requires only a single 1 or else it will return 0. At each step. 

At each point, we also make sure to decrement the `x` by shifting it one bit to the right by 1.