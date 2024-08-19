# Baby Bundle

Point count: 306 pts

Challenge author: A~Z

Description: A crane flew by, and delivered this baby chall. I can't understand a word it speaks.

Provided files: `chall.sage`, `out.txt`, and `README.md`

Solver: wjaaaaaaat

Writeup author: wjaaaaaaat

## Writeup

First, let's break down the source code.
```py
# Patch deprecation warnings
sage.structure.element.is_Matrix = lambda z: isinstance(z, sage.structure.element.Matrix)
# See README.md for this package
from vector_bundle import *
from string import printable
from tqdm import tqdm
```
This is just package installation, but caused quite a headache during the CTF. Using this Dockerfile:
```Dockerfile
from sagemath/sagemath:latest
RUN sudo apt update
RUN sudo apt install -y make
RUN sudo apt install -y wget
RUN sudo apt install -y git
RUN sudo apt install -y python3
RUN wget https://bootstrap.pypa.io/get-pip.py
RUN sage -python3 get-pip.py
RUN git clone https://git.disroot.org/montessiel/vector-bundles-sagemath.git
RUN cd vector-bundles-sagemath && \
    sudo make install
RUN sage -pip install tqdm
```
and running these commands with the `Dockerfile` in current directory:
```bash
docker build -t imsage .
docker run -it imsage
```
creates a Docker container with Sagemath 10.4 and all the packages used in the source code, including the nasty-to-install `vector_bundle`.
```
password = ''.join(choice(printable) for _ in range(15)).encode()

p = 66036476783091383193200018291948785097
F = GF(p)
K.<x> = FunctionField(F)
L = VectorBundle(K, -x.zeros()[0].divisor()) # L = O(-1)
```
This sets up:
- `password`, bytes representing a random 15-length string containing printable characters,
- `F = GF(p)`, also known as Z_p, the finite field to calculate on,
- `K.<x> = FunctionField(F)`, a field of rational functions on `F`, and
- `L`, which is `O(-1)`.
You might be thinking: what is `O(-1)`? What the heck is a vector bundle? What even is a field? Before this CTF, I only knew the answer to that last question. Let's take a brief pause to explain the mechanics on a very basic level. (This is a massive oversimplification; mathematicians - don't hate me.)

Z_p is basically integers mod p. A ring is a number system where addition and multiplication behave nicely. A field is a ring where everything has a reciprocal, except 0.
A vector bundle is a complicated concept; I'm not in a great position to explain it. There are operations called the direct sum and tensor product. Let the direct sum be denoted by typical algebraic addition notation, and let the tensor product be denoted by typical algebraic multiplication notation. The tensor power is repetition of the tensor product; if '^' denotes the tensor power, we have X^3 = X*X*X.

These operations are very easy to work with. We have O(a)O(b) = O(a+b). Note that this means O(n)^k = O(kn). We also have the typical distributive property: O(a)(O(b)+O(c)) = O(a)O(b)+O(b)O(c).
There's also a function called h0, the set of global sections of a vector bundle. Here's what you need to know: (|| is the cardinality/size of a set) |h0(O(n))| = n+1 for nonnegative n, and 0 otherwise. We also have |h0(A+B)| = |h0(A)|+|h0(B)| for A and B composed of O(n), direct sums, and tensor powers.

Alright, back to source code analysis.

