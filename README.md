# https-authentication-example
Introduction:

This document will guide you on how to configure HTTPS listener secured by TLS 1.2 for SSL and two-way SSL (Mutual Authentication)
TLS is cryptographic protocol that secures communications in mule apps. Mule provide out-of-the-box support for HTTPS.

What is one way SSL and two way SSL?

One way SSL: In one way SSL server presents certificate to the client , but the client is not required to present a certificate to the server. The client must authenticate the server, but the server will accept any client into the connection.

Two way SSL: In two way SSL both the client and server has to present a certificate before the connection is established between two. With two way SSL authentication, Server not only authenticates itself to the client (which is the minimum requirement for certificate authentication), it also requires authentication from the requesting client. Clients are required to submit digital certificates issued by a trusted certificate authority. 

Mutual Authentication Overview:
1.	A client requests access to a protected resource.
2.	The server presents its certificate to the client.
3.	The client verifies the server’s certificate.
4.	If successful, the client sends its certificate to the server.
5.	The server verifies the client’s credentials.
6.	If successful, the server grants access to the protected resource requested by the client.


When to use two way SSL: This type of authentication is useful when you must restrict access to trusted clients only. Two-way SSL authentication is a form of mutual authentication.
 

What is keystore and truststore?

KeyStore is used to store private key and identity certificates that a specific program should present to both parties (server or client) for verification.

TrustStore is used to store certificates from Certified Authorities (CA) that verify the certificate presented by the server in SSL connection.

KeyStore	TrustStore
Keystore stores your credential. (server or client)	Truststore stores others credentials (CA)
Keystore is needed when you are setting up the server side on SSL	Truststore setup is required for the successful connection at the client side
Client will store its private key and identify certificate on Keystore	Server will authenticate the client against the certificate stored on the server’s Truststore

Create keystore and truststore

Now let’s create KeyStore and truststore which is required to setup one-way and two-way ssl in mule 4












 
Steps to configure one-way SSL in Studio:

1.	Create a folder named ssl under src/main/resource directory and copy trust store and keystore in ssl folder.
Your project structure should look like below:
  

2.	HTTPS listener configuration for one way ssl:

The tls:context element defines a configuration for TLS, which can be used from both the client and server sides. It can be referenced by other configuration objects of other modules (or defined as a nested element of one of them).
Inside it, you can include two nested elements: key-store and trust-store. You don’t need to include both, but at least one of the two must be present:
•	From the server side: the trust store contains certificates of the trusted clients, the key store contains the private and public key of the server.
•	From the client side: the trust store contains certificates of the trusted servers, the key store contains the private and public key of the client.
Adding a trust store or a key store to a TLS configuration implicitly implements the corresponding kind of authentication. Adding both a KeyStore and a trust store to one same config (as in the code example above) implicitly implements two way TLS authentication, also known as mutual authentication.
 

3.	Sample flow config:

4.	Now that you have api ready. Let’s test it. I am using postman to test this api. If you are using self-signed certificate, you may have to disabled SSL certificate verification option in settings/preferences.
 
 

Steps to configure two-way SSL in Studio:

In case of two-way SSL server needs to validate certificates from clients, then a tls:trust-store element should also be added, with the path field set to the location of the trust store file that contains the certificates of the trusted clients.

1.	HTTPS listener configuration for two way ssl:


2.	Sample flow config:

3.	You need to setup client certificates in Postman to test 2 way SSL
Here are the commands to generate client certificate in pem format and key.

 

Now, go to postman settings/preferences -> Navigate to certificate tab -> Click on Add certificate -> add following details in popup -> Hit ADD
 

You are all set to test 2 way authenticate app in postman. Provide service end point and hit send button:
 

The details of SSL connection negotiations are shown in the following figure. When we run the application, we can see these steps from the log, if we turn the debug on. 
-Djavas.net.debug=all or -Djavax.net.debug=ssl:handshake

 



