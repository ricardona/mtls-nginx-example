# Example of mTLS with NGINX

This example is to introduce you to the world of mutual authentication. This tutorial walks you through the steps of configuring two-way security using a NGINX server and connect a website. Therefore, I assume you have some familiarity with the above technologies as well as using Bash and Docker.
```
┌─────────┐       ┌─────────┐       ┌─────────┐       ┌─────────┐
│         │       │         │       │         │       │         │
│ Browser ├──────►│ nginx 1 ├──────►│ nginx 2 ├──────►│ Website │
│         │ HTTPS │         │ mTLS  │         │ HTTPS │         │
└─────────┘       └─────────┘       └─────────┘       └─────────┘
```

## Create Server Certificate and Key

Change the working directory to the following to `certs`.

Run the following command to generate the a server certificate and its corresponding private key. We are are going to use X.509 Certificate Data Management which is the standard format for public key certificates which contain the cryptographic key pairs with identities and information related to websites or organizations.

`openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout backend.key -out backend.crt`

For this example the `Common Name (e.g. server FQDN or YOUR name)` must be `backend`.

```
Generating a RSA private key
......+++++
.............................................+++++
writing new private key to 'backend.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:backend
Email Address []:
```

It will prompt for some details, simply hit enter all the way till the end. It will generate the following files:

    `backend.crt` — Certificate for your server.
    `backend.key` — Private key for your server.

In fact, you can now use this self-signed certificate to run your server as https. You should not use self-signed certificate for production server.

## Client Certificate and Key

For mutual TLS authentication, you will need a certificate and private key for client. Run the following command to generate them.

`openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout client.key -out client.crt`

Likewise, you should get the following certificates and private key

    `client.crt`
    `client.key`

The next step is to combine both of them together as PKCS12 file so that you can import them into the client’s browser for mutual TLS authentication. It will prompt you for password. Simply click enter to create a PKCS12 file without a password.

`openssl pkcs12 -export -out client.pfx -inkey client.key -in client.crt`

## Deploy the NGINX mTLS Proxies 

To finally spin up all components implemented before, we use Docker Compose for deploying the NGINX proxies. 

`docker-compose up --build`

## Test the Solution

Visite the page https://localhost or execute:

`curl https://localhost` 

Successful response with correct website should be returned.

## Referencs
https://levelup.gitconnected.com/certificate-based-mutual-tls-authentication-with-nginx-57c7e693
https://mailman.nginx.org/pipermail/nginx/2020-June/059526.html
https://medium.com/geekculture/mtls-with-nginx-and-nodejs-e3d0980ed950
https://smallstep.com/hello-mtls/doc/client/nginx-proxy