```py
V = L.tensor_power(password[0])
for b in tqdm(password[1:]):
    V = V.direct_sum(L.tensor_power(b))
```
Now, V is of the form O(-n_1) + O(-n_2) + ... + O_(-n_{15}) where the n_i are the byte values in `password`.
```py
L = L.dual() # L = O(1)
out = [
    len(V.tensor_product(L.tensor_power(m)).h0())
    for m in tqdm(printable.encode())
]

print(out)
```
We redefine `L` to be O(1). Using what was explained above, `len(V.tensor_product(L.tensor_power(m)).h0())` is |h0(V*O(m))| for all `chr(m) in printable`. Note that incrementing `m` to `m+1` (given that they are both `in printable.encode()`) increases this quantity by the number of (not necessarily unique) characters `c` in `password` such that `ord(c) <= m+1`. If we examine the rate of change of this quantity (the change between `m` and `m+1`), we find that the rate of change changes by k on `m` to `m+1` where `k` is the number of occurences of `m+1` in `password.encode()`. So, we can let `b = [[out[printable.index(chr(i))],chr(i)] if chr(i) in printable else None for i in range(128)]` to examine values. Printing `b` gives:
```py
[None, None, None, None, None, None, None, None, None, [0, '\t'], [1, '\n'], [2, '\x0b'], [3, '\x0c'], [4, '\r'], None, None, None, None, None, None, None, None, None, None, None, None, None, None, None, None, None, None, [23, ' '], [24, '!'], [25, '"'], [26, '#'], [27, '$'], [28, '%'], [29, '&'], [31, "'"], [33, '('], [35, ')'], [37, '*'], [39, '+'], [41, ','], [43, '-'], [45, '.'], [47, '/'], [49, '0'], [52, '1'], [55, '2'], [58, '3'], [62, '4'], [66, '5'], [71, '6'], [76, '7'], [81, '8'], [86, '9'], [91, ':'], [96, ';'], [101, '<'], [106, '='], [113, '>'], [120, '?'], [127, '@'], [134, 'A'], [141, 'B'], [148, 'C'], [155, 'D'], [162, 'E'], [169, 'F'], [176, 'G'], [184, 'H'], [192, 'I'], [200, 'J'], [208, 'K'], [216, 'L'], [224, 'M'], [232, 'N'], [240, 'O'], [248, 'P'], [257, 'Q'], [266, 'R'], [275, 'S'], [284, 'T'], [293, 'U'], [303, 'V'], [313, 'W'], [323, 'X'], [333, 'Y'], [345, 'Z'], [357, '['], [369, '\\'], [381, ']'], [393, '^'], [405, '_'], [418, '`'], [431, 'a'], [444, 'b'], [457, 'c'], [470, 'd'], [484, 'e'], [498, 'f'], [512, 'g'], [526, 'h'], [540, 'i'], [554, 'j'], [568, 'k'], [582, 'l'], [596, 'm'], [610, 'n'], [625, 'o'], [640, 'p'], [655, 'q'], [670, 'r'], [685, 's'], [700, 't'], [715, 'u'], [730, 'v'], [745, 'w'], [760, 'x'], [775, 'y'], [790, 'z'], [805, '{'], [820, '|'], [835, '}'], [850, '~'], None]
```
The rate of change changes:
- by `1` at `\n`
- by 1 at `'`
- by 1 at `1`
- by 1 at `4`
- by 1 at `6`
- by 2 at `>`
- by 1 at `H`
- by 1 at `Q`
- by 1 at `V`
- by 2 at `Z`
- by 1 at the backtick character
- by 1 at `e`
- by 1 at `o`
So, `password` is some permutation of `\n'146>>HQVZZeo` and a backtick before `f`.

```py
from Crypto.Cipher import AES
from hashlib import sha256
from flag import flag
flag += bytes((16-len(flag)) % 16)

key = sha256(bytes(sorted(password))).digest()[:16]
aes = AES.new(key, AES.MODE_ECB)
enc = aes.encrypt(flag)
print('enc:', enc.hex())
```
As it turns out, we don't need to know the order, as it is sorted when used in the key. Modifying the script:
```py
from Crypto.Cipher import AES
from hashlib import sha256
from Crypto.Util.number import *
password = b'\n\'146>>HQVZZ`eo'
ct = long_to_bytes(int('5f0a8761f98748422d97f60f11d8590d56e1462409a677fbf52259b084b8a724',16))
key = sha256(bytes(sorted(password))).digest()[:16]
aes = AES.new(key, AES.MODE_ECB)
dec = aes.decrypt(ct)
print('dec:', dec)
```

Final flag:
`idek{R34dy_f0r_m0r3?}`