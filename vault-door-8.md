# vault-door-8

**difficulty:** hard

## the lock
the source is messy on purpose, with leftover commented-out code in between. here it is cleaned up to the parts that matter:

```java
public char[] scramble(String password) {
    char[] a = password.toCharArray();
    for (int b=0; b<a.length; b++) {
        char c = a[b];
        c = switchBits(c,1,2);
        c = switchBits(c,0,3);
        c = switchBits(c,5,6);
        c = switchBits(c,4,7);
        c = switchBits(c,0,1);
        c = switchBits(c,3,4);
        c = switchBits(c,2,5);
        c = switchBits(c,6,7);
        a[b] = c;
    }
    return a;
}

public char switchBits(char c, int p1, int p2) {
    /* Move the bit in position p1 to position p2, and move the bit
       that was in position p2 to position p1. Precondition: p1 < p2 */
    ...
}

public boolean checkPassword(String password) {
    char[] scrambled = scramble(password);
    char[] expected = { 0xF4, 0xC0, 0x97, 0xF0, 0x77, 0x97, 0xC0, 0xE4, 0xF0, 0x77, 0xA4, 0xD0, 0xC5, 0x77, 0xF4, 0x86, 0xD0, 0xA5, 0x45, 0x96, 0x27, 0xB5, 0x77, 0xA4, 0xD1, 0xF0, 0xE1, 0xD0, 0xE1, 0xD0, 0xA4, 0xB4 };
    return Arrays.equals(scrambled, expected);
}
```

## how i cracked it
this one looks scary, but it breaks down nicely. starting from `checkPassword`:

1. our password goes through `scramble`
2. `scramble` takes every character and runs 8 calls of `switchBits` on it
3. `switchBits(c, p1, p2)` swaps the bit at position `p1` with the bit at position `p2` (positions are counted from the right, starting at 0)
4. the result must equal `expected`

so we need the character that turns into each `expected` value after those 8 swaps. we have to undo the swaps.

**rewriting switchBits in python.** to swap two bits:

- `(val >> a) & 1` reads the bit at position `a`: shifting right by `a` puts it at the far right, and `& 1` keeps only that bit. do the same for `b`.
- compare the two bits with each other. if they're equal, swapping changes nothing. if they're different, swapping means flipping both.
- flipping is what xor does: `x ^ 1` flips a bit, `x ^ 0` leaves it alone. so we xor with a mask that has 1s only at positions `a` and `b`: `(1 << a) | (1 << b)`.

**the order.** the 8 swaps overlap each other (bit 1 is used in `(1,2)` and again in `(0,1)`, for example), so the order matters. to undo a chain of steps we undo them in reverse, like taking off shoes before socks. and a swap undoes itself, so each step can be reused as is. that's why we keep the pairs in a list and walk through it with `reversed`.

side note: the first four swaps touch each bit exactly once, and so do the last four, so the order *inside* each group doesn't matter. but between the two groups it does. running the steps in the source order on `expected` gives garbage.

## solve.py
```python
expected = [0xF4, 0xC0, 0x97, 0xF0, 0x77, 0x97, 0xC0, 0xE4, 0xF0, 0x77, 0xA4, 0xD0, 0xC5, 0x77, 0xF4, 0x86, 0xD0, 0xA5, 0x45, 0x96, 0x27, 0xB5, 0x77, 0xA4, 0xD1, 0xF0, 0xE1, 0xD0, 0xE1, 0xD0, 0xA4, 0xB4]

def switchBits(val, a, b):
    if ((val >> a) & 1) != ((val >> b) & 1):
        mask = (1 << a) | (1 << b)
        val ^= mask
    return val

decoded = []

steps = [(1,2),(0,3),(5,6),(4,7),(0,1),(3,4),(2,5),(6,7)]
for c in expected:
    for a, b in reversed(steps):
        c = switchBits(c, a, b)
    decoded.append(c)

print(bytes(decoded).decode('utf-8'))
```

- `steps` is the list of pairs, copied from `scramble` in the same order.
- `reversed(steps)` reads that list from the last pair to the first.
- `for a, b in ...` unpacks each pair into `a` and `b`.
- what we get back are numbers (bytes), not text. `bytes(decoded)` packs them together and `.decode('utf-8')` turns them into the password string.

## takeaways
- to undo a chain of steps, undo them in reverse order.
- swapping two bits = xor with a mask when they differ.
