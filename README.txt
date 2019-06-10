Prerequisites:
- Apache/2.4.39 (Win64)
- OpenSSL 1.1.1c
- Chrome 74
- Windows 10

Debugging:
- Windows: Event Viewer > Windows Logs > Application for apache errors
- Chrome: DevTools > Security for certificate errors
- Windows: Services to start/stop apache service

Run OpenSSL in a working directory with

1. Create a 2048 bit Certificate Authority (CA) private key

OpenSSL> genrsa -out privkey.pem 2048

output files: privket.pem

2. Create a self signed CA certificate

OpenSSL> req -new -x509 -days 3650 -nodes -key privkey.pem -sha256 -out ca.pem

Country Name (2 letter code) [AU]:NL
State or Province Name (full name):Gelderland
Locality Name (eg, city):Amersfoort
Organization Name(eg, company) [Internet Widgits Pty Ltd]:Secure4U CA

output files: ca.pem

3. Create a server Certificate Signing Request (CSR) and server private key

Create server configuration file called 'server.csr.cnf' with the following content:
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn 

[dn]
C=NL
ST=Gelderland
L=Amersfoort
O=End Point
OU=Pen-Testing
emailAddress=v.packo@secure4u.com
CN=server.com

OpenSSL> req -new -nodes -out server.csr -keyout server.key -config server.csr.cnf

output files: server.csr, server.key

4. Create server certificate

Create server extension file name 'server_v3.ext' with the following content:
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names 

[alt_names]
DNS.1=server.com
DNS.2=www.server.com

OpenSSL> x509 -req -in server.csr -CA ca.pem -CAkey privkey.pem -CAcreateserial -out server.crt -days 3650 -extfile server_v3.ext

output files: server.crt, ca.srl

5. Place 'server.crt' and 'server.key' in a folder somewhere safe

6. Enable SSL on Apache.

Open httpd.conf and enable the following lines:

LoadModule ssl_module modules/mod_ssl.so
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
Include conf/extra/httpd-ssl.conf

Open httpd-ssl.conf and set the SSL-files locations:

SSLCertificateFile "${YOUR_SSL_FILES_FOLDER}/server.crt"
SSLCertificateKeyFile "${YOUR_SSL_FILES_FOLDER}/server.key"

Alternatively, disable SSLEngine and the SSL-files locations and enable these settings in httpd-vhosts.conf

7. Setup SSL virtual host.

Open httpd-vhosts.conf and insert the following:

<VirtualHost *:443>
    ServerAdmin v.packo@secure4u.com
    DocumentRoot "${YOUR_PUBLIC_INDEX_FOLDER}"
    ServerName server.com
    ServerAlias www.server.com
    ErrorLog "logs/server.com-error.log"
    CustomLog "logs/server.com-access.log" common
    SSLEngine On
    SSLCertificateFile "${YOUR_SSL_FILES_FOLDER}/server.crt"
    SSLCertificateKeyFile "${YOUR_SSL_FILES_FOLDER}/server.key"
</VirtualHost>

Make the directory of the project public

<Directory "${YOUR_PUBLIC_INDEX_FOLDER}">
    Require all granted
</Directory>

8. Install the 'ca.pem' file as a trusted root certificate

Run 'mmc'(Microsoft Management Console)
Press Ctrl+M to open Add or Remove Snap-in
Add Certificates from your local machine

Go to Console Root > Trusted Root Certification Authorities > Certificates
Open context menu on Certificates folder and go to All Tasks > Import...
Import the 'ca.pem' certificate

Restart the machine

Browse to:
https://server.com

Click 'View site information' (the "lock") to see if the certificate is valid





