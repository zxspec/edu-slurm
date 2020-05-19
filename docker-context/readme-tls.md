Using Ubuntu 18.04 + Docker installed with Snappy

## Install
```
openssl rand -out ~/.rnd -hex 256
sudo snap install docker
```
## Post install
https://docs.docker.com/engine/install/linux-postinstall/
```
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
$ sudo snap restart docker.dockerd
```
## TLS
### Docker daemon’s host machine
https://docs.docker.com/engine/security/https/

Generate CA private and public keys
```
$ openssl genrsa -aes256 -out ca-key.pem 4096
$ openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
```
Generate server private and public keys
```
$ openssl genrsa -out server-key.pem 4096
$ openssl req -subj "/CN=dockertmp" -sha256 -new -key server-key.pem -out server.csr
```

Generate client private and public keys
```
$ openssl genrsa -out key.pem 4096
$ openssl req -subj '/CN=client' -new -key key.pem -out client.csr
$ echo extendedKeyUsage = clientAuth > extfile-client.cnf
$ openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out cert.pem -extfile extfile-client.cnf
```
Clean up
```
$ rm -v client.csr server.csr extfile.cnf extfile-client.cnf
$ chmod -v 0400 ca-key.pem key.pem server-key.pem
$ chmod -v 0444 ca.pem server-cert.pem cert.pem
```
Sign the public key with our CA
```
echo subjectAltName = DNS:dockertmp,IP:192.168.1.30,IP:127.0.0.1 >> extfile.cnf
echo extendedKeyUsage = serverAuth >> extfile.cnf
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out server-cert.pem -extfile extfile.cnf
```
### Docker client machine

