 TLS 1.3 will become the standard shortly, while TLS 1.2 is now considered the de-facto. Apps (servers, services, midtiers, all-your-tiers etc.)  should remove support for TLS 1.1 and 1.0. SSL should have been de-supported already. That's the narrative at least, and of course it's good to push for greater standards and improvements. However, de-supporting a version of TLS comes with the question of compatibility, for example, www.Google.com and www.icloud.com continue to support TLS 1.0 and 1.1 - Why? Frankly speaking, I don't know, my guess is; compatibility reasons. While my on-line bank now only supports TLS 1.2. (though interestingly don't support TLS 1.3), and also interestingly they don't support the more efficient ECDSA signing. 
TLS and the Cipher Suites have a lot of components/elements and moving parts, and in recent times I have had a reason to delve into this a little more. You often hear people talk about my certificate is RSA - what does that mean? Now let me be frank and honest here, I have not read the entire RFC specification for TLS 1.2 and 1.3. I do however want to understand TLS a little more and I have opened the RFC a little and read some of the sections in a few chapters, i.e., enough to have a reasonably good conversation with like minded security engineers. This post wont get into the complex math behind Elliptic Curves (EC), it will suggest why EC might be favored over RSA. 
To summarize this post will outline these key elements:
Cipher Suites
Key Exchange
Perfect Forward Secrecy & Ephemeral
AEAD Ciphers
Elliptic Curves v's RSA
Discrete Log Problem
TLS 1.3 improvements
Pre-Master-Secret
O-RTT

So, lets get started:
Cipher Suites
Do a scan of your web app using a tool such as Qualys [1] or via the NMAP command[2], and you will see a list of Cipher Suites that your Web App / Server supports. So here are some examples:

A Cipher Suite is a list of Crypto stuff that will be used during the TLS handshake process, and will also define how encryption will work for the duration of the user's interaction with your App. As you can see this Cipher Suite version TLS 1.2, it is slightly different to TLS 1.3, but, we will come to that later. 
First up, there are several options listed here, and these are ordered in the preference that the server has defined, so you would expect the first version to be the 'most secure'.  During the TLS handshake process, the Client will indicate its preference and the server will select from its list a common Cipher Suite. The Cipher Suite selected will depend on the Client's Operating System and the Browser being used. Obviously the more recent OS' and Browsers will support most recent Cipher Suites.
So, what do each of these components do, lets go through each component using the first Cipher Suite from above:
			TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS - thats easy, its the protocol, i.e., the TLS version is defined primarily by your server configuration. The client browser can request by suggesting to the server what it supports, but the server typically selects the version. 
ECDHE - The second element defines the Key Exchange. In this case the selected Key Exchange is Diffie Hellman i.e., the 'DH' element. The 'EC' is Elliptic Curve while the last 'E' represents Ephemeral (i.e., short lived). Each of these play an important role. An alternative Key Exchange is RSA. A key exchange is required for switching to Symmetric encryption.
RSA - The third element defines the authentication method i.e., the signing method. In this case its RSA, an alternative is ECDSA. This is dependent on your certificate.
AES_256_GCM - The fourth element defines the symmetric cipher that will be used when transferring data. The bulk of the data is encrypted with a Symmetric encryption. This is because Symmetric encryption is faster than Asymmetric encryption. At present it is very common for this to be AES, while the size of the Symmetric key tends to be 128 or 256. The value 'GCM' defines the mode to use with AES. A common alternative here is 'CBC'. These days the preference is 'GCM' mode, this is tied to AEAD ciphers. In fact if 'CBC' mode is used, it will be flagged as weaker than 'GCM' mode. TLS 1.3 actually mandates AEAD ciphers. More on AEAD below.
SHA386 - The final element identifies the hash signature to use, and this is used for message authentication.

Key Exchange
The reason this concept is needed, is to create a private key for Symmetric encryption. In the Cipher suite example the Symmetric encryption algorithm is AES, and the key size is 256. Both the Client and the Server need to agree on this private key. So, how does this work, previously the most common method was to use RSA, i.e., the Client would set the 'Pre-Master-Secret' key and encrypt this with the servers public key and the server then decrypts with the its private RSA key. More on this 'Pre-Master-Secret' key later. The main issue here is that an important ingredient is transmitted with the 'Pre-Master-Secret', so, the question becomes, is it possible to agree on a key in public without transmitting this particular information? This is where 'Diffie Hellman' 'DH' comes into play. 'DH' allows for key agreement in public, and it's safer than RSA. TLS 1.3 mandates 'DH' to be used, and if I am correct it actually mandates 'DHE', i.e., requires Ephemeral 'DH'. More on Ephemeral later.

Perfect Forward Secrecy & Ephemeral
This PFS concept looks to solve two issues, if the private key of the Asymmetric encryption is compromised, or, somehow a TLS session is decrypted. This is an issue when RSA is used for the key exchange. So, RSA key exchanges are not PFS compliant. 'DHE' solves this issue, first the 'Pre-Master-Secret' is never transmitted (because of what is explained in the key exchange section), and the 'E' (Ephemeral) indicates that the random integer of the Diffie Hellman algorithm is newly generated for each session, rather than relying on the same integer for each session i.e., its short lived. In the Cipher Suite using Qualys [1] you can check if its PFS, look at again at the image above, you can see 'FS' tagged at the end of the Cipher Suite. 

AEAD Ciphers - Authentication Encryption and Associated Data Ciphers
These are ciphers that add Authentication into the process i.e., the cipher includes a MAC of the data. In addition Associated Data is also added into the equation and a MAC is applied to the Associated Data. For example, Associated Data might be the Header attributes of a HTTP request. What is the benefit of adding 'AD'? The reason is to add context to the data. By adding context to the data it becomes more difficult to move blocks of the Cipher around.

Discrete Log Problem
I am not going to explain this with Math, instead I will explain this 


Reference








