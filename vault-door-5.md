# vault-door-5

**difficulty:** medium

## the lock
```java
public String base64Encode(byte[] input) {
    return Base64.getEncoder().encodeToString(input);
}

public String urlEncode(byte[] input) {
    StringBuffer buf = new StringBuffer();
    for (int i=0; i<input.length; i++) {
        buf.append(String.format("%%%2x", input[i]));
    }
    return buf.toString();
}

public boolean checkPassword(String password) {
    String urlEncoded = urlEncode(password.getBytes());
    String base64Encoded = base64Encode(urlEncoded.getBytes());
    String expected = "JTYzJTMwJTZlJTc2JTMzJTcyJTc0JTMxJTZlJTY3JTVm"
                    + "JTY2JTcyJTMwJTZkJTVmJTYyJTYxJTM1JTY1JTVmJTM2"
                    + "JTM0JTVmJTM3JTM1JTM4JTM2JTM1JTYzJTYzJTMz";
    return base64Encoded.equals(expected);
}
```

the door doesn't compare our password directly. it first url-encodes it (every byte becomes `%` plus its hex value, so `c` becomes `%63`), then base64-encodes the result, and only then compares with `expected`.

## how i cracked it
the door applies two steps, so we undo them in reverse order:

| step | the door does | we do |
|------|---------------|-------|
| 1 | url-encode | base64-decode |
| 2 | base64-encode | url-decode |

base64-decoding `expected` gives something like `%63%30%6e...`, and url-decoding that gives the password.

## solve.py
```python
import base64
from urllib.parse import unquote

expected = "JTYzJTMwJTZlJTc2JTMzJTcyJTc0JTMxJTZlJTY3JTVm" + "JTY2JTcyJTMwJTZkJTVmJTYyJTYxJTM1JTY1JTVmJTM2" + "JTM0JTVmJTM3JTM1JTM4JTM2JTM1JTYzJTYzJTMz"

step1 = base64.b64decode(expected).decode()   # bytes -> str
password = unquote(step1)                     # %63 -> 'c', ...
print(password)
```

- `import base64` gives us the base64 tools.
- `from urllib.parse import unquote` gives us the url tools. `quote` encodes (`c` -> `%63`) and `unquote` decodes (`%63` -> `c`).
- `base64.b64decode(...)` returns bytes, and `.decode()` turns those bytes into a normal string, which `unquote` needs.

## takeaways
- to undo a chain of encodings, go through the steps backwards.
