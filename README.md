*** execute make to generate SSL certification files needed *** 

my dir
======
drwxr-xr-x 2 sli sli 4096 Jul 12 10:55 ca
drwxr-xr-x 2 sli sli 4096 Jul 12 10:55 client
-rwxr-xr-x 1 sli sli 1293 Jul 12 10:53 Makefile
drwxr-xr-x 2 sli sli 4096 Jul 12 10:55 router
 

sli@linux-36:~/ssl-san$ more ca/ca.cnf
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
sli@linux-36:~/ssl-san$ more client/client.cnf 
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
sli@linux-36:~/ssl-san$ more router/router.cnf 
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
sli@linux-36:~/ssl-san$ more Makefile 
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
sli@linux-36:~/ssl-san$ 

