#Implementar openvpn server (AWS) client to client amb dos hosts clients de l’aula. 
#Usar de CA VeritatAbsoluta. Crear el certificat Test-server ipel servidor i 
#Test-Client1 i Test-Client2 per els clients. Recordeu que segons sigui 
#el certificat de servidor o de client ha d’incorporar unes determinades extensions. 
#El servidor que està a AWS ha de permetre el tràfic UDP al port 1194.

Copiem els certificasts de la CA a la maquina de amazon AWS que actuará com a servidor 
```
-rw-r--r--. 1 isx53320159 hisx2  1444 Mar 25 11:33 cacert.pem
-rw-r--r--. 1 isx53320159 hisx2    17 Mar 25 11:46 cacert.srl
-rw-------. 1 isx53320159 hisx2  1675 Mar 25 11:33 cakey.pem
-rw-r--r--. 1 isx53320159 hisx2   451 Mar 25 11:33 capub.pem
```

Generem la key del servidor TLS y la request
```
[fedora@ip-172-31-19-108 vpn-server]$ openssl genrsa -out test-serverkey.pem
Generating RSA private key, 2048 bit long modulus
......................................+++
.................................................+++
e is 65537 (0x010001)

[fedora@ip-172-31-19-108 vpn-server]$ openssl req -new -key serverke.pem -out serverreq.pem
Can't open serverke.pem for reading, No such file or directory
139703939655488:error:02001002:system library:fopen:No such file or directory:crypto/bio/bss_file.c:74:fopen('serverke.pem','r')
139703939655488:error:2006D080:BIO routines:BIO_new_file:no such file:crypto/bio/bss_file.c:81:
unable to load Private Key
[fedora@ip-172-31-19-108 vpn-server]$ openssl req -new -key test-serverkey.pem -out test-serverreq.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:ca
State or Province Name (full name) []:bcn
Locality Name (eg, city) [Default City]:bcn
Organization Name (eg, company) [Default Company Ltd]:edt
Organizational Unit Name (eg, section) []:www.test-server.org
Common Name (eg, your name or your server's hostname) []:www.test-server.org
Email Address []:admin@edt.org

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:jupiter
An optional company name []:jupiter
```
Per poder generar els certificats amb les extensions podem generar un fitxer d’extensions
específic per a server i un per a client. Les directives que cal usar en cada cas les podem
copiar del fitxer openssl.conf de samples.
El fitxer que crearem es diura ext.server.conf que tindra la seguent configuració
```
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
extendedKeyUsage = serverAuth
keyUsage = digitalSignature, keyEncipherment
```
Creem el certificat firmat per la nostra CA
```
[fedora@ip-172-31-19-108 vpn-server]$ openssl x509 -CAkey cakey.pem -CA cacert.pem -req -in test-serverreq.pem -CAcreateserial -days 3650 -extfile ext.server.conf -out test-servercert.pem 
Signature ok
subject=C = ca, ST = bcn, L = bcn, O = edt, OU = www.test-server.org, CN = www.test-server.org, emailAddress = admin@edt.org
Getting CA Private Key
```
Observerm el nostre certificat del servidor firmat per la nostra CA
```
[fedora@ip-172-31-19-108 vpn-server]$ openssl x509 -CAkey cakey.pem -CA cacert.pem -req -in test-serverreq.pem -CAcreateserial -days 3650 -extfile ext.server.conf -out test-servercert.pem 
Signature ok
subject=C = ca, ST = bcn, L = bcn, O = edt, OU = www.test-server.org, CN = www.test-server.org, emailAddress = admin@edt.org
Getting CA Private Key
[fedora@ip-172-31-19-108 vpn-server]$ openssl x509 -noout -text -in test-servercert.pem 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            db:bd:47:83:21:4b:a9:6e
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = ca, ST = barcelona, L = barcelona, O = edt, OU = informatica, CN = VeritatAbsoluta, emailAddress = veritat@edt.org
        Validity
            Not Before: Mar 29 08:27:22 2019 GMT
            Not After : Mar 26 08:27:22 2029 GMT
        Subject: C = ca, ST = bcn, L = bcn, O = edt, OU = www.test-server.org, CN = www.test-server.org, emailAddress = admin@edt.org
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:ce:4a:c0:d1:21:62:7d:70:7a:91:75:1c:f2:dd:
                    ef:08:e0:f2:7d:a5:12:0a:f1:93:69:69:f5:4c:63:
                    db:64:9f:e4:5b:c2:df:8a:17:74:56:a0:61:cc:8e:
                    e6:a9:90:18:11:fc:50:79:09:fc:cc:26:7d:f3:78:
                    8d:9f:5c:7b:05:c4:c5:1e:80:bd:1c:68:19:95:f8:
                    db:12:9b:1d:e9:8c:a1:99:ca:9f:3c:4a:e3:ac:be:
                    60:c2:39:bf:8c:d1:b8:a6:bf:21:05:7c:f9:55:3e:
                    33:c9:f7:d6:a5:78:3e:3a:e1:02:1e:44:2f:6a:52:
                    8d:fe:e3:3a:bf:91:9c:8a:3f:f9:27:19:0d:c8:c0:
                    44:32:2e:13:22:37:29:9a:01:33:7d:d3:b2:ce:0e:
                    bc:83:46:14:30:bf:de:20:af:15:60:38:7a:3e:c7:
                    e7:02:02:c5:0e:a6:fd:44:c9:d7:d5:7c:ae:59:4e:
                    c6:94:7e:31:8b:a9:62:6f:24:99:91:5b:d5:3a:66:
                    ba:5f:0c:5a:30:df:0c:1d:f2:82:d4:72:7b:33:fd:
                    c2:2b:d3:cb:c8:1a:e1:d5:e2:f7:c0:04:0d:0b:d2:
                    ba:da:96:fa:da:88:60:21:c8:0c:17:ff:9a:95:83:
                    c7:ee:6e:65:41:cb:29:f0:72:fb:4b:a8:33:ca:39:
                    91:dd
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Cert Type: 
                SSL Server
            Netscape Comment: 
                OpenSSL Generated Server Certificate
            X509v3 Subject Key Identifier: 
                8D:C7:92:5F:E0:85:80:A5:BE:DB:DC:7A:41:8F:83:0C:08:66:73:BD
            X509v3 Authority Key Identifier: 
                keyid:18:EB:B3:ED:AB:FA:47:B5:47:95:81:59:55:9E:E2:95:B5:5E:89:FF
                DirName:/C=ca/ST=barcelona/L=barcelona/O=edt/OU=informatica/CN=VeritatAbsoluta/emailAddress=veritat@edt.org
                serial:A4:98:42:AF:58:D1:C9:DE

            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Key Usage: 
                Digital Signature, Key Encipherment
    Signature Algorithm: sha256WithRSAEncryption
         05:97:ec:77:3d:dd:ed:72:79:58:08:ad:f7:6f:60:da:86:8f:
         a9:61:ce:21:ce:8d:4c:39:75:db:ae:36:77:2c:b9:35:55:29:
         cc:b5:dc:5c:54:61:49:01:09:08:66:2c:99:b4:0b:63:b3:8e:
         36:19:f6:43:7f:53:bd:ef:79:11:5a:37:52:d2:16:13:79:06:
         23:e3:cc:79:2c:72:26:40:ca:ea:b7:4e:9a:31:ea:b2:6c:fa:
         bd:d4:3b:00:ee:66:58:4c:6b:6f:28:75:88:08:07:78:94:86:
         8b:cf:fd:65:a1:4e:14:3e:c4:40:c1:f2:91:8b:6f:54:0e:4d:
         f1:9b:00:6a:a1:99:9e:d1:9d:2c:4b:3b:f5:66:56:50:cd:f4:
         77:a4:b0:5e:69:95:ed:f5:ab:73:3d:c9:91:cd:bc:00:49:e5:
         3a:de:33:04:92:7a:10:5e:88:c5:66:73:2b:81:b6:f3:90:3c:
         44:33:3c:2c:f7:bc:c5:fa:5b:86:fc:41:97:7a:8c:98:10:b1:
         c6:41:9f:7e:a5:10:bd:6f:89:d5:50:38:d1:05:52:a0:c4:1c:
         6f:61:22:0e:07:0a:5e:82:02:97:a3:ef:f6:6e:08:40:00:09:
         4f:e5:71:a2:03:ac:23:b7:ac:d1:60:5b:77:b9:a0:1b:27:9b:
         2b:00:6a:be
```

