## Testing Network Communication in iOS Apps

### Testing Endpoint Identity Verification

#### Overview

-- TODO [Provide a general description of the issue "Testing Endpoint Identity Verification".]

#### Static Analysis

-- TODO [Describe how to assess this given either the source code or installer package (APK/IPA/etc.), but without running the app. Tailor this to the general situation (e.g., in some situations, having the decompiled classes is just as good as having the original source, in others it might make a bigger difference). If required, include a subsection about how to test with or without the original sources.] --

-- TODO [Add content on "Testing Endpoint Identity Verification" with source code] --

#### Dynamic Analysis

-- TODO [Describe how to test for this issue "Testing Endpoint Identity Verification" by running and interacting with the app. This can include everything from simply monitoring network traffic or aspects of the app’s behavior to code injection, debugging, instrumentation, etc.] --

#### Remediation

-- TODO [Describe the best practices that developers should follow to prevent this issue "Testing Endpoint Identity Verification".] --

#### References

#### OWASP Mobile Top 10 2016

* M3 - Insecure Communication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M3-Insecure_Communication

##### OWASP MASVS

* V5.3: "The app verifies the X.509 certificate of the remote endpoint when the secure channel is established. Only certificates signed by a valid CA are accepted."

##### CWE

* CWE-296 - Improper Following of a Certificate's Chain of Trust - https://cwe.mitre.org/data/definitions/296.html
* CWE-297 - Improper Validation of Certificate with Host Mismatch - https://cwe.mitre.org/data/definitions/297.html
* CWE-298 - Improper Validation of Certificate Expiration - https://cwe.mitre.org/data/definitions/298.html

##### Info

- [1] Meyer's Recipe for Tomato Soup - http://www.finecooking.com/recipes/meyers-classic-tomato-soup.aspx
- [2] Another Informational Article - http://www.securityfans.com/informational_article.html

##### Tools

-- TODO [Add relevant tools for "Testing Endpoint Identity Verification"] --

### Testing App Transport Security

#### Overview

App Transport Security (ATS)<sup>[1]</sup> is a set of security checks that the operating system enforces when making connections with NSURLConnection <sup>[2]</sup>, NSURLSession and CFURL<sup>[3]</sup> to public hostnames. ATS is enabled by default for applications build on iOS SDK 9 and above.

ATS is enforced only when making connections to public hostnames. Therefore any connection made to an IP address, unqualified domain names or TLD of .local is  not protected with ATS.

The following is a summarised list of App Transport Security Requirements<sup>[1]</sup>:

- No HTTP connections are allowed
- The X.509 Certificate has a SHA256 fingerprint and must be signed with at least 2048-bit RSA key or a 256-bit Elliptic-Curve Cryptography (ECC) key.
- Transport Layer Security (TLS) version must be 1.2 or above and must support Perfect Forward Secrecy (PFS) through Elliptic Curve Diffie-Hellman Ephemeral (ECDHE) key exchange and AES-128 or AES-256 symmetric ciphers.

The cipher suit must be one of the following:

* TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
* TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
* TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
* TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
* TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA

##### ATS Exceptions

ATS restrictions can be disabled by configuring exceptions in the Info.plist file under the NSAppTransportSecurity key. These exceptions can be applied to:

* allow insecure connections (HTTP),
* lower the minimum TLS version,
* disable PFS and 
* allow connections to local domains

Starting from January 1 2017, Apple App Store review and requires justification if one of the following ATS exceptions are defined. However this decline is extended later by Apple stating "to give you additional time to prepare, this deadline has been extended and we will provide another update when a new deadline is confirmed"<sup>[5]</sup>

* NSAllowsArbitraryLoads - disables ATS globally for all the domains
* NSExceptionAllowsInsecureHTTPLoads - disables ATS for a single domain
* NSExceptionMinimumTLSVersion - enable support for TLS versions less than 1.2

-- TODO: Describe ATS exceptions --

#### Static Analysis

— TODO —

#### Dynamic Analysis

— TODO —

#### Remediation

— TODO —

#### References

— TODO —

##### OWASP Mobile Top 10 2016

* M3 - Insufficient Transport Layer Protection - https://www.owasp.org/index.php/Mobile_Top_10_2014-M3

##### OWASP MASVS

* V5.1: "Data is encrypted on the network using TLS. The secure channel is used consistently throughout the app."
* V5.2: "The TLS settings are in line with current best practices, or as close as possible if the mobile operating system does not support the recommended standards."

##### CWE

— TODO —

##### Info

* [1] Information Property List Key Reference: Cocoa Keys - https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html
* [2] API Reference NSURLConnection - https://developer.apple.com/reference/foundation/nsurlconnection
* [3] API Reference NSURLSession - https://developer.apple.com/reference/foundation/urlsession
* [4] API Reference CFURL - https://developer.apple.com/reference/corefoundation/cfurl-rd7
* [5] Supporting App Transport Security - https://developer.apple.com/news/?id=12212016b

##### Tools

— TODO —

### Testing Custom Certificate Stores and SSL Pinning

#### Overview

Certificate pinning allows to hard-code in the client the certificate that is known to be used by the server. This technique is used to reduce the threat of a rogue CA and CA compromise. Pinning the server’s certificate take the CA out of games. Mobile applications that implement certificate pinning only can connect to a limited numbers of servers, as a small list of trusted CAs or server certificates are hard-coded in the application.

