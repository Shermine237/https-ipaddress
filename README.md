# https-ipaddress
How to secure a private IP address with HTTPS (Linux)
## Creating the Keys and Certificates
Create a configuration file openssl-v3-san.cnf and pass in the information about your organisation.
```bash
sudo vim openssl-v3-san.cnf
```
And paste :
```bash
[req]
default_bits       = 2048
prompt             = no
default_md         = sha256
req_extensions     = req_ext
distinguished_name = dn

[dn]
CN = 57.129.53.74

[req_ext]
subjectAltName = @alt_names

[alt_names]
IP.1 = 57.129.53.74

[v3_ca]
subjectAltName = @alt_names
basicConstraints = CA:TRUE

```
Create a private key
```bash
sudo openssl genrsa -out myAwesomeCA.key 2048
```
Generate a CSR (Certificate Signing Request
```bash
sudo openssl req -new -key myAwesomeCA.key -out myAwesomeCA.csr -config openssl-v3-san.cnf
```
Signing a certificate with v3 and SAN
```bash
sudo openssl req -x509 -new -nodes -key myAwesomeCA.key -sha256 -days 3650 \
    -out myAwesomeCA.crt -config openssl-v3-san.cnf -extensions v3_ca
```
## Configure the server certificates
Now all that’s left is to configure the server certificates with Nginx Web server.
```bash
sudo vim /etc/nginx/sites-available/fineract
```
Content sample withs certificate :
```bash
server {
    listen 8443 ssl; # add "ssl" here
    server_name 57.129.53.74;
        ssl_certificate /root/certs/myAwesomeCA.crt; # Certificate
        ssl_certificate_key /root/certs/myAwesomeCA.key; # Certificate key

    location / {
        proxy_pass https://192.168.1.200:8443;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Reload nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
```
## Add certificate to browser
And the most important step is to import the myAwesomeCA.cer file into your browser, this will tell the browser that files being served from our server is legit.
Or just accept the certificate if popup appear.