Ara generarem una key del client
Ens copiem els fitxers de la nostra CA a la carpeta del client
```
[isx53320159@i12 client]$ ll
total 20
-rw-r--r--. 1 isx53320159 hisx2 1444 Mar 29 09:32 cacert.pem
-rw-r--r--. 1 isx53320159 hisx2   17 Mar 29 09:32 cacert.srl
-rw-r--r--. 1 isx53320159 hisx2   83 Mar 29 09:32 ca.conf
-rw-------. 1 isx53320159 hisx2 1675 Mar 29 09:32 cakey.pem
-rw-r--r--. 1 isx53320159 hisx2  451 Mar 29 09:32 capub.pem
```
Generem la clau del client
```
[isx53320159@i12 client]$  openssl genrsa -out test-clientkey.pem
Generating RSA private key, 2048 bit long modulus
..............................................................................................+++
.......+++
e is 65537 (0x010001)
```
Generem la request del nostre client
```
[isx53320159@i12 client]$ openssl req -new -key test-clientkey.pem -out test-clientreq.pem 
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:ca
State or Province Name (full name) []:bcn
Locality Name (eg, city) [Default City]:bcn
Organization Name (eg, company) [Default Company Ltd]:inf
Organizational Unit Name (eg, section) []:edt 
Common Name (eg, your name or your server's hostname) []:www.test-client.org
Email Address []:admin@edt.org

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:jupiter
An optional company name []:jupiter
```

