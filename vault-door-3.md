# vault-door-3

**difficulty:** medium

## the lock
```java
public boolean checkPassword(String password) {
    if (password.length() != 32) {
        return false;
    }
    char[] buffer = new char[32];
    int i;
    for (i=0; i<8; i++) {
        buffer[i] = password.charAt(i);
    }
    for (; i<16; i++) {
        buffer[i] = password.charAt(23-i);
    }
    for (; i<32; i+=2) {
        buffer[i] = password.charAt(46-i);
    }
    for (i=31; i>=17; i-=2) {
        buffer[i] = password.charAt(i);
    }
    String s = new String(buffer);
    return s.equals("jU5t_a_sna_3lpm10g747_u_4_m7re49");
}
```

our password has 32 characters, but this time it gets shuffled into `buffer` before being compared with the target string. so the door isn't asking for the target itself, it's asking for the password that turns into the target after the shuffle.

## how i cracked it
the shuffle is just four loops, and we can read each one:

| loop | what it does |
|------|--------------|
| `i` from 0 to 7 | `buffer[i] = password[i]`, nothing moves |
| `i` from 8 to 15 | `buffer[i] = password[23-i]`, the block gets reversed |
| `i` from 16 to 30, step 2 | `buffer[i] = password[46-i]`, even slots get reversed |
| `i` from 31 down to 17, step -2 | `buffer[i] = password[i]`, odd slots stay |

note that the second loop doesn't reset `i`, it continues from where the first loop stopped (8).

we want to go backwards: we know `buffer` (the target) and need `password`. luckily every move here is a swap (`i` and `23-i` trade places, same for `i` and `46-i`), and undoing a swap is the same swap. so we can reuse the exact same formulas, just with the sides flipped. rewriting the loops in python does the job.

## solve.py
```python
target = "jU5t_a_sna_3lpm10g747_u_4_m7re49"

key = [""] * 32

for i in range (0,8):
    key[i] = target[i]

for i in range (8,16):
    key[i] = target[23-i]

for i in range (16,32,2):
    key[i] = target[46-i]

for i in range (31,16,-2):
    key[i] = target[i]

print("".join(key))
```

about the range thing:

`range(start, stop, step)` counts from `start` in jumps of `step`, and **stops before `stop`**, so the `stop` value itself is never included.

> notice the the bolded part, that's why `16` and not `17` in the last loop. the java loop is `i>=17`, which includes 17. but python's `stop` is excluded, so we have to go one past it. with `range(31, 17, -2)` we'd get 31, 29, ..., 19 and miss 17, leaving that slot empty and the password one character short.

## takeaways
- an excluded `stop` in `range` is an easy off-by-one trap, check the last value.
- when a shuffle only swaps positions, the same formula undoes it.
