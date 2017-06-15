Example of a Grails 3 application running in port 8443 with a self signed certificate

The app will start in **https://localhsot:8443**

## Certificates 

Inside the certificates folder you will find the certificate used in this applciation. Please, don't use in your application. These are just an example. 

The key store password for the keystore.p12 certificate is `exportpassword`. 

The PEM pass phrase is `pempassphrase`

For this example, we generate a self-signed SSL certificate. In a real production environment, you may consider using a certificate issued by a Certificate Authority (CA). [Risks of self signing a certificate for SSL](https://security.stackexchange.com/questions/8110/what-are-the-risks-of-self-signing-a-certificate-for-ssl).

## Steps to generate the certificate

Generate our self-signed SSL certificate: 

````
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
````

We use several flags:

- **x509** this option outputs a self signed certificate instead of a certificate request.
- **newkey** this option creates a new certificate request and a new private key. rsa:nbits, where nbits is the number of bits, generates an RSA key nbits in size.
- **keyout** this gives the filename to write the newly created private key to. 
- **out** This specifies the output filename to write
- **days** when the -x509 option is being used this specifies the number of days to certify the certificate for. The default is 30 days

You can read more about by running `man req`

Grails (and Spring Boot) doesn’t support the PEM format. Instead, we need to use the PKCS12 format for our keys. 
Fortunately, there is a single `openssl` command to make the conversion:

````
openssl pkcs12 -export -in cert.pem -inkey key.pem -out keystore.p12 -name tomcat -caname root
````

We use several flags

- **export** This option specifies that a PKCS#12 file will be created rather than parsed.
- **name**  This specifies the "friendly name" for the certificate and private key. When we modify application.yml, we will use this name as the `keyAlias`
- **in** This specifies filename of the PKCS#12 file to be parsed. 
- **inkey** file to read private key from.
- **caname** This specifies the "friendly name" for other certificates

You can read more about pkcs121 by running `man pks12`


Update `server/grails-app/conf/application.yml` with the following lines at the end:

```
---
server:
    port: 8443
    ssl:
        enabled: true
        key-store: /Users/sdelamo/git/grails-samples/grails-ssl/certificates/keystore.p12
        key-store-password: exportpassword
        keyStoreType: PKCS12
        keyAlias: tomcat
```

Note: the key alias matches the 
