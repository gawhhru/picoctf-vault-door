# vault-door-4

**difficulty:** medium

## the lock
```java
public boolean checkPassword(String password) {
    byte[] passBytes = password.getBytes();
    byte[] myBytes = {
        106 , 85  , 53  , 116 , 95  , 52  , 95  , 98  ,
        0x55, 0x6e, 0x43, 0x68, 0x5f, 0x30, 0x66, 0x5f,
        0142, 0131, 0164, 063 , 0163, 0137, 070 , 066 ,
        'c' , '8' , '4' , '3' , 'f' , 'd' , '1' , '0' ,
    };
    for (int i=0; i<32; i++) {
        if (passBytes[i] != myBytes[i]) {
            return false;
        }
    }
    return true;
}
```

the lock compares our password with `myBytes`, byte by byte. so the password is just those 32 bytes turned back into characters. the only twist is that the array writes its numbers in four different ways.

## how i cracked it
every entry is a number, just written differently:

| written as | meaning |
|------------|---------|
| `106` | plain decimal |
| `0x55` | hex (starts with `0x`) |
| `0142` | octal (starts with `0`) |
| `'c'` | a character, which java treats as its number |

python understands all of these too, with one change: octal starts with `0o` instead of `0`, so `0142` becomes `0o142`.

once everything is a number, `chr()` turns a number into its character. for the entries that are already characters like `'c'`, `ord()` does the opposite and gives the number, so `chr(ord('c'))` is just `'c'` again. this way the whole list is numbers and one `chr` handles it all.

## solve.py
```python
nums = [
    106, 85, 53, 116, 95, 52, 95, 98,
    0x55, 0x6e, 0x43, 0x68, 0x5f, 0x30, 0x66, 0x5f,
    0o142, 0o131, 0o164, 0o63, 0o163, 0o137, 0o70, 0o66,
    ord('c'), ord('8'), ord('4'), ord('3'), ord('f'), ord('d'), ord('1'), ord('0')
]

print("".join(chr(n) for n in nums))
```

## takeaways
- hex, octal and decimal are the same numbers in different clothes.
- `chr()` and `ord()` are opposites.
