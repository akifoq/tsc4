# TON Smart Challenge #4 top1 solutions

Here you can find the winner's solutions for each task. You can read a brief explanation of the solutions below:

## Task 1

Key ideas:
1. Store the dfs stack of cells-to-visit directly on the TVM stack
2. Use the `TRY` primitive to exit the loop in the case the stack is empty by stack underflow exception (that is, in the case we have traversed the whole tree but didn't find the requested cell)
3. Store the requested hash in a global variable to avoid unnecessary stack manipulations
4. Use `UNTIL` loop to automatically exit on equality of the hashes

## Task 2

It's slighty optimized FunC solution (with `TUPLEVAR` instead of `TPUSH` or `SETINDEXVAR`). There are better solutions due to smaller amount of stack manipulations.

## Task 3

Key ideas:
1. Handle the case of the flag occupying two consequent cells by rebuiling the current cell with the bits from the next cell on demand, so that the same read-from-one-cell loop can be used.
2. Use the quiet primitives `PLDUXQ` and `STSLICERQ` to read and write data to avoid checking the size of slices/builders in advance.

You can get a better solution using `SDPFX` or `SDBEGINSXQ` instead of `PLDUXQ`.

## Task 4 

The key idea is to use the following bit magic to process 32 bytes at a time:

```
const int shift = 15; ;; your shift here

int main(int x) {
    int mask7 = 0x8080808080808080808080808080808080808080808080808080808080808080;
    int mask0 = 0x0101010101010101010101010101010101010101010101010101010101010101;
    int mask5 = 0x2020202020202020202020202020202020202020202020202020202020202020;

    int nx5 = (~ x) & mask5;
    x = x | mask5;
    int nx7 = (~ x) & mask7;
    
    int x' = x | mask7;
    int x_gt = x' - (mask0 * 97);
    int x_mid = x' - (mask0 * (123 - shift));

    int x+ = nx7 & x_gt & (~ x_mid);
    x += (x+ >> 7) * shift;

    int x_lt = x' - (mask0 * 123);
    int x- = nx7 & x_mid & (~ x_lt);
    x -= (x- >> 7) * (26 - shift);

    x ^= nx5;
    return x;
}
```

The magic uses the fact that uppercase and lowercase letters in ASCII differ only in the 5th bit. It sets the most significant bit to 1 in every byte and then subtracts a lower bound (97, 123 - shift or 123) to get the mask of bytes which are at least the subtracted number, then perfom the shift only for bytes needed based on the obtained mask.

## Task 5

The key idea is to use `IFBITJMPREF` to obtain the n-th and (n + 1)-th Fibonacci numbers, then use `REPEAT:<{ 2DUP ADD }>` to obtain the answer and `TUPLEVAR` to pack it in the tuple.

Here is the python script I used to generate the `IFBITJMPREF` table:
```
def gen_dict(n):
    a = 0
    b = 1
    res = {}
    for i in range(n + 1):
        res[i] = a
        a, b = b, a + b
    res[371] = 0
    return res

def gen_asm(dict, bits, l, x):
    if bits == l:
        if x in dict and x + 1 in dict:
            return "DROP " + str(dict[x]) + " PUSHINT " + str(dict[x + 1]) + " PUSHINT"
        else:
            return ""

    r0 = gen_asm(dict, bits + 1, l, x * 2)
    r1 = gen_asm(dict, bits + 1, l, x * 2 + 1)

    if r0 == "":
        return r1
    if r1 == "":
        return r0

    return "<{ " + r1 + " }>c " + str(l - 1 - bits) + " IFBITJMPREF " + r0

d = gen_dict(370)
table = gen_asm(d, 0, 9, 0)
print(table)
```
