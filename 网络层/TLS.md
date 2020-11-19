# TSL

When securited by TLS, connections between clients and servers should have one or more the following properties.

1. The connection is private or security because symmetric cryptogrphy is used to encrypt the data transmitted. The keys for symmetric cryptogrphy are genereated fro each connection and are based on the share key negotiated at the start of the session 。The server and the client negotiated the details of algorithm and keys to use before the first byte of data is transimitted.
2. The identify of the communicating parties can be authenticated by the public-key cryptography. 
3. The connection is reliable because each message transmitted includes a message integrity check using a message authentication code to prevent undetected loss or alteration of the data during transmission. 

### why use 443 as https port  ?

 Since application can communicate with or without TLS / SSL, it’s necessary for clients to indicates to the server the setup of TLS connection. For instance, HTTPS use 443 as default port  instead of 80 which normorlly used by the HTTP

## Handshake procedure

客户端和服务器同意使用TLS后，他们将通过握手过程协商有状态连接

Once the client and the server negotiated use TLS, the create a stateful connection through the using of the handshake procedure

1. 当客户端连接到启用了TLS的服务器并请求安全连接并且客户端提供了受支持的密码套件列表时，握手开始
2. The handshake begins when a client requesting a secure connection to a TLS-enabled server and presenting a list of support cypher-suits(ciphers and hash-function).
3. From the presenting list, the server picks one of cipher-suits that it also support and notifies the client.
4. The server then provide a ceritificate, usually contains server name, countru ext.
5. The client confirms and validate the certificate before proceeding.
6. Generate a session-key to use in symmetric crptography.
   1. encrypt a random-number and encrypt it by the server-provide public-key, the cipher-result can only be decrypt by the server.
   2. uses [Diffie–Hellman key exchange](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange) to securely generate a random and unique session key for encryption and decryption that **has the additional property of forward secrecy**: if the server's private key is disclosed in future, it cannot be used to decrypt the current session, even if the session is intercepted and recorded by a third party.

## Algorithm

The TLS algorithm including three parts

 1. key exchange : [DHE](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange)**-**[RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) **(**[forward secrecy](https://en.wikipedia.org/wiki/Transport_Layer_Security#Forward_secrecy)),[ECDHE](https://en.wikipedia.org/wiki/Elliptic-curve_Diffie–Hellman)**-**[ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_DSA) **(**[forward secrecy](https://en.wikipedia.org/wiki/Transport_Layer_Security#Forward_secrecy)**)** 

 2. Cipher ： AES-GCM / AEC-CCM / 3desc

 3. Data intergrity :  [HMAC](https://en.wikipedia.org/wiki/HMAC)-[SHA256/384](https://en.wikipedia.org/wiki/SHA-2) ， AEAD

example : 

​	`TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384` 

1. ECDHE : key exchange algorithm
2. ECDSA : signature algorithm
3. AES-256-GCM:  symmetric cipher algorithm
   1. SHA284(HMAC-SHA384) : data integrity 

## Protocol details

#### Basic TLS handshake

​	Negotiation Phash

1. client sends `client hello` msg specifying the highest TLS protocol version it support, a list of suggested [cipher suites](https://en.wikipedia.org/wiki/Cipher_suite) and suggested compression methods. if a client is attempted to resumed a handshake, it may send a session ID.
2. Server response with a `server hello` msg, containing chosen cipher suit, a random number, protocol version and compression method. which is offered by the client.
   1. currently, the random number that server sends to client is plaintext.
3. Server sends its `certificate msg`.
4. Server sends its `ServerKeyExchange` msg
5. Server sends`ServerHelloDone` msg, indicating it is done with handshake negotiation
6. client response with `ClientKeyExchange`, which may contains a `PreMasterSecret`, pulick-key,or nothing. The preMasterSecret is using when 
   1. **The client and server then use the random numbers and *PreMasterSecret* to compute a common secret, called the "master secret".** All other key data for this connection is derived from this master secret (and the client- and server-generated random values), which is passed through a carefully designed [pseudorandom](https://en.wikipedia.org/wiki/Pseudorandomness) function.
7. The client now sends a `ChangeCipherSpec`  msg, esstential telling the server that, from now on everying I send to you is authenticated and encrypted
8. Finally, the server sends a **ChangeCipherSpec**, telling the client, "Everything I tell you from now on will be authenticated (and encrypted, if encryption was negotiated)."



## Resumed TLS handshake

Publick-key operation are relatively expensive in therms of computational power. TLS provide a secure shortcut on the handshake mechanism, it can avoid following operation : resums sessions. Resumed sessions could be implemted using session ids or session tickets.

##### Session Id  

session id is send from server to client as a part of `ServerHello` message. client associates session is with server’s ip address and port, so that when client connect the server again, it can use the session id to shortcut handshake. The session-id maps to the cryptography parameters especially the “master key” which is sent from client to server when doing a full handshake previously.

##### Session Ticket 

As for session tocket, it solved the problem that the server has to store all the handshake master security keys, In other words :  session ticket is send from client to server and it containing `master key` and the master key is is encrypt by the server configed aes key. **So the session ticket mechanism is all config by the server, detail check golang’s tls implementation** [decryptTicket](https://github.com/golang/go/blob/6f08e89ec3280bf6577c2bdb01243cbeeb1a259d/src/crypto/tls/ticket.go#L147)  , [encryptTicket](https://github.com/golang/go/blob/6f08e89ec3280bf6577c2bdb01243cbeeb1a259d/src/crypto/tls/ticket.go#L119)

 



 

**in the resumed TLS handshake, there is no `master key` be sent either from client to server or server to client , it only have the session id!**

1. Client send `clienthello` msg include a session id from previous TLS connection, and other msg such as support cipher suits, a random number and highest TLS protocol version
2. The server received the `client hello` message and got the previous session id, the server uses this session id to retrive the master key (verify phase.)
3. server then sends  **ChangeCipherSpec** record, essentially telling the client, "Everything I tell you from now on will be encrypted."
   1. Finally, the server sends an encrypted **Finished** message, containing a hash and MAC over the previous handshake messages.
   2. The client will attempt to decrypt the server's *Finished* message and verify the hash and MAC. If the decryption or verification fails, the handshake is considered to have failed and the connection should be torn down.
4. Finally, the client sends a **ChangeCipherSpec**, telling the server, "Everything I tell you from now on will be encrypted. "



## How TLS used DHE( Diffie-Hellman Ephemeral)  not  Diffie-Hellman key exchange 

[HTTPS: The TLS Handshake Using Diffie-Hellman Ephemeral]([https://thecybersecurityman.com/2018/04/25/https-the-tls-handshake-using-diffie-hellman-ephemeral/#:~:text=The%20client%20and%20server%20must,%2DHellman%20Ephemeral%20(DHE).&text=The%20server%20picks%20a%20secret,where%20X%20%3D%20server's%20DH%20parameter.](https://thecybersecurityman.com/2018/04/25/https-the-tls-handshake-using-diffie-hellman-ephemeral/#:~:text=The client and server must,-Hellman Ephemeral (DHE).&text=The server picks a secret,where X %3D server's DH parameter.))



## Forward serecy

**An implementation of TLS can provide forward secrecy by requiring the use of ephemeral [Diffie–Hellman key exchange](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange) to establish session keys**, however  many clients and servers supporting TLS (including browsers and web servers) are not configured to implement such restrictions. In practice, unless a web service uses Diffie–Hellman key exchange to implement forward secrecy, all of the encrypted web traffic to and from that service can be decrypted by a third party if it obtains the server's master (private) key; e.g., by means of a court order.







   

   

 