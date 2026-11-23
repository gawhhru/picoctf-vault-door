# vault-door-1

**difficulty:** easy

## the lock
`main` is the same as in the training door, so let's go straight to `checkPassword`:

```java
public boolean checkPassword(String password) {
    return password.length() == 32 &&
           password.charAt(0)  == 'd' &&
           password.charAt(29) == 'e' &&
           password.charAt(4)  == 'r' &&
           password.charAt(2)  == '5' &&
           // ... 27 more lines like this
           password.charAt(31) == '2';
}
```

this time the password isn't written in one piece. the lock checks it one character at a time, and the checks are shuffled, so we can't just read it from top to bottom.

## how i cracked it
with 32 characters, we could rebuild the password by hand. but that gets painful fast, and later doors will have far more characters. so let's make python do the boring part.

every check has the same shape: `password.charAt(index) == 'character'`. each line tells us *"slot `index` holds `character`"*. so we read the file, pull out the index and the character, and drop each character into its slot.

## solve.py
```python
key = [""] * 32

for line in open("VaultDoor1.java"):
    if "charAt(" in line:
        i = int(line.split("charAt(")[1].split(")")[0])
        c = line.split("'")[1]
        key[i] = c

print ("".join(key))
```

how it works:

- **`split`** cuts a string at a given text and returns a list of the pieces. the `[ ]` after it picks one piece from that list, counting from 0.
- **`int()`** is needed because that `"29"` is still text, and a list index must be a number. so, `key["29"]` crashes, `key[29]` works.
- **`join`** glues a list of strings into one string, and the text in front of `.join` goes between the pieces.
  > for example, with `letters = ["a", "b", "c"]`: `"-".join(letters)` gives `"a-b-c"`

## takeaways
- when a check follows a pattern, let python do the boring part.
