# My OpenSSL Documentation
This is my personal list of OpenSSL commands that I have used thoughout the years. 

- [OpenSSL commands](#openssl-commands)
  - [Conversions](#conversions)
  - [Checks](#checks)
  - [Generation](#generation)
  - [Misc](#misc)
  - [Timestamping](#timestamping)
  - [Connections](#connections)
  - [Encryption](#encryption)
    - [Symmetic encryption](#symmetic-encryption)
    - [Asymmetric encryption](#asymmetric-encryption)
  - [Verification](#verification)
- [Formats](#formats)


# OpenSSL commands 

## Conversions

**Convert a DER file (.crt .cer .der) to PEM**

    openssl x509 -inform der -in <cert.der> -out <cert.pem>

**Convert a PEM file to DER**

    openssl x509 -outform der -in <cert.pem> -out <cert.der>

**Convert a PKCS#12 file (.pfx .p12) containing a private key and certificates to PEM**

    openssl pkcs12 -in <cert.pfx> -out <cert.pem> -nodes


**Convert a PKCS#12 file (.pfx .p12) to certificate and keyfile**

    openssl pkcs12 -in <cert.pfx> -nocerts -nodes -out <privatekey.pem>
    openssl pkcs12 -in <cert.pfx> -clcerts -nokeys -out <cert.pem>


**Convert a PEM certificate file and a private key to PKCS#12 (.pfx .p12)** 

    openssl pkcs12 -export -out <cert.pfx> -inkey <privatekey.pem> -in <cert.pem> -certfile <CAcerts.pem>


**Convert a PEM certificate file and a private key to PKCS#12 compatible with older version of Windows (.pfx .p12) 
    openssl pkcs12 -keypbe PBE-SHA1-3DES -certpbe PBE-SHA1-3DES -export -out <cert.pfx> -inkey <privatekey.pem> -in <CAcerts.pem>

(NB: older versions of Windows do not support AES256 for the password, this command exports the .pfx with SHA1-3DES instead)


**Convert PKCS8 to PKCS1** 
    
    openssl rsa -in <privatekey-PKCS8.pem> -out <privatekey-PKCS1.pem>

(NB: PKCS8 is privatekey with OID key identifier, PKCS1 is without)


## Checks

**Check a Certificate Signing Request (CSR)**

    openssl req -text -noout -verify -in <CSR.csr>


**Check a private key** 

    openssl rsa -in <privatekey.pem> -check


**Check a certificate**

    openssl x509 -in <cert.pem> -text -noout


**Check a PKCS#12 file (.pfx or .p12)**

    openssl pkcs12 -info -in <cert.pfx>


**Check an MD5 hash of the public key to ensure that it matches with what is in a CSR or private key**

    openssl x509 -noout -modulus -in <cert.pem> | openssl md5
    openssl rsa -noout -modulus -in <privatekey.pem> | openssl md5
    openssl req -noout -modulus -in <CSR.csr> | openssl md5

(If value is the same it matches)


**Check if the encoding of special characters is correct**

    openssl asn1parse -in <CSR.csr>

(NB: include -utf8 when generating a csr to make sure that the encoding is correct if using special characters)


## Generation

**Generate new RSA private key**

    openssl genrsa -out <privatekey.pem> 2048


**Generate a new RSA private key and Certificate Signing Request**

    openssl req -out <CSR.csr> -new -newkey rsa:2048 -nodes -keyout <privatekey.pem>


**Generate a self-signed certificate**

    openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout <privatekey.pem> -out <cert.pem>


**Generate a certificate signing request (CSR) for an existing private key**

    openssl req -out <CSR.csr> -key <privatekey.pem> -new


**Generate a certificate signing request based on an existing certificate**

    openssl x509 -x509toreq -in <cert.pem> -out <CSR.csr> -signkey <privatekey.pem>


## Misc

**Remove a passphrase from a private key**

    openssl rsa -in <privatekey_withpw.pem> -out <privatekey_withoutpw.pem>

**Extract public key from CSR**

    openssl req -in <CSR.csr> -noout -pubkey -out <publickey.pem>

**Very certificate chain. Root is set to untrusted because it is self-signed**

    openssl verify -verboose -CAfile <root_cert.pem> -untrusted <intermediate_cert.pem> <cert.pem>

(NB: openssl verify stops at the first self-signed certificate it encounters, remember if used for private trust)

## Timestamping

**Generate a timestamp request for a file**

    openssl ts -query -data <file.txt> -cert -sha256 -no_nonce -out <tsrequest.tsq>

**Send the timestamp request to be signed by a timestamp service**

    curl -k -H "Content-Type: application/timestamp-query" -o <tsresponse.tsr> -X POST -d @<tsrequest.tsq> <http://timestamp.globalsign.com>

**Verify the signed timestamp request**

    openssl ts -reply -in <tsresponse.tsr> -text

## Connections

**Check an SSL connection. All the certificates (including Intermediates) should be displayed**

    openssl s_client -connect <www.trustzone.com>:443


**Check if website supports specific TLS protocols**

    openssl s_client -connect <www.trustzone.com>:443 -tls1
    openssl s_client -connect <www.trustzone.com>:443 -tls1_2
    openssl s_client -connect <www.trustzone.com>:443 -tls1_3


**Check client authentification on website**

    openssl s_client -cert <client_cert.pem> -key <client_privatekey.pem> -connect <www.trustzone.com>:443 -CAfile <intermediate_cert.pem>


## Encryption

### Symmetic encryption

**For symmetic encryption, you can use the following:**

To encrypt:

    openssl aes-256-cbc -salt -a -e -in <plaintext.txt> -out <encrypted.txt>

To decrypt:

    openssl aes-256-cbc -salt -a -d -in <encrypted.txt> -out <plaintext.txt>

### Asymmetric encryption

For Asymmetric encryption you must first generate your private key and extract the public key.

    openssl genrsa -aes256 -out <privatekey.pem> 4096
    openssl rsa -in private.key -pubout -out <publickey.pem>

(The 4096 value refers to the keysize this can changed if needed to fx. 8192 or 2048)

**To encrypt:**

    openssl rsautl -encrypt -pubin -inkey <publickey.pem> -in <plaintext.txt> -out <encrypted.txt>

**To decrypt: (Does not work with GlobalSign Atlas .enc files)**

    openssl rsautl -decrypt -inkey <privatekey.pem> -in <encrypted.txt> -out <plaintext.txt>

**To decrypt GlobalSign Atlas .enc files:**

    openssl pkeyutl -inkey <privatekey.pem> -in <encrypted.txt> -out <plaintext.txt> -decrypt -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256


## Verification

**Creating a signed digest of a file:**

    openssl dgst -sha512 -sign <privatekey.pem> -out digest.sha512 <file.txt>

**Verify a signed digest:**

    openssl dgst -sha512 -verify <publickey.pem> -signature digest.sha512 <file.txt>


# Formats

**DER**
Distinguished Encoding Rules files are binary encoded x.509 certificates or private keys. They do not contain the usual "-----BEGIN CERTIFICATE-----" and cannot be read by a normal text editor  
DER certificates is commonly used with Java-based platforms.

Common extensions: `.der` `.cer`

**PEM**
Privacy Enchanced Mail is the most common format for certificates and private keys. Data is encoded as base64 ASCII and usually contains a variation of "-----BEGIN CERTIFICATE-----".
Can be read by a text editor and is easy to copy-pasted between systems. Ensure that the header and footer is correctly formatted when copy-pasting. This format is default on most systems and especially on Linux-based systems.

Common extensions: `.pem` `.cer` `.crt` `.key`

**PKCS1**
Fairly minmal format for RSA public and private keys (not certificates!). Can be recognized by "RSA" being included in the header/footer ie. "-----BEGIN RSA KEY----".
Older version of OpenSSL used this a default but now uses X.509 formatting instead for public keys and PKCS#8 for private keys. 

Common extensions: `.key`

**PKCS#7**
Know as Cryptographic Message Syntax (CMS). Certificate container which can hold certificate files in DER or PEM encoding. Can be recognized by having "PKCS7" in the header/footer.
This format is usually used to transfer certificates and the corresponding intermediate in one file. Does not include private keys. 
Also used for certificate revocations lists (CRL's) 

Common extensions: `.p7b`

**PKCS#8**
Most common format for private keys on Linux-based systems. Can be recognized by having the following header/footer "-----BEGIN PRIVATE KEY----". Can also be encrypted with a password, if 
password is set the header/footer will include "ENCRYPTED". Will sometimes include addtional information if exported from a pfx generated by Windows. 

Common extensions: `.key`

**PKCS#10**
Format for Certificate Signing Requests (CSR's). Contains a public key and identifying information about the key holder. The CSR is sent to a Certificate Authority (CA) to obtain a digital certificate.
A CSR includes a hash of the content signed by the corresponding private key. If any information is changed the CA will reject it. CA's can choose to ignore fiels and use other during certificate issuance
fx. correcting identifying information or adding domain names as SAN's  

Common extensions: `.csr`


**PKCS#11**
Vendor neutral API used to interface with Hardware Security Modules (HSM) and Smart Cards. Can be used to do common cryptograhic operations such as generation, modifying or deleting keys and certificates.  

Common extensions: N/A

**PKCS#12**
More commonly knows as PFX format. A container that can hold multiple certificates and keys. Commonly used to transport a certificate chain and corresponding private key in a single file. 
Requires a password to be set when created.  

Common extensions: `.pfx` `.p12`


Created by Christian Brandenburg 2023
