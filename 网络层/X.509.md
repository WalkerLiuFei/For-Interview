# X.509

## glossary

PKI : public key infrastructure.

CRL: Certificate revocation list : a list of digit certificate, those certificate **has been revoked** by the CA before it schedule expiring data and should not longer be trusted.	



## certificate chain

A **certificate chain** (see the equivalent concept of "certification path" defined by [RFC 5280](https://tools.ietf.org/html/rfc5280))[[12\]](https://en.wikipedia.org/wiki/X.509#cite_note-RFC_5280_Certification_Path_Validation-12) is a list of certificates (usually starting with an end-entity certificate) followed by one or more [CA](https://en.wikipedia.org/wiki/Certificate_authority) certificates (usually the last one being a self-signed certificate), with the following properties:

1. The Issuer of each certificate (except the last one) matches the Subject of the next certificate in the list.
2. **Each certificate (except the last one) is signed by the secret key corresponding to the next certificate in the chain (i.e. the signature of one certificate can be verified using the public key contained in the following certificate).**
3. **The last certificate in the list is a [trust anchor](https://en.wikipedia.org/wiki/Trust_anchor): a certificate that you trust because it was delivered to you by some trustworthy procedure.、**

Certificate chain is used to verify PK contained in the first certificate and data of the certificate contained in it belongs to its object.

In order to achevie this, the signature contained in the certificated is verified by the following certificate’s public key until trust anchor(root certificate)

## Sample X.509 certificates

```yaml
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            10:e6:fc:62:b7:41:8a:d5:00:5e:45:b6
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=BE, O=GlobalSign nv-sa, CN=GlobalSign Organization Validation CA - SHA256 - G2
        Validity
            Not Before: Nov 21 08:00:00 2016 GMT
            Not After : Nov 22 07:59:59 2017 GMT
        Subject: C=US, ST=California, L=San Francisco, O=Wikimedia Foundation, Inc., CN=*.wikipedia.org
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
            pub: 
                    00:c9:22:69:31:8a:d6:6c:ea:da:c3:7f:2c:ac:a5:
                    af:c0:02:ea:81:cb:65:b9:fd:0c:6d:46:5b:c9:1e:
                    9d:3b:ef
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Agreement
            Authority Information Access: 
                CA Issuers - URI:http://secure.globalsign.com/cacert/gsorganizationvalsha2g2r1.crt
                OCSP - URI:http://ocsp2.globalsign.com/gsorganizationvalsha2g2
            X509v3 Certificate Policies: 
                Policy: 1.3.6.1.4.1.4146.1.20
                  CPS: https://www.globalsign.com/repository/
                Policy: 2.23.140.1.2.2
            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 CRL Distribution Points: 
                Full Name:
                  URI:http://crl.globalsign.com/gs/gsorganizationvalsha2g2.crl
            X509v3 Subject Alternative Name: 
                DNS:*.wikipedia.org, DNS:*.m.mediawiki.org, DNS:*.m.wikibooks.org, DNS:*.m.wikidata.org, DNS:*.m.wikimedia.org, DNS:*.m.wikimediafoundation.org, DNS:*.m.wikinews.org, DNS:*.m.wikipedia.org, DNS:*.m.wikiquote.org, DNS:*.m.wikisource.org, DNS:*.m.wikiversity.org, DNS:*.m.wikivoyage.org, DNS:*.m.wiktionary.org, DNS:*.mediawiki.org, DNS:*.planet.wikimedia.org, DNS:*.wikibooks.org, DNS:*.wikidata.org, DNS:*.wikimedia.org, DNS:*.wikimediafoundation.org, DNS:*.wikinews.org, DNS:*.wikiquote.org, DNS:*.wikisource.org, DNS:*.wikiversity.org, DNS:*.wikivoyage.org, DNS:*.wiktionary.org, DNS:*.wmfusercontent.org, DNS:*.zero.wikipedia.org, DNS:mediawiki.org, DNS:w.wiki, DNS:wikibooks.org, DNS:wikidata.org, DNS:wikimedia.org, DNS:wikimediafoundation.org, DNS:wikinews.org, DNS:wikiquote.org, DNS:wikisource.org, DNS:wikiversity.org, DNS:wikivoyage.org, DNS:wiktionary.org, DNS:wmfusercontent.org, DNS:wikipedia.org
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Subject Key Identifier: 
                28:2A:26:2A:57:8B:3B:CE:B4:D6:AB:54:EF:D7:38:21:2C:49:5C:36
            X509v3 Authority Key Identifier: 
                keyid:96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C

    Signature Algorithm: sha256WithRSAEncryption
         8b:c3:ed:d1:9d:39:6f:af:40:72:bd:1e:18:5e:30:54:23:35:
         ...
```

To verify this entity certificate, needs an intermidate certificate that matchs its issuer and Authourity key identifier.

In this case we need find certificate which id is ’96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C‘ and use its id to verify the signature  ‘8b:c3:ed:d1:9d:39:6f:af:40:72:bd:1e:18:5e:30:54:23:35:.......’

## PKI standards for X.509



## How to generate a certificate chain 



# Create Root CA (Done once)

#### Create Root Key

**Attention:** this is the key used to sign the certificate requests, anyone holding this can sign certificates on your behalf. So keep it in a safe place!

```
openssl genrsa -des3 -out rootCA.key 4096
```

If you want a non password protected key just remove the `-des3` option

##### Create and self sign the Root Certificate

```
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.crt
```

Here we used our root key to create the root certificate that needs to be distributed in all the computers that have to trust us.

##### Create a certificate (Done for each server)

This procedure needs to be followed for each server/appliance that needs a trusted certificate from our CA

##### Create the certificate key

```
openssl genrsa -out mydomain.com.key 2048
```

##### Create the signing (csr)

The certificate signing request is where you specify the details for the certificate you want to generate. This request will be processed by the owner of the Root key (you in this case since you create it earlier) to generate the certificate.

**Important:** Please mind that while creating the signign request is important to specify the `Common Name` providing the IP address or domain name for the service, otherwise the certificate cannot be verified.

I will describe here two ways to gener

##### Method A (Interactive)

If you generate the csr in this way, openssl will ask you questions about the certificate to generate like the organization details and the `Common Name` (CN) that is the web address you are creating the certificate for, e.g `mydomain.com`.

```
openssl req -new -key mydomain.com.key -out mydomain.com.csr
```

##### Verify the csr's content

```
openssl req -in mydomain.com.csr -noout -text
```

##### Generate the certificate using the `mydomain` csr and key along with the CA Root key

```
openssl x509 -req -in mydomain.com.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out mydomain.com.crt -days 500 -sha256
```

##### Verify the certificate's content

```
openssl x509 -in mydomain.com.crt -text -noout
```

##### parse ans1 format certificate and keys 

```bash
openssl asn1parse -i -in your_request
```

