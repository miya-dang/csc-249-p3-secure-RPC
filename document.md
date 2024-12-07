# 1.    Overview of Application

This application simulates a secure communication between a client and a server, through a VPN. The VPN, as the "man-in-the-middle" is unable to read any sensitive communication between the client and the server. This is because the communication is encrypted through a TLS handshake, using key cryptography and certificate verification. The client and server establishes a symmetric encryption key during the TLS handshake. Then, all messages exchanged between them afterward is encrypted using this symmetric key.

### Certificate Authority:
Establishes trust between the server and client by signing and verifying certificates. 
- Receives unsigned certificates from servers, signs them using its private key, and returns the signed certificate to the server. 
- Makes its public key available. Clients can use this key to verify certificates.

### Server: 
Securely communicates with client. Echoes back client's message.
- Generates an asymmetric public-private key pair. The public key is shared with the client through a certificate, and the private key is kept to decrypt sensitive data.
- Generates a certificate containing its public key, IP address, and port number, and sends it to the Certificate Authority for signing. The signed certificate is used to prove its authenticity to clients.
- During the TLS handshake, the server sends the signed certificate to the client. It receives and decrypts a symmetric key sent by the client, establishing a shared key for secure communication.
- Once the TLS handshake is complete, the server uses the symmetric key to encrypt and decrypt all subsequent communication with the client (echoing back messages received from the client).

### VPN:
An intermediary that forwards communication between the client and server.

### Client:
Initiates secure communication with the server. Then sends a message and receives the echo message back from the server.
- Requests a secure handshake with the server. During the handshake, it verifies the server’s certificate and establishes a shared symmetric encryption key using the server’s public key.
- After the handshake, the client encrypts all outgoing messages with the shared symmetric key and decrypts responses from the server, ensuring secure communication.


# 2.    Format of an unsigned certificate

An unsigned certificate is a string with the format:

(<server_public_key>): <server_IP>: <server_port>

Example: (35212, 56533): 127.0.0.1: 65432

In that:
* 35212, 56533 is the server’s public key.
* 127.0.0.1 is the server's IP address.
* 65432 is the server's listening port.


# 3.    Example output

* Server:
Generated public key '(35212, 56533)' and private key '21321'

Connecting to the certificate authority at IP 127.0.0.1 and port 55553

Prepared the formatted unsigned certificate '(35212, 56533): 127.0.0.1: 65432'

Connection established, sending certificate '(35212, 56533): 127.0.0.1: 65432' to the certificate authority to be signed

Received signed certificate 'D_(12763, 56533)[(35212, 56533): 127.0.0.1: 65432]' from the certificate authority

server starting - listening for connections at IP 127.0.0.1 and port 65432

Connected established with ('127.0.0.1', 53165)

Received handshake request: hello hello hello

Signed certificate sent to client: D_(12763, 56533)[(35212, 56533): 127.0.0.1: 65432]

Received encrypted symmetric key: E_(35212, 56533)[47326]

Decrypted the symmetric key: 47326

TLS handshake complete: established symmetric key '47326', acknowledging to client

Received client message: 'b'HMAC_20008[symmetric_47326[Hello, world]]'' [41 bytes]

Decoded message 'Hello, world' from client

Responding 'Hello, world' to the client

Sending encoded response 'HMAC_20008[symmetric_47326[Hello, world]]' back to the client

server is done!


* VPN:
VPN starting - listening for connections at IP 127.0.0.1 and port 55554

Connected established with ('127.0.0.1', 53164)

Received client message: 'b'127.0.0.1~IP~65432~port~hello hello hello'' [41 bytes]

connecting to server at IP 127.0.0.1 and port 65432

server connection established, sending message 'hello hello hello'

message sent to server, waiting for reply

Received server response: 'b'D_(12763, 56533)[(35212, 56533): 127.0.0.1: 65432]'' [50 bytes], forwarding to client

Received client message: 'b'E_(35212, 56533)[47326]'' [23 bytes], forwarding to server

Received server response: 'b"symmetric_47326[Symmetric key '47326' received]"' [47 bytes], forwarding to client

Received client message: 'b'HMAC_20008[symmetric_47326[Hello, world]]'' [41 bytes], forwarding to server

Received server response: 'b'HMAC_20008[symmetric_47326[Hello, world]]'' [41 bytes], forwarding to client

VPN is done!


