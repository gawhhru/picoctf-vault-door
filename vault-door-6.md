# vault-door-6

**difficulty:** medium

## the lock
```java
public boolean checkPassword(String password) {
    if (password.length() != 32) {
        return false;
    }
    byte[] passBytes = password.getBytes();
    byte[] myBytes = {
        0x3b, 0x65, 0x21, 0xa , 0x38, 0x0 , 0x36, 0x1d,
        0xa , 0x3d, 0x61, 0x27, 0x11, 0x66, 0x27, 0xa ,
        0x21, 0x1d, 0x61, 0x3b, 0xa , 0x2d, 0x65, 0x27,
        0xa , 0x6d, 0x31, 0x65, 0x63, 0x36, 0x66, 0x60,
    };
    for (int i=0; i<32; i++) {
        if (((passBytes[i] ^ 0x55) - myBytes[i]) != 0) {
            return false;
        }
    }
    return true;
}
```

each byte of our password gets xor-ed with `0x55`, and the result must match `myBytes`. the `- myBytes[i]) != 0` part just means "not equal", so the door wants `passBytes[i] ^ 0x55 == myBytes[i]`.

## how i cracked it
xor has two handy rules:

- `a ^ a = 0`
- `a ^ 0 = a`

so xor-ing something with the same key twice brings it back: `(p ^ k) ^ k = p ^ (k ^ k) = p ^ 0 = p`.

the door computed `password ^ 0x55 = myBytes`. xor both sides with `0x55` again and we get `password = myBytes ^ 0x55`. so we xor every byte of `myBytes` with `0x55` and turn it back into a character.

## solve.py
```python
byte = [
    0x3b, 0x65, 0x21, 0xa , 0x38, 0x0 , 0x36, 0x1d,
    0xa , 0x3d, 0x61, 0x27, 0x11, 0x66, 0x27, 0xa ,
    0x21, 0x1d, 0x61, 0x3b, 0xa , 0x2d, 0x65, 0x27,
    0xa , 0x6d, 0x31, 0x65, 0x63, 0x36, 0x66, 0x60,
]

password = "".join(chr(val ^ 0x55) for val in byte)

print(password)
```

`^` is xor in python too. `chr(val ^ 0x55) for val in byte` does that for every number in the list, and `join` sticks the characters together.

## takeaways
- xor with the same key twice cancels out, so the same operation encrypts and decrypts.
