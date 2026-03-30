# JWT - Introduction

## Statement :

To validate the challenge, connect as admin.

## Analysis

![Main page](../../assets/rootme/Web-Server/jwt/main-page.png)

We get a simple login form with `username` and `password` fields. There is also an option to connect as a guest, let's click on it.

![Guest page](../../assets/rootme/Web-Server/jwt/guest-page.png)

Since the challenge is about JWT (JSON Web Token), logging in as a guest probably generates a token. Let's intercept the request with Burp.

![Burp Token](../../assets/rootme/Web-Server/jwt/burp-token.png)

We find the guest token in the cookie header:

```
Cookie: jwt=eyJhbGciOiJIUzI1NiJ9.eyJlc2VybmFtZSI6Imd1ZXN0IiwiYWRtaW4iOmZhbHNl.OnuZnYMdetcg7AWGVeWURn8CFSfasAQej4V9Ml3nsk
```

Now we need to decode it. Using [jwt.io](https://jwt.io), we can inspect its structure:

![Decoded Token](../../assets/rootme/Web-Server/jwt/decoded-token.png)

## Exploit

The token consists of three parts: a header, a payload, and a signature. The payload contains the role we authenticated as, which is currently `guest`. If we change it to `admin`, we might gain access. The signature would normally prevent tampering, but we can bypass it by setting the `alg` field in the header to `none`, telling the server not to verify the signature at all.

This gives us a forged token:

![Encoded Token](../../assets/rootme/Web-Server/jwt/encoded-token.png)

Replacing the original token in the Burp request with our forged token validates the challenge.

![Admin Page](../../assets/rootme/Web-Server/jwt/admin-page.png)
