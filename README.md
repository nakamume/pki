## pki
This repo contains my private ca key and cert along with a dummy client and server certs.

I'll document all the steps and commands used during the process here:
1. create directories
    ```bash
    mkdir {ca,server,client}
    ```
1. create ca key (This should be kept super secret. And should be hard to guess - using 4096 bits)
    ```bash
    openssl genrsa -out ca/ca.key 4096
    ```
1. generate ca cert using ca.key (ca cert should typically have long validity `-days 1000`)
    ```bash
    openssl req -new -x509 -days 1000 -key ca/ca.key -sha256 -out ca/ca.pem

    -----
    Country Name (2 letter code) []:IN
    State or Province Name (full name) []:
    Locality Name (eg, city) []:
    Organization Name (eg, company) []:nakam.org
    Organizational Unit Name (eg, section) []:
    Common Name (eg, fully qualified host name) []:www.nakam.org
    Email Address []:
    ```
1. Now, that we have our CA, we can create client keys and then create CSR(certificate signing requests) for the CA and get client certs
1. generate a key for a server
    ```bash
    openssl genrsa -out server/server.key 1024
    ```
1. Use the `csr.conf.tpl` template file to create a csr config for server
1. Generate csr based on config file
    ```bash
    openssl req -new -key server/server.key -out server/server.csr -config server/csr.conf
    ```
1. Generate server cert using ca.key, ca.pem and server.csr
    ```bash
    openssl x509 -req -in server/server.csr -CA ca/ca.pem -CAkey ca/ca.key \
    -CAcreateserial -out server/server.pem -days 365 \
    -extensions v3_ext -extfile server/csr.conf
    ```
1. (optional) create a client key and cert (maybe for mTLS mutual TLS) signed by same ca
    ```bash
    openssl genrsa -out client/client.key 1024 # generate key
    # Use the `csr.conf.tpl` template file to create a csr config for client and put it into client folder
    openssl req -new -key client/client.key -out client/client.csr -config client/csr.conf # generate csr
    openssl x509 -req -in client/client.csr -CA ca/ca.pem -CAkey ca/ca.key \
    -CAcreateserial -out client/client.pem -days 365 \
    -extensions v3_ext -extfile client/csr.conf # create cert
    ```
1. Once you get your certs from CA, you can delete the csr
1. (optional) View csr
    ```bash
    openssl req  -noout -text -in server/server.csr


    Certificate Request:
        Data:
            Version: 0 (0x0)
            Subject: C=IN, CN=192.168.64.5
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    Public-Key: (1024 bit)
                    Modulus:
                        00:c4:07:ab:cd:de:1a:a1:1e:cd:f2:31:4d:bf:e7:
                        8d:43:e8:97:7d:b9:63:39:f2:55:1a:6b:7a:0c:98:
                        34:b7:28:fe:12:ee:83:a2:d9:0f:37:53:35:9a:06:
                        a1:2e:d1:28:6c:75:bf:b3:fd:83:c6:1e:45:6a:73:
                        f7:94:35:e5:0e:38:ae:7c:92:44:44:6e:d4:32:3b:
                        13:0a:42:17:07:c0:56:64:4a:d4:5d:de:cc:3d:47:
                        5a:d0:43:93:dd:07:1f:ee:39:4c:59:7e:ad:69:eb:
                        86:de:38:54:9e:2c:9a:a2:e2:1e:03:97:a2:85:a2:
                        bf:e6:14:bb:4a:89:18:4a:ed
                    Exponent: 65537 (0x10001)
            Attributes:
            Requested Extensions:
                X509v3 Subject Alternative Name:
                    DNS:www.nakam.org, DNS:nakam.org, DNS:nakam.com, IP Address:192.168.64.5, IP Address:172.17.0.1
        Signature Algorithm: sha256WithRSAEncryption
            b6:5c:e4:88:da:65:3d:bf:a3:c5:5e:b0:a9:9b:96:92:81:ba:
            3a:50:63:32:f4:0f:17:79:5a:a2:ff:24:7c:e8:1c:9f:4e:be:
            f8:ca:08:33:94:40:96:1b:1a:e8:e6:0c:45:08:3d:a4:74:8e:
            14:07:20:22:71:4c:62:c4:4c:0b:b9:75:46:18:4b:11:bb:50:
            d7:43:67:58:11:75:10:e5:fb:d3:ae:4a:b3:d8:f6:a0:e3:ee:
            67:48:a9:ce:2a:15:95:fe:b7:ff:06:05:5f:29:cf:77:63:5e:
            90:25:0e:60:f0:16:fb:3c:51:c5:36:84:35:12:26:53:61:45:
            8c:f9
    ```
