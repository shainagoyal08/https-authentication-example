# https-authentication-example
### Introduction:

This document will guide you on how to configure HTTPS listener secured by TLS 1.2 for SSL and two-way SSL (Mutual Authentication)
TLS is cryptographic protocol that secures communications in mule apps. Mule provide out-of-the-box support for HTTPS.

### What is one way SSL and two way SSL?

**One way SSL**: In one way SSL server presents certificate to the client , but the client is not required to present a certificate to the server. The client must authenticate the server, but the server will accept any client into the connection.

**Two way SSL**: In two way SSL both the client and server has to present a certificate before the connection is established between two. With two way SSL authentication, Server not only authenticates itself to the client (which is the minimum requirement for certificate authentication), it also requires authentication from the requesting client. Clients are required to submit digital certificates issued by a trusted certificate authority. 

Mutual Authentication Overview:
1.	A client requests access to a protected resource.
2.	The server presents its certificate to the client.
3.	The client verifies the server’s certificate.
4.	If successful, the client sends its certificate to the server.
5.	The server verifies the client’s credentials.
6.	If successful, the server grants access to the protected resource requested by the client.


**When to use two way SSL**: This type of authentication is useful when you must restrict access to trusted clients only. Two-way SSL authentication is a form of mutual authentication.
 

### What is keystore and truststore?

KeyStore is used to store private key and identity certificates that a specific program should present to both parties (server or client) for verification.

TrustStore is used to store certificates from Certified Authorities (CA) that verify the certificate presented by the server in SSL connection.

| KeyStore	     | TrustStore    | 
| ------------- |:-------------:|
| Keystore stores your credential.  (server or client)	| Truststore stores others credentials (CA)| 
| Keystore is needed when you are setting up the server side on SSL	|Truststore setup is required for the successful connection at the client side|
|Client will store its private key and identify certificate on Keystore	|Server will authenticate the client against the certificate stored on the server’s Truststore|

#### Create keystore and truststore:

Now let’s create KeyStore and truststore which is required to setup one-way and two-way ssl in mule 4

```
Step 1:
keytool -noprompt -validity 365 -genkeypair -v -alias server -keyalg RSA -storetype PKCS12 -keystore server-dev-keystore.p12 -dname "CN=localhost,OU=devServer,O=Mulesoft,L=SFO,ST=CA,c=US" -storepass password -keypass password -ext san="DNS:localhost,IP:127.0.0.1"
Step 2:
keytool -noprompt -validity 365 -genkeypair -v -alias localhost-client -keyalg RSA -storetype PKCS12 -keystore trusted-client-truststore.p12 -dname "CN=localhost,OU=client,O=Shaina,L=eLaptop,ST=CA,c=US" -storepass password -keypass password
Step 3:
keytool -noprompt -export -v -alias server -keystore server-dev-keystore.p12 -storepass password -rfc -file server.cer
Step 4:
keytool -noprompt -import -v -alias server -file server.cer -keystore trusted-client-truststore.p12 -storepass password
```

Steps to configure one-way SSL in Studio:
---

1.	Create a folder named ssl under src/main/resource directory and copy trust store and keystore in ssl folder.
Your project structure should look like below:
  

2.	HTTPS listener configuration for one way ssl:

The tls:context element defines a configuration for TLS, which can be used from both the client and server sides. It can be referenced by other configuration objects of other modules (or defined as a nested element of one of them).

Inside it, you can include two nested elements: key-store and trust-store. You don’t need to include both, but at least one of the two must be present:

*	From the server side: the trust store contains certificates of the trusted clients, the key store contains the private and public key of the server.

*	From the client side: the trust store contains certificates of the trusted servers, the key store contains the private and public key of the client.  

Adding a trust store or a key store to a TLS configuration implicitly implements the corresponding kind of authentication.
Adding both a KeyStore and a trust store to one same config (as in the code example above) implicitly implements two way TLS authentication, also known as mutual authentication.
 

3.	Sample flow config:
```
<flow name="secure_flow_one_way_ssl" doc:id="c8c0e81f-2e13-41a6-9640-62af59118eeb" >
		<http:listener doc:name="GET 8082 /single" doc:id="e6b6fac1-f0a9-4dbd-8fea-fca15b1b7c84" config-ref="Single_HTTPS_Listener" path="/single" allowedMethods="GET"/>
		<ee:transform doc:name="Transform Message" doc:id="b368fe18-7e26-4761-82d3-5738b5f98255">
			<ee:message>
				<ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
	"Output":"Hello World"
}]]></ee:set-payload>
			</ee:message>
		</ee:transform>
	</flow>

```
4.	Now that you have api ready. Let’s test it. I am using postman to test this api. If you are using self-signed certificate, you may have to disabled SSL certificate verification option in settings/preferences.
 
 

Steps to configure two-way SSL in Studio:
---

In case of two-way SSL server needs to validate certificates from clients, then a tls:trust-store element should also be added, with the path field set to the location of the trust store file that contains the certificates of the trusted clients.

1.	HTTPS listener configuration for two way ssl:
```
<http:listener-config name="Dual_HTTPS_Listener" doc:name="HTTP Listener config" doc:id="5fa58c46-1737-4ce4-a06c-2d2e63e95103" >
		<http:listener-connection host="0.0.0.0" port="8083" protocol="HTTPS">
			<tls:context >
				<tls:trust-store path="ssl/trusted-client-truststore.p12" password="password" type="pkcs12" />
				<tls:key-store type="pkcs12" path="ssl/server-dev-keystore.p12" alias="server" keyPassword="password" password="password" />
			</tls:context>
		</http:listener-connection>
	</http:listener-config>

```

2.	Sample flow config:
```
<flow name="secure_flow_mutual_ssl" doc:id="c1dcdf01-568c-4353-a49d-e9239dda6c1c" >
		<http:listener doc:name="GET 8083 /dual" doc:id="12ad9105-5de6-44c2-8ed4-9023e2a417b1" config-ref="Dual_HTTPS_Listener" path="/dual" allowedMethods="GET"/>
		<ee:transform doc:name="Transform Message" doc:id="8a6fcdb1-ebcc-418b-a31c-51864458865c">
			<ee:message>
				<ee:set-payload><![CDATA[%dw 2.0
output application/json
---
{
	"Output":"Hello World"
}]]></ee:set-payload>
			</ee:message>
		</ee:transform>
	</flow>

```

3.	You need to setup client certificates in Postman to test 2 way SSL
Here are the commands to generate client certificate in pem format and key.
```
openssl pkcs12 -in trusted-client-truststore.p12 -out client.pem -clcerts -nokeys
openssl pkcs12 -in trusted-client-truststore.p12 -out key.pem -nocerts
```
 

Now, go to postman settings/preferences -> Navigate to certificate tab -> Click on Add certificate -> add following details in popup -> Hit ADD
 

You are all set to test 2 way authenticate app in postman. Provide service end point and hit send button:
 

The details of SSL connection negotiations are shown in the following figure. When we run the application, we can see these steps from the log, if we turn the debug on. 
``` -Djavas.net.debug=all or -Djavax.net.debug=ssl:handshake ```

 



