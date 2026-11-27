# vault-door-7

**difficulty:** hard

## the lock
```java
public int[] passwordToIntArray(String hex) {
    int[] x = new int[8];
    byte[] hexBytes = hex.getBytes();
    for (int i=0; i<8; i++) {
        x[i] = hexBytes[i*4]   << 24
             | hexBytes[i*4+1] << 16
             | hexBytes[i*4+2] << 8
             | hexBytes[i*4+3];
    }
    return x;
}

public boolean checkPassword(String password) {
    if (password.length() != 32) {
        return false;
    }
    int[] x = passwordToIntArray(password);
    return x[0] == 1096770097
        && x[1] == 1952395366
        // ... 5 more lines like this
        && x[7] == 909534257;
}
```

this time the password is never compared as text. the door turns it into 8 numbers first, and those numbers are what it checks.

## how i cracked it
the idea is **packing**. a character is one byte (8 bits), and an `int` holds 32 bits, so 4 characters fit into one int, laid end to end.

| char of the group | shifted by | position in the int |
|-------------------|-----------|---------------------|
| 1st | `<< 24` | leftmost byte |
| 2nd | `<< 16` | second byte |
| 3rd | `<< 8` | third byte |
| 4th | no shift | rightmost byte |

`<<` slides the bits to the left, and `|` merges the four shifted bytes into one number. for example, `"01ab"` is the bytes `0x30 0x31 0x61 0x62`, and glued together they become `00110000 00110001 01100001 01100010`, which is the number `808542562`.

32 characters make 8 groups of 4, which is why there are 8 ints in the check.

to reverse it, we take each int and split it back into its 4 bytes, leftmost first. python can do that in one call: `to_bytes(4, byteorder="big")`. `"big"` means the biggest (leftmost) byte comes first, which matches the `<< 24` of the first character.

## solve.py
```python
x = [
    1096770097, 1952395366, 1600270708, 1601398833,
    1716808014, 1734293602, 895574369, 909534257,
]

b = bytearray()
for val in x:
    b.extend(val.to_bytes(4, byteorder="big", signed=False))

print(b.decode("utf-8"))
```

- `bytearray()` starts an empty, editable sequence of bytes.
- `val.to_bytes(4, ...)` turns one int into exactly 4 bytes. `signed=False` means the number is never negative.
- `b.extend(...)` adds those 4 bytes to the end, so after 8 rounds we have all 32 bytes.
- `b.decode("utf-8")` turns the bytes back into a string.

## extra: bytearray cheat sheet
a `bytearray` is a mutable sequence of bytes, each one an integer from 0 to 255. it's handy for building or editing raw binary data. only `extend` was needed here, but these will come in handy later:

| method | what it does |
|--------|--------------|
| `append(int)` | adds one byte to the end |
| `extend(iterable)` | adds many bytes (from `bytes`, `bytearray` or a list of ints) |
| `insert(index, int)` | puts one byte at a given position |
| `remove(int)` | removes the **first** byte with that value |

and `int.to_bytes(length, byteorder, *, signed=False)` turns an int into `length` bytes, which is what we used above.

## takeaways
- shifting by 8, 16, 24 is how bytes get packed into one int, and `to_bytes` unpacks them.
- `byteorder="big"` means the leftmost byte comes first.