1. (optional) View server cert
    ```bash
    openssl x509  -noout -text -in server/server.pem


    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number: 14254672693457661269 (0xc5d2c54a4b012d55)
        Signature Algorithm: sha1WithRSAEncryption
            Issuer: C=IN, O=nakam.org, CN=www.nakam.org
            Validity
                Not Before: Feb  2 03:00:06 2022 GMT
                Not After : Feb  2 03:00:06 2023 GMT
            Subject: C=IN, CN=192.168.64.5
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    Public-Key: (1024 bit)
                    Modulus:
                        00:c4:07:ab:cd:de:1a:a1:1e:cd:f2:31:4d:bf:e7:
                        8d:43:e8:97:7d:b9:63:39:f2:55:1a:6b:7a:0c:98:
                        34:b7:28:fe:12:ee:83:a2:d9:0f:37:53:35:9a:06:
                        a1:2e:d1:28:6c:75:bf:b3:fd:83:c6:1e:45:6a:73:
                        f7:94:35:e5:0e:38:ae:7c:92:44:44:6e:d4:32:3b:
                        13:0a:42:17:07:c0:56:64:4a:d4:5d:de:cc:3d:47:
                        5a:d0:43:93:dd:07:1f:ee:39:4c:59:7e:ad:69:eb:
                        86:de:38:54:9e:2c:9a:a2:e2:1e:03:97:a2:85:a2:
                        bf:e6:14:bb:4a:89:18:4a:ed
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Authority Key Identifier:
                    DirName:/C=IN/O=nakam.org/CN=www.nakam.org
                    serial:AA:A6:C4:46:4E:1E:3C:7B

                X509v3 Basic Constraints:
                    CA:FALSE
                X509v3 Key Usage:
                    Key Encipherment, Data Encipherment
                X509v3 Extended Key Usage:
                    TLS Web Server Authentication
                X509v3 Subject Alternative Name:
                    DNS:www.nakam.org, DNS:nakam.org, DNS:nakam.com, IP Address:192.168.64.5, IP Address:172.17.0.1
        Signature Algorithm: sha1WithRSAEncryption
            89:e6:af:3c:e8:ab:c0:b0:bc:7b:6c:5c:6a:7c:6b:6c:2c:96:
            2d:a9:df:00:3c:97:50:54:4b:b7:da:7a:4e:a5:2c:e0:cb:44:
            6f:f1:3b:7e:2e:7a:61:db:7d:54:9d:0d:4f:16:ea:e2:22:67:
            7e:67:04:d7:9e:7d:f6:b6:75:2d:09:4a:98:1d:57:e1:35:78:
            f6:5e:82:80:9b:57:ba:18:2d:a7:b4:15:db:a7:15:73:75:ed:
            7d:78:9d:a5:5e:2f:5d:29:ee:87:b8:aa:62:30:bf:1c:c5:56:
            54:e7:b5:91:39:99:b1:f0:43:53:17:c3:46:19:85:4c:be:ff:
            65:4d:ec:17:74:03:10:fd:2d:61:1c:bd:b7:35:92:66:bb:6b:
            1e:1a:60:83:96:7c:f7:ac:fa:00:4c:81:28:56:99:28:a1:e8:
            f6:fb:ef:57:4b:f6:57:06:59:36:3d:85:3a:87:3b:8f:a3:cd:
            53:30:41:ef:97:6b:69:00:09:56:f1:e8:ea:b0:31:ca:db:13:
            0c:92:15:68:d8:fa:d9:f5:69:81:44:ed:ea:fd:21:6f:a2:ae:
            f9:91:ba:81:84:26:8b:ce:3e:bb:1a:c5:eb:37:ad:8c:b9:a6:
            ff:5c:ed:2f:c1:73:6c:41:31:aa:49:04:c7:ba:df:c4:cf:62:
            7e:57:21:ef:a9:6a:de:c1:e8:1b:25:48:8d:0f:c2:51:a9:38:
            3c:6d:e9:e8:14:bc:5e:20:a8:a7:ef:38:af:43:12:d2:41:19:
            97:45:ff:59:86:8e:d8:ad:0f:7e:28:dd:cf:fe:06:68:7e:c4:
            e6:89:97:aa:79:bd:be:82:5a:c1:a3:6a:1d:52:c9:3e:cb:d4:
            98:6f:a8:2e:3c:3a:54:b8:94:aa:bb:12:c7:7f:94:7a:da:a3:
            be:60:d5:78:0c:91:3e:33:93:00:c2:9b:80:d8:b9:a4:ec:e3:
            89:b0:74:8b:c3:70:be:25:f3:81:1a:85:54:a2:f5:7b:bf:e6:
            06:64:cb:09:da:38:d6:b7:89:d9:f2:5e:b2:70:4f:7b:a5:9c:
            f7:c2:66:91:26:ab:18:49:56:6a:23:b0:7a:e2:4e:cc:a4:81:
            77:4b:9a:12:14:d2:5e:b6:dd:ff:b5:57:3b:ba:d1:bf:50:ee:
            5d:7d:7b:3a:01:e6:4c:44:2b:da:29:00:48:74:72:5f:58:c7:
            35:24:5d:cb:df:c3:df:18:42:8a:b6:8c:88:dd:3e:51:8a:cd:
            cb:ad:af:d6:11:2c:e8:be:bf:ac:c7:a5:2b:92:a6:6e:ee:03:
            9b:d9:f0:e1:2b:d3:ce:28:7c:59:a6:49:03:92:cd:db:63:0a:
            51:e2:03:9b:b8:9b:53:18
    ```
