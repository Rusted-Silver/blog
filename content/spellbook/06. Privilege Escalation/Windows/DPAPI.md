# Manual

The stored masterkey is stored at`%userprofile%\AppData\Roaming\Micosoft\Protect\`.

In this directory, there will be another directory starts with `S-1-5-21` which is a `SID` of a user. Copy the `SID`, we need it to generate a decryption key.

Go in the directory and grab the blob. The file name should look like a **random GUID**

Then we can generate a decryption key like this. File is the file we grabbed, `SID` is the directory name, or `SID` of the user, password is the password of that user

```sh
impacket-dpapi masterkey -file '11111111-1111-1111-1111-111111111111' -sid 'S-1-5-21-1111111111-1111111111-1111111111-1111' -password 'password'
```

The DPAPI encrypted credentials is stored at

- `%userprofile%\AppData\Local\Micosoft\Credentials\`
- `%userprofile%\AppData\Roaming\Micosoft\Credentials\`

We go in, grab the files. The file name should look like **32 characters hex**

And we will decrypt with this. `-file` is the encrypted credentials file that we grabbed, and `-key` is the key we generated with the above step

```sh
impacket-dpapi credential -file BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB -key '0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa'
```