#### Static Analysis

The code presented below shows how it is possible to check if the certificate provided by the server reflects the certificate hard-coded  in the application. The method below implements the connection authentication tells the delegate that the connection will send a request for an authentication challenge.

The delegate must implement connection:canAuthenticateAgainstProtectionSpace: and connection: forAuthenticationChallenge. Within connection: forAuthenticationChallenge, the delegate must call SecTrustEvaluate to perform customary X509 checks. Below a snippet who implements a check of the certificate.  

```Objective-C
(void)connection:(NSURLConnection *)connection willSendRequestForAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge
{
  SecTrustRef serverTrust = challenge.protectionSpace.serverTrust;
  SecCertificateRef certificate = SecTrustGetCertificateAtIndex(serverTrust, 0);
  NSData *remoteCertificateData = CFBridgingRelease(SecCertificateCopyData(certificate));
  NSString *cerPath = [[NSBundle mainBundle] pathForResource:@"MyLocalCertificate" ofType:@"cer"];
  NSData *localCertData = [NSData dataWithContentsOfFile:cerPath];
  The control below can verify if the certificate received by the server is matching the one pinned in the client.
  if ([remoteCertificateData isEqualToData:localCertData]) {
  NSURLCredential *credential = [NSURLCredential credentialForTrust:serverTrust];
  [[challenge sender] useCredential:credential forAuthenticationChallenge:challenge];
}
else {
  [[challenge sender] cancelAuthenticationChallenge:challenge];
}
```

#### Dynamic Analysis

##### Server certificate validation

We start our analysis by testing the application's behaviour while establishing secure connection. Our test approach is to gradually relax security of SSL handshake negotiation and check which security mechanisms are enabled.

1. Having burp set up as a proxy in wifi settings, make sure that there is no certificate added to trust store (Settings -> General -> Profiles) and that tools like SSL Kill Switch are deactivated. Launch your application and check if you can see the traffic in Burp. Any failures will be reported under 'Alerts' tabl. If you can see the traffic, it means that there is no certificate validation performed at all! This effectively means that an active attacker can silently do MiTM against your application. If however, you can't see any traffic and you have an information about SSL handshake failure, follow the next point.
2. Now, install Burp certificate, as explained in [Basic Security Testing section](./0x06b-Basic-Security-Testing.md). If the handshake is successful and you can see the traffic in Burp, it means that certificate is validated against device's trust store, but the pinning is not performed. The risk is less significant than in previous scenario, as two main attack scenarios at this point are misbehaving CAs and phishing attacks, as discussed in [Basic Security Testing section](./0x06b-Basic-Security-Testing.md).
3. If executing instructions from previous step doesn't lead to traffic being proxied through burp, it means that certificate is actually pinned and all security measures are in place. However, you still need to bypass the pinning in order to test the application. Please refer to [Basic Security Testing section](./0x06b-Basic-Security-Testing.md) for more information on this.

##### Client certificate validation

Some applications use two-way SSL handshake, meaning that application verifies server's certificate and server verifies client's certificate. You can notice this if there is an error in Burp 'Alerts' tab indicating that client failed to negotiate connection.

There is a couple of things worth noting:

1. client certificate contains private key that will be used in key exchange
2. usually certificate would also need a password to use (decrypt) it
3. certificate itself can be stored in the binary itself, data directory or the keychain

Most common and improper way of doing two-way handshake is to store client certificate within the application bundle and hardcode the password. This obviously does not bring much security, because all clients will share the same certificate.

Second way of storing the certificate (and possibly password) is to use the keychain. Upon first login, the application should download personal certificate and store it securely in the keychain.

Sometimes application have one certificate that is hardcoded and used for first login and then personal certificate is downloaded. In this case, check if it's possible to still use the 'generic' certificate to connect to the server.

Once you have extracted the certificate from the application (e.g. using Cycript or Frida), add it as client certificate in Burp, and you will be able to intercept the traffic.

#### Remediation

As a best practice, the certificate should be pinned. This can be done in several ways, where most common include:

1. Including server's certificate in the application bundle and performing verification on each connection. This requires an update mechanisms whenever the certificate on the server is updated
2. Limiting certificate issuer to e.g. one entity and bundling the root CA's public key into the application. In this way we limit the attack surface and have a valid certificate.
3. Owning and managing your own PKI. The application would contain the root CA's public key. This avoids updating the application every time you change the certificate on the server, due to e.g. expiration. Note that using your own CA would cause the certificate to be self-singed.

#### References

##### OWASP Mobile Top 10 2016

* M3 - Insecure Communication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M3-Insecure_Communication

##### OWASP MASVS

* V5.4 "The app either uses its own certificate store, or pins the endpoint certificate or public key, and subsequently does not establish connections with endpoints that offer a different certificate or key, even if signed by a trusted CA."

##### CWE

* CWE-295 - Improper Certificate Validation

##### Info

* [1] Setting Burp Suite as a proxy for iOS Devices : https://support.portswigger.net/customer/portal/articles/1841108-configuring-an-ios-device-to-work-with-burp
* [2] OWASP - Certificate Pinning for iOS : https://www.owasp.org/index.php/Certificate_and_Public_Key_Pinning#iOS
