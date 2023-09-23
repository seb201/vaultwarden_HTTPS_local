# vaultwarden_HTTPS_local

This tutorial shows how to run a Vaultwarden installation encrypted only on your own network with iOS integration.

I use a Raspberry Pi 3, but also works on other devices.

My username is pi, when your name is not pi change pi in the follow commands

Type in:
1. cd /home/pi
2. mkdir Docker/ssl
3. cd Docker/ssl
4. Create a CA key (your own little on-premise Certificate Authority):
```
openssl genpkey -algorithm RSA -aes128 -out private-ca.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```
Now answer the questions. Actually, it doesn't matter what you enter here. The only thing that matters here is Common Name.
You can also use another Common Name like vaultwarden.com or mypassword.com

5. Create a CA certificate:
```
openssl req -x509 -new -nodes -sha256 -days 3650 -key private-ca.key -out self-signed-ca-cert.crt
```
Note: the -nodes argument prevents setting a pass-phrase for the private key (key pair) in a test/safe environment, otherwise you'll have to input the pass-phrase every time you start/restart the server.

6. Create a bitwarden key:
```
openssl genpkey -algorithm RSA -out bitwarden.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```
7. Create the bitwarden certificate request file:
```
openssl req -new -key bitwarden.key -out bitwarden.csr
```
8. Now create a new text file with nano bitwarden.ext
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = vaultwarden.de
DNS.2 = www.vaultwarden.de
```
<img width="682" alt="Bildschirmfoto 2021-10-03 um 10 35 00" src="https://user-images.githubusercontent.com/35576062/136704477-37750cc3-0a73-42c9-bc3f-634fd4588f84.png">

9. Save the file with CMD+X, accept with Y+ENTER (or J+ENTER)
Create the bitwarden certificate, signed from the root CA:
```
openssl x509 -req -in bitwarden.csr -CA self-signed-ca-cert.crt -CAkey private-ca.key -CAcreateserial -out bitwarden.crt -days 365 -sha256 -extfile bitwarden.ext
```
Note: As of April 2019 iOS 13+ and macOS 15+, the server certificate can not have an expiry > 825 and must include ExtendedKeyUsage extension https://support.apple.com/en-us/HT210176

10. Now we can create a Docker Compose File:
```
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    environment:
      ROCKET_TLS: '{certs="/ssl/bitwarden.crt",key="/ssl/bitwarden.key"}'
      ADMIN_TOKEN: your_own_token
    volumes:
      - /home/pi/Docker/ssl/:/ssl/
      - bw-data:/data/
    ports:
      - "4430:80"
    restart: unless-stopped

volumes:
  bw-data:
```

11. Now you need to set up a local dns forwarding. Some routers can do this. But you can also use a Pi-hole or AdGuard system. If you now enter vaultwarden.de in the local network, you will not land on the vaultwarden.de website but will be redirected to our local Vaultwarden instance. This step is necessary for iOS to accept our self generated certificate.
<img width="1303" alt="Bildschirmfoto 2021-10-03 um 11 00 16" src="https://user-images.githubusercontent.com/35576062/136704505-df5a54b0-c4b6-42ee-a034-c7abb471f607.png">

12. Start the browser and open 
```
https://vaultwarden.de:4430
```

13. Go through the setup


**Install the self-signed certificate on your iOS Device**<br/>
14. Download the certificates from your Raspberry to your Computer. For example with Filezilla
<img width="1312" alt="Bildschirmfoto 2021-10-10 um 19 05 55" src="https://user-images.githubusercontent.com/35576062/136706081-cc06ed86-eb34-40df-b641-cf89d770d2d7.png">

15. Transfer the bitwarden_cert.pem and the bitwarden_key.pem to your iOS Device with AirDrop or Email
<img width="1223" alt="Bildschirmfoto 2021-10-03 um 10 47 03" src="https://user-images.githubusercontent.com/35576062/136706189-c71b2fcf-e72c-44f8-ab19-279c79c2e6ef.png">

16. Install both Certificates<br/>
![IMG_5412](https://user-images.githubusercontent.com/35576062/136706907-fa377009-97e8-4e9e-a2a0-d9c1ee7c3524.PNG)
![IMG_5415](https://user-images.githubusercontent.com/35576062/136706911-4022460e-f395-4195-8748-9c032f6deca6.PNG)
![IMG_5413](https://user-images.githubusercontent.com/35576062/136706923-dbdba9f5-4977-46f7-b297-f35b28889915.PNG)
![IMG_5414](https://user-images.githubusercontent.com/35576062/136706925-965cbe1f-49f8-4b95-85cf-d05189187405.PNG)


If you have set multiple dns servers, it may not work. Set only the DNS server where the DNS forwarding set up above is enabled<br/>
![IMG_5418](https://user-images.githubusercontent.com/35576062/136706225-649f3768-a76a-41b2-b93a-930328a75bfb.PNG)

17. Start the Bitwarden App<br/>
![IMG_5416](https://user-images.githubusercontent.com/35576062/136706404-53b463a1-59cb-4195-8711-c50eb2ca9cda.PNG)

18. Enter at server url
```
https://vaultwarden.de:4430
```
![IMG_5438](https://user-images.githubusercontent.com/35576062/136706415-3034c4a2-c914-497c-bee1-ed64bf6963ac.PNG)


More information:

https://www.reddit.com/r/Bitwarden/comments/ep9qyz/self_signed_certs_iosmacos_issue_solved/

https://github.com/dani-garcia/vaultwarden/wiki/Private-CA-and-self-signed-certs-that-work-with-Chrome

https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development

https://github.com/dani-garcia/vaultwarden/wiki/Enabling-HTTPS

Backup your Vaultwarden Data:

https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault#restoring-backup-data

