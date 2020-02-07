>

Let's have a look at some of the basics of a TLS handshake. TLS has various options and extensions (e.g, the requirement for the Client to issue a certificate and so on). Rather than going into the various extensions and options, I wanted to outline the basic details (i.e., the minimum flow) for the majority of TLS connections.

Of course, if you are looking for the full RFC head over to [TLS RFC5246](https://tools.ietf.org/html/rfc5246).

> Overall

The TLS handshake's objective is to *securely* agree on the highest mutually accepted symmetric encryption algorithm (and the associated parameters e.g. the key, the key-size, the encryption mode etc.). The process starts with asymmetric encryption and then changes to symmetric encryption to transmit the data. This is primarily for performance reasons, as asymmetric encryption is more intensive than symmetric encryption.

In this post I am not going to be explaining the cipher-suites, that will be a separate post.

First, I will provide a summary of the handshake, and go through the main message exchanges, and then, I will list the major differences between TLS 1.2 and 1.3.

> Summary Handshake

Here is the summary flow of a basic handshake:

![TLSHandshake-2](https://www.darraghduffy.ie/content/images/2020/02/TLSHandshake-2.png)

I have merged a few of the flows together, so this is not an exact 1:1 for the messages that are actually sent.

> The Hello messages

The hello messages are designed to agree on various attributes, primarily:

* The protocol version
* Session ID
* Cipher-suite
* Compression method
* Exchange random values for the pre-master key

The SessionID attribute is used in cases where an abbreviated handshake takes place, i.e., the client requests to resume a previous session. If the server agrees (i.e., the sessionID is still in the server's cache), a full handshake is not required, and the previous agreement associated with the SessionID is used. The random values are used to help compute the symmetric private master key. The values are concatenated and input into the PRF (Pseudo Random Function).

> Selected Cipher-suite

In order for the server to send its hello, the server will have selected an agreed cipher-suite. The Client will have provided a list of the cipher-suites that it supports. The Server will have been configured with a *prioritized* list of ciphers. The Server will select the highest mutually accepted cipher. If an agreed cipher-suite could not be found an error is returned at this point.

> Servers Certificate & Verify Certificate

Having completed the hello messages, the server will send its certificate for the client to verify its authenticity (the client can also send a certificate, but this is not included above). The server's certificate must be appropriate for the key exchange algorithm. The certificate message is required to include the certificate chain (i.e., a list). The Server's certificate must be first, and each subsequent certificate must *directly certify* the previous certificate. The root certificate does not need to be included, because the client should already have the root certificate installed. The root certificates will have been distributed independently e.g. with your Windows or Apple Mac OSs'.

> Key Exchange

The Client Key exchange message is *typically* the first message sent by the Client after the Server's hello message is completed. The contents of this message will slightly differ depending on the method of key exchange being used.

The method of key exchange is defined by the selected cipher-suite (cipher-suite will be a separate post). The Key Exchange is either RSA or Diffie-Hellman. This message is all about the *pre-master secret*.

The server can also send a server key exchange message (it actually takes place before the Client Key Exchange), but this is only sent if the certificate does not contain enough information for the client. Usually this is for cases when ephemeral key exchange is being implemented.

> Key Exchange - DH, RSA and Ephemeral (Forward secrecy)

In the DH key exchange process, the actual *pre-master* secret is **not** sent, and so, this is an advantage! While in RSA Key Exchange the *pre-master* secret is issued directly (this approach has more Risk associated with it), in this case, the Client creates a pre-master secret and encrypts with the Server's public key.

DH requires the public exponents to be exchanged, in cases where static exponents are used (i.e., non ephemeral), the Server's certificate will contain the Server's DH public exponent. While the Client's public exponent will be sent with the Client's Key Exchange.

When DH **ephemeral** is being used, the Server will issue its DH public exponent as a Server Key Exchange message (which takes place before the Client Key Exchange and is not shown above).The Client will use the Client Key Exchange to send its DH public exponent.

> Finished

The Client finished message is the first message that is encrypted with the newly agreed symmetric algorithm.

> Differences between TLS 1.2 and 1.3

* The list of supported symmetric encryption algorithms has been updated. Only AEAD (Authenticated Encryption and Associated Data) algorithms are allowed.  

* The key exchange mechanisms allows for **only** forward secrecy (ephemeral style).

* Performance improvement with respect to connection setup, a new zero round trip mode has been added.

* All messages after the server hello are now encrypted.

* TLS version negotiation has been deprecated, and is now based on a version list.