Creem un fitxer anomenat ext.client.conf, que tindra la configuracio. Aqeusta configutacio es troba en el fitxer openssl.conf de fedora24
```
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
```
Firmem la request amb la nostra CA y generem el certificat
```
[isx53320159@i12 client]$ openssl x509 -CAkey cakey.pem -CA cacert.pem -req -in test-clientreq.pem -CAcreateserial -days 3650 -extfile ext.client.conf -out test-clientcert.pem 
Signature ok
subject=C = ca, ST = bcn, L = bcn, O = inf, OU = edt, CN = www.test-client.org, emailAddress = admin@edt.org
Getting CA Private Key
```
Observem el certificat 
```
[isx53320159@i12 client]$ openssl x509 -noout -text -in test-clientcert.pem 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            db:bd:47:83:21:4b:a9:67
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = ca, ST = barcelona, L = barcelona, O = edt, OU = informatica, CN = VeritatAbsoluta, emailAddress = veritat@edt.org
        Validity
            Not Before: Mar 29 08:40:02 2019 GMT
            Not After : Mar 26 08:40:02 2029 GMT
        Subject: C = ca, ST = bcn, L = bcn, O = inf, OU = edt, CN = www.test-client.org, emailAddress = admin@edt.org
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b4:1d:a1:5f:c6:e7:80:5b:9c:fc:1f:fd:93:2b:
                    b5:1e:76:d5:72:72:5a:03:55:a3:32:a4:39:6b:7d:
                    43:42:5f:9e:c0:21:a6:83:95:5e:6c:66:8c:af:b8:
                    0a:95:7d:ac:6f:88:df:ba:b4:92:c8:22:80:15:d6:
                    c3:de:f9:55:32:78:0d:99:51:e3:34:e3:18:70:04:
                    2e:9b:3b:6d:81:54:75:ff:31:bd:e4:75:6e:e5:a9:
                    e3:f4:7d:60:4e:2d:e7:2e:fa:ee:65:89:5b:19:1a:
                    bb:7f:d0:2a:ad:76:ac:b9:0d:bb:26:9b:76:39:9b:
                    4d:84:ff:ff:cf:01:cf:c9:5a:6e:25:57:9e:4d:fb:
                    c0:1f:7d:94:50:85:5a:c4:18:db:7e:8a:3f:37:5c:
                    87:45:8e:b1:07:11:19:30:d1:e5:c4:86:9a:ba:5c:
                    38:c3:e1:d8:2b:7a:bf:42:bb:ae:bd:b3:e5:19:63:
                    b1:c3:57:a4:dc:7c:2b:83:f8:c8:75:a1:9e:06:39:
                    f7:91:da:39:c3:74:e3:73:f0:cc:12:25:64:aa:66:
                    15:f6:ea:74:fb:8d:26:9d:ba:c8:c2:d5:e6:8a:c2:
                    b4:99:94:af:76:0f:44:18:92:23:10:c1:7d:6a:c2:
                    3e:83:75:ba:f0:9d:29:5c:8c:2c:2b:f2:0b:24:15:
                    13:e7
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 Subject Key Identifier: 
                44:37:41:64:AB:29:C6:62:DB:8B:FD:0B:5B:A1:70:7D:82:07:E8:EC
            X509v3 Authority Key Identifier: 
                keyid:18:EB:B3:ED:AB:FA:47:B5:47:95:81:59:55:9E:E2:95:B5:5E:89:FF
                DirName:/C=ca/ST=barcelona/L=barcelona/O=edt/OU=informatica/CN=VeritatAbsoluta/emailAddress=veritat@edt.org
                serial:A4:98:42:AF:58:D1:C9:DE

    Signature Algorithm: sha256WithRSAEncryption
         b2:e4:d9:3c:07:34:dd:4c:62:07:10:0c:b4:67:a6:69:63:93:
         f0:d0:76:67:a0:88:09:5a:67:18:26:f9:e7:aa:85:09:17:5f:
         ac:0a:28:6b:fc:a8:16:a5:7a:86:bb:13:4c:26:fd:4e:2e:e7:
         6b:9b:30:bc:1f:2d:64:07:90:ca:ae:99:1d:9f:e6:dc:3e:c6:
         7b:7a:d3:f1:83:f7:18:1d:10:70:76:33:0f:24:66:95:21:87:
         dc:2f:a6:ff:13:ca:74:74:1d:c1:aa:68:1d:da:34:fa:f2:5b:
         90:0f:74:ef:6b:36:11:66:c1:92:14:ab:8c:61:d3:07:b8:ec:
         8e:1d:3d:84:85:4d:6f:ea:82:3f:80:83:19:66:ae:7b:c9:51:
         2b:86:3c:41:4b:dd:7e:fb:44:1e:17:ba:17:34:45:ae:ad:38:
         9a:30:39:b4:e8:63:e4:c6:24:98:9c:76:cf:af:fe:2a:80:04:
         58:67:54:fb:ad:1a:9d:af:b4:01:3b:c4:3d:d9:45:25:12:b8:
         31:17:8e:88:0c:7e:c2:09:05:f4:a4:c0:61:81:bb:fa:65:6a:
         ce:01:a9:07:ed:75:df:c2:9d:3a:26:b9:43:7c:e7:07:dd:02:
         57:48:e2:25:8d:f5:06:46:9f:f6:19:af:db:bd:89:63:5b:9f:
         a0:0b:2c:4d
```
Configuracio del server vpn multiclient (Tot com a root)
Per començar hem d'instalar el paquet de vpn server
```
dnf -y install openvpn
```
Com usarem TLS hem de disposar dels fitxers creats anteriorment en el servidor
```
fedora@ip-172-31-19-108 vpn-server]$ ll
total 36
-rw-rw-r--. 1 fedora fedora 1444 Mar 29 08:13 cacert.pem
-rw-rw-r--. 1 fedora fedora   17 Mar 29 08:27 cacert.srl
-rw-rw-r--. 1 fedora fedora   83 Mar 29 08:13 ca.conf
-rw-rw-r--. 1 fedora fedora 1675 Mar 29 08:13 cakey.pem
-rw-rw-r--. 1 fedora fedora  451 Mar 29 08:13 capub.pem
-rw-rw-r--. 1 fedora fedora  247 Mar 29 08:23 ext.server.conf
-rw-rw-r--. 1 fedora fedora 1814 Mar 29 08:27 test-servercert.pem
-rw-------. 1 fedora fedora 1679 Mar 29 08:17 test-serverkey.pem
-rw-rw-r--. 1 fedora fedora 1127 Mar 29 08:19 test-serverreq.pem
```
Tambe hem de tenir en compte que hem de disposar del fitxer dh2048.pem que es troba a:
```
/usr/share/doc/openvpn/sample/sample-keys/dh2048.pem
```
