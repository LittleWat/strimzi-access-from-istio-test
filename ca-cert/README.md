#### create ca

```shell
$ openssl genrsa -out ca.key 2048

$ openssl req -x509 -new -nodes -key ca.key -sha256 -days 1024 -out ca.crt -extensions v3_ca -config openssl-with-ca.cnf
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) []:JP
State or Province Name (full name) []:
Locality Name (eg, city) []:
Organization Name (eg, company) []:
Organizational Unit Name (eg, section) []:
Common Name (eg, fully qualified host name) []:test-ca.local
Email Address []:

$ cat ca.crt | base64 | pbcopy
$ cat ca.key | base64 | pbcopy
```

## ref
- [Generate root CA key and certificate - IBM Documentation](https://www.ibm.com/docs/en/runbook-automation?topic=certificate-generate-root-ca-key)
- [Automatically provision TLS certificates in kubernetess with cert-manager – Michał Wójcik](https://michalwojcik.com.pl/2021/08/29/automatically-provision-tls-certificates-in-kubernetess-with-cert-manager/)