* Client:
Connecting to the certificate authority at IP 127.0.0.1 and port 55553

Connection established, requesting public key

Received public key (43770, 56533) from the certificate authority for verifying certificates

Client starting - connecting to VPN at IP 127.0.0.1 and port 55554

Client requesting TLS handshake from server at IP: 127.0.0.1, port: 65432.

Sent TLS handshake request with message: hello hello hello

Received a signed certificate from server: D_(12763, 56533)[(35212, 56533): 127.0.0.1: 65432]

Verifying certificate with certificate authority's public key.

Verification is successfull. Unsigned certificate: (35212, 56533): 127.0.0.1: 65432

Extracted server public key: (35212, 56533), IP: 127.0.0.1, Port: 65432

Server IP and port verified.

Generated symmetric key: 47326

Encrypted symmetric key: E_(35212, 56533)[47326]

Sent the encrypted symmetric key to server.

TLS handshake complete: sent symmetric key '47326', waiting for acknowledgement

Received acknowledgement 'Symmetric key '47326' received', preparing to send message

Sending message 'HMAC_20008[symmetric_47326[Hello, world]]' to the server

Message sent, waiting for reply

Received raw response: 'b'HMAC_20008[symmetric_47326[Hello, world]]'' [41 bytes]

Decoded message 'Hello, world' from server

client is done!


# 4.    Walkthrough of the steps of a TLS handshake, and what each step accomplishes

* Step 1: Client sends TLS handshake request to Server using the client hello message "hello hello hello".

Purpose: Establish initial contact with Server, the client hello message signals the request for secure communication through a TLS handshake.

* Step 2: After receiving request, Server sends a signed certificate to Client.

Purpose: Prove Server's identity to Client, allow Client to verify Server's authenticity. The certificate contains Server’s public key, which Client will use to encrypt the symmetric key later.

* Step 3: Client verifies the received signed certificate with the Certificate Authority's public key. If verification fails, the connection is closed. If verification succeeds, Client receives the unsigned certificate from the CA.

Purpose: Confirm that the certificate was verified by the CA, prevent potential impersonation attacks by malicious servers.

* Step 4: Client extracts Server's public key, IP address, and port number from the unsigned certificate. Then verifies that the information matches the intended recipient.

Purpose: Prevent man-in-the-middle attacks, ensure that Client is communicating with the right Server and not an intermediary attacker.

* Step 5: Client generates a symmetric key to send to Server.
  
Purpose: Ensure secure communication using symmetric encryption. The symmetric key will be used for all subsequent encrypted communication between Client and Server.

* Step 6: Client uses Server's public key to encrypt the generated symmetric key.

Purpose: Prevent the symmetric key from being read and intercepted during transit, and from being used to decrypt further secure communications between Client and Server encrypted and HMAC'd with that key.

* Step 7: Client sends the encrypted symmetric key to Server.

Purpose: Deliver the symmetric key securely to Server, allow Server to decrypt the symmetric key using its private key.

* Step 8: After receiving encrypted symmetric key from Client, decrypts it with Server's private key.

Purpose: Ensure that only the right Server holding the private key can access the symmetric key. The decrypted symmetric key will then be used for encrypting and decrypting all future communication between Client and Server.

* Step 9: Server acknowledges symmetric key receipt.

Purpose: Confirm establishment of symmetric key, finalize the TLS handshake.


# 5.    Two ways in which our simulation fails to achieve real security, and how these failures might be exploited by a malicious party:

* The asymmetric key generation scheme in this simulation relies on simplified operations that are not cryptographically secure. 
Exploit: As it lacks mathematical complexity, an attacker could reverse-engineer the public key to derive the private key, breaking the confidentiality of encrypted messages and compromising the TLS handshake. 

* The certificate authority's public key distribution system: The CA’s public key is transmitted to the client insecurely. 
Exploit: A man-in-the-middle attacker could potentially intercept with this and replace the public key with their own key. This would allow them to sign forged certificates and therefore impersonate the server.

* The encryption/decryption/HMAC/verification algorithms is oversimplified: These operations use basic string manipulation rather than other secure, standard cryptographic algorithms.
Exploit: An attacker could easily decrypt the symmetric key or messages and generate different HMAC values, allowing them to tamper with the messages and the communication channel.


# 6.    Acknowledgement
N/A. I worked alone on this assignment.
