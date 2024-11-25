# https-ipaddress
How to secure a private IP address with HTTPS (Linux)
## Creating the Keys and Certificates
Create a configuration file myAwesomeCA.cnf for Certificate Authority (CA certificate) and pass in the information about your organisation.
```bash
sudo vim myAwesomeCA.cnf
```
And paste :
```bash
[ req ]
distinguished_name  = req_distinguished_name
x509_extensions     = root_ca

[ req_distinguished_name ]
countryName             = CM
countryName_min         = 2
countryName_max         = 2
stateOrProvinceName     = State or Province Name
localityName            = Locality Name
0.organizationName      = Organization Name
organizationalUnitName  = Organizational Unit Name
commonName              = 57.129.53.74
commonName_max          = 64
emailAddress            = email@sample.com
emailAddress_max        = 64

[ root_ca ]
basicConstraints            = critical, CA:true
```
Create a new file for server configuration, myAwesomeServer.ext. Here we can give the available private IPs (both IPv4 and v6) for the server.
```bash
sudo vim myAwesomeServer.ext
```
Paste : 
```bash
[v3_req]
subjectAltName = @alt_names
extendedKeyUsage = serverAuth

[alt_names]
IP.1 = 57.129.53.74
```
Type the commands below to generate both CA certificates+keys and Server certificates+keys. CA — will be used by the Client’s browser and Server — for server, obviously…
```bash
sudo openssl req -x509 -newkey rsa:2048 -out myAwesomeCA.cer -outform PEM -keyout myAwesomeCA.pvk -days 10000 -verbose -config myAwesomeCA.cnf -nodes -sha256 -subj "/CN=57.129.53.74"
```
```bash
sudo openssl req -newkey rsa:2048 -keyout myAwesomeServer.pvk -out myAwesomeServer.req -subj /CN=57.129.53.74 -sha256 -nodes
```
```bash
sudo openssl x509 -req -CA myAwesomeCA.cer -CAkey myAwesomeCA.pvk -in myAwesomeServer.req -out myAwesomeServer.cer -days 10000 -extfile myAwesomeServer.ext -sha256 -set_serial 0x1111
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
        ssl_certificate /root/certs/myAwesomeCA.cer; # Certificate
        ssl_certificate_key /root/certs/myAwesomeCA.pvk; # Certificate key

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
