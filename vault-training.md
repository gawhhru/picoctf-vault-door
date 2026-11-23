# vault-door-training

**difficulty:** easy

## the class
every vault door starts with the same `main`, so let's read this one first.

```java
public static void main(String args[]) {
    VaultDoor1 vaultDoor = new VaultDoor1();
    Scanner scanner = new Scanner(System.in);
    System.out.print("Enter vault password: ");
    String userInput = scanner.next();
    String input = userInput.substring("academy{".length(), userInput.length()-1);
    if (vaultDoor.checkPassword(input)) {
        System.out.println("Access granted.");
    } else {
        System.out.println("Access denied!");
    }
}
```

what it does: it takes what we type, cuts off the first 8 characters (`academy{`) and the last one (`}`), and passes the middle part to `checkPassword`.

so the door only accepts `academy{password}`. the string we find is the password.

this applies to every door in the series, so i won't repeat it later.

## the lock
now, `checkPassword`:

```java
public boolean checkPassword(String password) {
    return password.equals("w4rm1ng_Up_w1tH_jAv4_8352733cecd");
}
```

`equals()` checks whether two strings have exactly the same characters, and answers `true` or `false`. here it compares our input with one fixed string, so the password is simply the string written in the source.

**flag:** `academy{w4rm1ng_Up_w1tH_jAv4_8352733cecd}`
