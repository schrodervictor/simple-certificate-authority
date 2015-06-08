# How to setup a simple Certificate Authority

Make sure you have openssl instaled:

```
$ sudo apt-get install openssl
```

Create the folder structure:

```
$ mkdir ca
$ cd ca
$ mkdir certs crl newcerts private
$ chmod 700 private
$ touch index.txt
$ echo 1000 > serial
```

Prepare the root configuration file (see config.conf file in the root of this repo)

Generate the root CA key:

```
$ openssl genrsa -aes256 -out private/ca.key.pem 4096
$ chmod 400 private/ca.key.pem
```

Self-sign the root certificate:

```
$ openssl req -config openssl.cnf -key private/ca.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca.cert.pem
$ chmod 444 certs/ca.cert.pem
```

If you want to verify the certificate you just generated, run this command:

```
$ openssl x509 -noout -text -in certs/ca.cert.pem
```

Create the intermediate folder structure:

```
$ mkdir intermediate
$ mkdir intermediate/{certs,crl,csr,newcerts,private}
$ chmod 700 intermediate/private
$ touch intermediate/index.txt
$ echo 1000 > intermediate/serial
$ echo 1000 > intermediate/crlnumber
```

Prepare the intermediate configuration file (see intermediate/config.conf in this repo)

Generate the intermediate CA key:

```
$ openssl genrsa -aes256 -out intermediate/private/intermediate.key.pem 4096
$ chmod 400 intermediate/private/intermediate.key.pem
```

Generate the certificate sign request for the intermediate CA:

```
$ openssl req -config intermediate/openssl.cnf -new -sha256 -key intermediate/private/intermediate.key.pem -out intermediate/csr/intermediate.csr.pem
```

Sign the intermediate CSR using the root key, to generate the intermediate certificate:

```
$ openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -in intermediate/csr/intermediate.csr.pem -out intermediate/certs/intermediate.cert.pem
$ chmod 444 intermediate/certs/intermediate.cert.pem
```

To verify the certificate you just created, run these commands:

```
$ openssl x509 -noout -text -in intermediate/certs/intermediate.cert.pem
$ openssl verify -CAfile certs/ca.cert.pem intermediate/certs/intermediate.cert.pem
```

Concatenate the root and intermediate certificates in a file. This file will be the chain of authority to distribute to users:

```
$ cat intermediate/certs/intermediate.cert.pem certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem
$ chmod 444 intermediate/certs/ca-chain.cert.pem
```

Generate the key for your application:

```
$ openssl genrsa -out intermediate/private/wildcard.webapp.com.key.pem 2048
```

Generate the certificate sign request for your application:

```
$ openssl req -config intermediate/openssl.cnf -key intermediate/private/wildcard.webapp.com.key.pem -new -sha256 -out intermediate/csr/wildcard.webapp.com.csr.pem
```

Sign the CSR using the intermediate key, to generate the certificate:

```
$ openssl ca -config intermediate/openssl.cnf -extensions server_cert -days 375 -notext -md sha256 -in intermediate/csr/wildcard.webapp.com.csr.pem -out intermediate/certs/wildcard.webapp.com.cert.pem
```

To test the new certificate, run these commands:

```
$ openssl x509 -noout -text -in intermediate/certs/wildcard.webapp.com.cert.pem
$ openssl verify -CAfile intermediate/certs/ca-chain.cert.pem intermediate/certs/wildcard.webapp.com.cert.pem
```
