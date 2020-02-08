Let's have a quick chat about CSRF. OWASP 2013 had CSRF listed listed as number 8, while 2010 had this listed as number 5. Now, 2017 does not list CSRF. The 2017 list has it dropped, primarily becuase most frameworks include CSRF protections via CSRF tokens. 

I wanted to have a quick chat about it all the same for two reasons. Anytime I have asked someone either at interviews or outside of interviews, they never seem to know what the vulnerability is about. The second reason is that a newer cookie attribute is now available that really helps mitigate this issue. This newer approach can save significant server side resources, becuase the previous solution is really no longer needed (as long as you are only supporting browsers that support sameSite).

> its all about the **COOKIE**

Once you start with Cookies, you are on the right track. Its actually really easy. When you have a Cookie, the browser will always send a cookie to the server when the browser makes a request. 

When you login to bank.com, a cookie will be generated. Any time the browser makes a request to bank.com, the cookie will be included in the request. 

Now, if you are on a malicious.com website, and the malicious.com site makes a request to bank.com, e.g. GET /bank.com/transferfunds.php?from=1&to=2, the browser will send the cookie. OK, this assumes you have already accessed bank.com *recently*. If you close the browser tab, and the cookie is session based, it will still exist in the browser. So, the browser still makes the request and includes the Cookie. 

> No protection from SOP

By the way, SOP (Same Origin Policy) does not protect you here. SOP prevents scripts from reading responses from other origins. But doesn't prevent requests from being issued, e.g., GET an image and other resources from some CDN, SOP prevents scripts from reading content from the responses. Requests and posting *FORMS* are both permitted.

> Standard Prevention

The normal prevention is to include some unique *CSRF* token, that is included with the request. The token is held independently to the Cookie but is included with the request, therefore cross site requests wont be able to read the token (we are ignoring XSS of course!), and wont be able to include the token with the request. Therefore the server side will only see the Cookie and the CSRF token wont be included, and, therefore the server should reject the request. Of course all of this has to include resources on the server side. The newer approach (below) allows removal od CSRF token checking on the server side.

> sameSite Attribute

Browsers have now inlucded a new Cookie attribute called sameSite, you can see this in the dev tools of the browser. You really want to set this value to *STRICT*. This ensures that the browser will only issue the Cookie if and only if the request has come from the same Origin that the cookie is associated with. So, the browsers security realm will help prevent CSRF, this will reduce the overhead on the server side having to validate CSRF tokens!



