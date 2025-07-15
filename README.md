Directory Files:
================
drwxr-xr-x 2 sli sli 4096 Jul 14 16:59 ca
drwxr-xr-x 2 sli sli 4096 Jul 14 16:59 client
-rwxr-xr-x 1 sli sli 1293 Jul 14 17:00 Makefile
drwxr-xr-x 2 sli sli 4096 Jul 14 16:59 router
 
$ more ca/ca.cnf
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_ca
prompt = no

[req_distinguished_name]
countryName = US
stateOrProvinceName = California
localityName = Sunnyvale
organizationalUnitName = Engineering
commonName = Self Signed Root CA

[v3_ca]
basicConstraints = critical,CA:TRUE
keyUsage = critical, keyCertSign, cRLSign
subjectKeyIdentifier = hash

$ more client/client.cnf 
[req]
distinguished_name = req_distinguished_name
prompt             = no
req_extensions     = v3_req

[req_distinguished_name]
countryName            = US
stateOrProvinceName    = California
localityName           = Sunnyvale
organizationalUnitName = Engineering
commonName             = client

[v3_req]
keyUsage = critical, digitalSignature
extendedKeyUsage = clientAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = client

$ more router/router.cnf 
[req]
distinguished_name = req_distinguished_name
req_extensions     = req_ext
prompt             = no

[req_distinguished_name]
countryName            = US
stateOrProvinceName    = California
localityName           = Sunnyvale
organizationalUnitName = Engineering
commonName             = router

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = router
DNS.2 = router.ultralab.juniper.net
DNS.3 = *.juniper.net

$ more Makefile 
all: ca-cert client-cert router-cert

ca-cert:
        openssl genrsa -out ca/ca.key 4096
        openssl req -x509 -new -nodes -key ca/ca.key \
                -days 3650 -out ca/ca.crt \
                -config ca/ca.cnf --extensions v3_ca

client-cert:
        openssl genrsa -out client/client.key 4096
        openssl req -new -key client/client.key \
                -out client/client.csr -config client/client.cnf
        openssl x509 -req -days 365 -in client/client.csr \
                -CA ca/ca.crt -CAkey ca/ca.key \
                -CAcreateserial -out client/client.crt -extensions v3_req -extfile client/client.cnf

router-cert:
        openssl genrsa -out router/router.key 4096
        openssl req -new -key router/router.key \
                -out router/router.csr -config router/router.cnf
        openssl x509 -req -days 365 -in router/router.csr \
                -CA ca/ca.crt -CAkey ca/ca.key \
                -CAcreateserial -out router/router.crt \
                -extensions req_ext -extfile router/router.cnf
        cat router/router.crt router/router.key > router/router.pem

read-ca-crt:
        openssl x509 -text -noout -in ca/ca.crt

read-client-csr:
        openssl req -noout -text -in client/client.csr

read-client-crt:
        openssl x509 -text -noout -in client/client.crt

read-router-csr:
        openssl req -noout -text -in router/router.csr

read-router-crt:
        openssl x509 -text -noout -in router/router.crt

clean:
        rm */*.crt */*.key */*.csr */*.srl */*.pem
$ 

$ make
openssl genrsa -out ca/ca.key 4096
openssl req -x509 -new -nodes -key ca/ca.key \
        -days 3650 -out ca/ca.crt \
        -config ca/ca.cnf --extensions v3_ca
openssl genrsa -out client/client.key 4096
openssl req -new -key client/client.key \
        -out client/client.csr -config client/client.cnf
openssl x509 -req -days 365 -in client/client.csr \
        -CA ca/ca.crt -CAkey ca/ca.key \
        -CAcreateserial -out client/client.crt -extensions v3_req -extfile client/client.cnf
Certificate request self-signature ok
subject=C = US, ST = California, L = Sunnyvale, OU = Engineering, CN = client
openssl genrsa -out router/router.key 4096
openssl req -new -key router/router.key \
        -out router/router.csr -config router/router.cnf
openssl x509 -req -days 365 -in router/router.csr \
        -CA ca/ca.crt -CAkey ca/ca.key \
        -CAcreateserial -out router/router.crt \
        -extensions req_ext -extfile router/router.cnf
Certificate request self-signature ok
subject=C = US, ST = California, L = Sunnyvale, OU = Engineering, CN = router
cat router/router.crt router/router.key > router/router.pem
$

$ ls -l ca/
total 16
-rw-rw-r-- 1 sli sli  372 Jul 14 16:59 ca.cnf
-rw-rw-r-- 1 sli sli 2017 Jul 14 17:04 ca.crt  
-rw------- 1 sli sli 3272 Jul 14 17:00 ca.key
-rw-rw-r-- 1 sli sli   41 Jul 14 17:00 ca.srl

$ ls -l client/
total 16
-rw-rw-r-- 1 sli sli  430 Jul 14 16:59 client.cnf
-rw-rw-r-- 1 sli sli 2074 Jul 14 17:00 client.crt
-rw-rw-r-- 1 sli sli 1781 Jul 14 17:00 client.csr  
-rw------- 1 sli sli 3272 Jul 14 17:00 client.key  

$ ls -l router/
total 24
-rw-rw-r-- 1 sli sli  422 Jul 14 16:59 router.cnf
-rw-rw-r-- 1 sli sli 2086 Jul 14 17:00 router.crt
-rw-rw-r-- 1 sli sli 1793 Jul 14 17:00 router.csr
-rw------- 1 sli sli 3272 Jul 14 17:00 router.key
-rw-rw-r-- 1 sli sli 5358 Jul 14 17:00 router.pem  
$ 

$ cp router/router.pem .
$ cp client/client.crt .
$ cp client/client.key .
$ cp ca/ca.crt .
$ ls -l
total 40
drwxr-xr-x 2 sli sli 4096 Jul 14 17:04 ca
-rw-rw-r-- 1 sli sli 2017 Jul 14 17:06 ca.crt
drwxr-xr-x 2 sli sli 4096 Jul 14 16:59 client
-rw-rw-r-- 1 sli sli 2074 Jul 14 17:06 client.crt
-rw------- 1 sli sli 3272 Jul 14 17:06 client.key
drwxr-xr-x 3 sli sli 4096 Jul 14 16:59 config
-rwxr-xr-x 1 sli sli 1293 Jul 14 17:00 Makefile
drwxr-xr-x 2 sli sli 4096 Jul 14 16:59 router
-rw-rw-r-- 1 sli sli 5358 Jul 14 17:06 router.pem
$ 


