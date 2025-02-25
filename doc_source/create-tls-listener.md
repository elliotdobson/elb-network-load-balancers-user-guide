# TLS listeners for your Network Load Balancer<a name="create-tls-listener"></a>

To use a TLS listener, you must deploy at least one server certificate on your load balancer\. The load balancer uses a server certificate to terminate the front\-end connection and then to decrypt requests from clients before sending them to the targets\. Note that if you need to pass encrypted traffic to the targets without the load balancer decrypting it, create a TCP listener on port 443 instead of creating a TLS listener\. The load balancer passes the request to the target as is, without decrypting it\.

Elastic Load Balancing uses a TLS negotiation configuration, known as a security policy, to negotiate TLS connections between a client and the load balancer\. A security policy is a combination of protocols and ciphers\. The protocol establishes a secure connection between a client and a server and ensures that all data passed between the client and your load balancer is private\. A cipher is an encryption algorithm that uses encryption keys to create a coded message\. Protocols use several ciphers to encrypt data over the internet\. During the connection negotiation process, the client and the load balancer present a list of ciphers and protocols that they each support, in order of preference\. The first cipher on the server's list that matches any one of the client's ciphers is selected for the secure connection\.

Network Load Balancers do not support TLS renegotiation or mutual TLS authentication \(mTLS\)\. For mTLS support, create a TCP listener instead of a TLS listener\. The load balancer passes the request through as is, so you can implement mTLS on the target\.

To create a TLS listener, see [Add a listener](create-listener.md#add-listener)\. For related demos, see [TLS Support on Network Load Balancer](https://exampleloadbalancer.com/nlbtls_demo.html) and [SNI Support on Network Load Balancer](https://exampleloadbalancer.com/nlbsni_demo.html)\.

## Server certificates<a name="tls-listener-certificates"></a>

The load balancer requires X\.509 certificates \(server certificate\)\. Certificates are a digital form of identification issued by a certificate authority \(CA\)\. A certificate contains identification information, a validity period, a public key, a serial number, and the digital signature of the issuer\.

When you create a certificate for use with your load balancer, you must specify a domain name\.

We recommend that you create certificates for your load balancers using [AWS Certificate Manager \(ACM\)](https://aws.amazon.com/certificate-manager/)\. ACM integrates with Elastic Load Balancing so that you can deploy the certificate on your load balancer\. For more information, see the [AWS Certificate Manager User Guide](https://docs.aws.amazon.com/acm/latest/userguide/)\.

Alternatively, you can use TLS tools to create a certificate signing request \(CSR\), then get the CSR signed by a CA to produce a certificate, then import the certificate into ACM or upload the certificate to AWS Identity and Access Management \(IAM\)\. For more information, see [Importing certificates](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html) in the *AWS Certificate Manager User Guide* or [Working with server certificates](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_server-certs.html) in the *IAM User Guide*\.

**Important**  
You cannot install certificates with RSA keys larger than 2048\-bit or EC keys on your Network Load Balancer\.

### Default certificate<a name="default-certificate"></a>

When you create a TLS listener, you must specify exactly one certificate\. This certificate is known as the *default certificate*\. You can replace the default certificate after you create the TLS listener\. For more information, see [Replace the default certificate](listener-update-certificates.md#replace-default-certificate)\.

If you specify additional certificates in a [certificate list](#sni-certificate-list), the default certificate is used only if a client connects without using the Server Name Indication \(SNI\) protocol to specify a hostname or if there are no matching certificates in the certificate list\.

If you do not specify additional certificates but need to host multiple secure applications through a single load balancer, you can use a wildcard certificate or add a Subject Alternative Name \(SAN\) for each additional domain to your certificate\.

### Certificate list<a name="sni-certificate-list"></a>

After you create a TLS listener, it has a default certificate and an empty certificate list\. You can optionally add certificates to the certificate list for the listener\. Using a certificate list enables the load balancer to support multiple domains on the same port and provide a different certificate for each domain\. For more information, see [Add certificates to the certificate list](listener-update-certificates.md#add-certificates)\.

The load balancer uses a smart certificate selection algorithm with support for SNI\. If the hostname provided by a client matches a single certificate in the certificate list, the load balancer selects this certificate\. If a hostname provided by a client matches multiple certificates in the certificate list, the load balancer selects the best certificate that the client can support\. Certificate selection is based on the following criteria in the following order:
+ Hashing algorithm \(prefer SHA over MD5\)
+ Key length \(prefer the largest\)
+ Validity period

The load balancer access log entries indicate the hostname specified by the client and the certificate presented to the client\. For more information, see [Access log entries](load-balancer-access-logs.md#access-log-entry-format)\.

### Certificate renewal<a name="ssl-certificate-renewal"></a>

Each certificate comes with a validity period\. You must ensure that you renew or replace each certificate for your load balancer before its validity period ends\. This includes the default certificate and certificates in a certificate list\. Renewing or replacing a certificate does not affect in\-flight requests that were received by the load balancer node and are pending routing to a healthy target\. After a certificate is renewed, new requests use the renewed certificate\. After a certificate is replaced, new requests use the new certificate\.

You can manage certificate renewal and replacement as follows:
+ Certificates provided by AWS Certificate Manager and deployed on your load balancer can be renewed automatically\. ACM attempts to renew certificates before they expire\. For more information, see [Managed renewal](https://docs.aws.amazon.com/acm/latest/userguide/acm-renewal.html) in the *AWS Certificate Manager User Guide*\.
+ If you imported a certificate into ACM, you must monitor the expiration date of the certificate and renew it before it expires\. For more information, see [Importing certificates](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html) in the *AWS Certificate Manager User Guide*\.
+ If you imported a certificate into IAM, you must create a new certificate, import the new certificate to ACM or IAM, add the new certificate to your load balancer, and remove the expired certificate from your load balancer\.

## Security policies<a name="describe-ssl-policies"></a>

When you create a TLS listener, you must select a security policy\. You can update the security policy as needed\. For more information, see [Update the security policy](listener-update-certificates.md#update-security-policy)\.

You can choose the security policy that is used for front\-end connections\. For backend connections, if your TLS listener is using a TLS 1\.3 security policy, the `ELBSecurityPolicy-TLS13-1-0-2021-06` security policy is used\. Otherwise, the `ELBSecurityPolicy-2016-08` security policy is used for backend connections\. Network Load Balancers do not support custom security policies\. 

Elastic Load Balancing provides the following security policies for Network Load Balancers:
+ `ELBSecurityPolicy-TLS13-1-2-2021-06`\*
+ `ELBSecurityPolicy-TLS13-1-2-Res-2021-06`
+ `ELBSecurityPolicy-TLS13-1-2-Ext1-2021-06`
+ `ELBSecurityPolicy-TLS13-1-2-Ext2-2021-06`
+ `ELBSecurityPolicy-TLS13-1-1-2021-06`
+ `ELBSecurityPolicy-TLS13-1-0-2021-06` 
+ `ELBSecurityPolicy-TLS13-1-3-2021-06` 
+ `ELBSecurityPolicy-FS-1-2-Res-2020-10`
+ `ELBSecurityPolicy-FS-1-2-Res-2019-08`
+ `ELBSecurityPolicy-FS-1-2-2019-08`
+ `ELBSecurityPolicy-FS-1-1-2019-08`
+ `ELBSecurityPolicy-FS-2018-06`
+ `ELBSecurityPolicy-TLS-1-2-Ext-2018-06`
+ `ELBSecurityPolicy-TLS-1-2-2017-01`
+ `ELBSecurityPolicy-TLS-1-1-2017-01`
+ `ELBSecurityPolicy-2016-08`\*\* 
+ `ELBSecurityPolicy-TLS-1-0-2015-04`
+ `ELBSecurityPolicy-2015-05` \(identical to `ELBSecurityPolicy-2016-08`\)

\*For TLS listeners, we recommend using the `ELBSecurityPolicy-TLS13-1-2-2021-06` security policy\. This is the default policy for listeners created using the AWS Management Console\. This security policy includes TLS 1\.3, which is optimized for security and performance, and is backward compatible with TLS 1\.2\. 

\*\*`ELBSecurityPolicy-2016-08` is the default security policy for listeners created using the AWS CLI\.

For Forward Secrecy, you can use one of the `ELBSecurityPolicy-FS` policies or an `ELBSecurityPolicy-TLS13` policy\. To meet compliance and security standards that require disabling certain TLS protocol versions, or to support legacy clients that require deprecated ciphers, use one of the `ELBSecurityPolicy-TLS` policies\. Only a small percentage of internet clients require TLS version 1\.0\. 

You can enable access logs for information about the TLS requests sent to your Network Load Balancer, analyze TLS traffic patterns to manage security policy upgrades, and troubleshoot issues\. For more information, see [Access Logs](load-balancer-access-logs.md) and [Network Load Balancer Example Queries](https://docs.aws.amazon.com/athena/latest/ug/networkloadbalancer-classic-logs.html#query-nlb-example)\.

### TLS 1\.3 security policies<a name="tls13-security-policies"></a>

The following table describes the recommended policy \(`ELBSecurityPolicy-TLS13-1-2-2021-06`\) and the other TLS 1\.3 policies\. The `ELBSecurityPolicy-` prefix has been removed from policy names in the heading row so that they fit\.


| Security policies | ![\[TLS13-1-2-2021-06*\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TLS13-1-2-2021-06.png)  | ![\[TLS13-1-3-2021-06\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TLS13-1-3-2021-06.png) | ![\[TLS13-1-2-Res-2021-06\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TLS13-1-2-Res-2021-06.png)  | ![\[TLS13-1-2-Ext2-2021-06\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TLS13-1-2-Ext2-2021-06.png)  | ![\[TLS13-1-2-Ext1-2021-06\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TLS13-1-2-Ext1-2021-06.png)  | ![\[TLS13-1-1-2021-06\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TLS13-1-1-2021-06.png)  | ![\[TLS13-1-0-2021-06\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TLS13-1-0-2021-06.png)  | 
| --- |--- |--- |--- |--- |--- |--- |--- |
| **TLS Protocols** | 
| --- |
|  Protocol\-TLSv1  |  |  |  |  |  |  | ✓ | 
|  Protocol\-TLSv1\.1  |  |  |  |  |  | ✓ | ✓ | 
|  Protocol\-TLSv1\.2  | ✓ |  | ✓ | ✓ | ✓ | ✓ | ✓ | 
| Protocol\-TLSv1\.3 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 
| **TLS Ciphers** | 
| --- |
| TLS\-AES\-128\-GCM\-SHA256 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 
| TLS\-AES\-256\-GCM\-SHA384 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 
| TLS\-CHACHA20\-POLY1305\-SHA256 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES128\-GCM\-SHA256  | ✓ |  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-RSA\-AES128\-GCM\-SHA256  | ✓ |  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES128\-SHA256  | ✓ |  |  | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-RSA\-AES128\-SHA256  | ✓ |  |  | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES128\-SHA  |  |  |  | ✓ |  | ✓ | ✓ | 
|  ECDHE\-RSA\-AES128\-SHA  |  |  |  | ✓ |  | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES256\-GCM\-SHA384  | ✓ |  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-RSA\-AES256\-GCM\-SHA384  | ✓ |  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES256\-SHA384  | ✓ |  |  | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-RSA\-AES256\-SHA384  | ✓ |  |  | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-RSA\-AES256\-SHA  |  |  |  | ✓ |  | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES256\-SHA  |  |  |  | ✓ |  | ✓ | ✓ | 
|  AES128\-GCM\-SHA256  |  |  |  | ✓ | ✓ | ✓ | ✓ | 
|  AES128\-SHA256  |  |  |  | ✓ | ✓ | ✓ | ✓ | 
|  AES128\-SHA  |  |  |  | ✓ |  | ✓ | ✓ | 
|  AES256\-GCM\-SHA384  |  |  |  | ✓ | ✓ | ✓ | ✓ | 
|  AES256\-SHA256  |  |  |  | ✓ | ✓ | ✓ | ✓ | 
|  AES256\-SHA  |  |  |  | ✓ |  | ✓ | ✓ | 

To view the configuration of a security policy for your load balancer using the AWS CLI, use the [describe\-ssl\-policies](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-ssl-policies.html) command\. The default policy in the AWS CLI is `ELBSecurityPolicy-2016-08`\. To upgrade to a TLS 1\.3 security policy using the AWS CLI, use the `ssl-policy` parameter with the [create\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-listener.html) and [modify\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-listener.html) commands\.

### FS security policies<a name="fs-security-policies"></a>

The following table describes the default policy `ELBSecurityPolicy-2016-08` \(default in the AWS CLI\) and the `ELBSecurityPolicy-FS` policies\. `ELBSecurityPolicy-` prefix has been removed from policy names in the heading row so that they fit\. 


| Security policies | ![\[ELBSecurityPolicy-2016-08\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/FS-default.png) | ![\[ELBSecurityPolicy-FS-1-2-Res-2020-10\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/FS-1-2-Res-2020-10.png)  | ![\[ELBSecurityPolicy-FS-1-2-Res-2019-08\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/FS-1-2-Res-2019-08.png)  | ![\[ELBSecurityPolicy-FS-1-2-2019-08\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/FS-1-2-2019-08.png)  | ![\[ELBSecurityPolicy-FS-1-1-2019-08\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/FS-1-1-2019-08.png)  | ![\[ELBSecurityPolicy-FS-2018-06\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/FS-2018-06.png)  | 
| --- |--- |--- |--- |--- |--- |--- |
| **TLS Protocols** | 
| --- |
| Protocol\-TLSv1 | ✓ |  |  |  |  | ✓ | 
| Protocol\-TLSv1\.1 | ✓ |  |  |  | ✓ | ✓ | 
| Protocol\-TLSv1\.2 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 
| **TLS Ciphers** | 
| --- |
| ECDHE\-ECDSA\-AES128\-GCM\-SHA256 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 
| ECDHE\-RSA\-AES128\-GCM\-SHA256 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 
| ECDHE\-ECDSA\-AES128\-SHA256 | ✓ |  | ✓ | ✓ | ✓ | ✓ | 
| ECDHE\-RSA\-AES128\-SHA256 | ✓ |  | ✓ | ✓ | ✓ | ✓ | 
| ECDHE\-ECDSA\-AES128\-SHA | ✓ |  |  | ✓ | ✓ | ✓ | 
| ECDHE\-RSA\-AES128\-SHA | ✓ |  |  | ✓ | ✓ | ✓ | 
| ECDHE\-ECDSA\-AES256\-GCM\-SHA384 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 
| ECDHE\-RSA\-AES256\-GCM\-SHA384 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 
| ECDHE\-ECDSA\-AES256\-SHA384 | ✓ |  | ✓ | ✓ | ✓ | ✓ | 
| ECDHE\-RSA\-AES256\-SHA384 | ✓ |  | ✓ | ✓ | ✓ | ✓ | 
| ECDHE\-RSA\-AES256\-SHA | ✓ |  |  | ✓ | ✓ | ✓ | 
| ECDHE\-ECDSA\-AES256\-SHA | ✓ |  |  | ✓ | ✓ | ✓ | 
| AES128\-GCM\-SHA256 | ✓ |  |  |  |  |  | 
| AES128\-SHA256 | ✓ |  |  |  |  |  | 
| AES128\-SHA | ✓ |  |  |  |  |  | 
| AES256\-GCM\-SHA384 | ✓ |  |  |  |  |  | 
| AES256\-SHA256 | ✓ |  |  |  |  |  | 
| AES256\-SHA | ✓ |  |  |  |  |  | 

### TLS security policies<a name="tls-security-policies"></a>

The following table describes the default policy `ELBSecurityPolicy-2016-08` \(default in the AWS CLI\), and the `ELBSecurityPolicy-TLS` policies\. The `ELBSecurityPolicy-` prefix has been removed from policy names in the heading row so that they fit\.


| Security policies | ![\[ELBSecurityPolicy-2016-08\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/FS-default.png) | ![\[ELBSecurityPolicy-TLS-1-2-Ext-2018-06\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TLS-1-2-Ext-2018-06.png)  | ![\[ELBSecurityPolicy-TLS-1-2-2017-01\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TLS-1-2-2017-01.png)  | ![\[ELBSecurityPolicy-TSL-1-1-2017-01\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TSL-1-1-2017-01.png)  | ![\[ELBSecurityPolicy-TLS-1-0-2015-04\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/TLS-1-0-2015-04.png)  | 
| --- |--- |--- |--- |--- |--- |
| **TLS Protocols** | 
| --- |
|  Protocol\-TLSv1  | ✓ |  |  |  | ✓ | 
|  Protocol\-TLSv1\.1  | ✓ |  |  | ✓ | ✓ | 
|  Protocol\-TLSv1\.2  | ✓ | ✓ | ✓ | ✓ | ✓ | 
| **TLS Ciphers** | 
| --- |
|  ECDHE\-ECDSA\-AES128\-GCM\-SHA256  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-RSA\-AES128\-GCM\-SHA256  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES128\-SHA256  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-RSA\-AES128\-SHA256  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES128\-SHA  | ✓ | ✓ |  | ✓ | ✓ | 
|  ECDHE\-RSA\-AES128\-SHA  | ✓ | ✓ |  | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES256\-GCM\-SHA384  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-RSA\-AES256\-GCM\-SHA384  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES256\-SHA384  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-RSA\-AES256\-SHA384  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  ECDHE\-RSA\-AES256\-SHA  | ✓ | ✓ |  | ✓ | ✓ | 
|  ECDHE\-ECDSA\-AES256\-SHA  | ✓ | ✓ |  | ✓ | ✓ | 
|  AES128\-GCM\-SHA256  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  AES128\-SHA256  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  AES128\-SHA  | ✓ | ✓ |  | ✓ | ✓ | 
|  AES256\-GCM\-SHA384  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  AES256\-SHA256  | ✓ | ✓ | ✓ | ✓ | ✓ | 
|  AES256\-SHA  | ✓ | ✓ |  | ✓ | ✓ | 
|  DES\-CBC3\-SHA  |  |  |  |  | ✓ | 

**\***Do not use this policy unless you must support a legacy client that requires the DES\-CBC3\-SHA cipher, which is a weak cipher\.

To view the configuration of a security policy for your load balancer using the AWS CLI, use the [describe\-ssl\-policies](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-ssl-policies.html) command\.

## ALPN policies<a name="alpn-policies"></a>

Application\-Layer Protocol Negotiation \(ALPN\) is a TLS extension that is sent on the initial TLS handshake hello messages\. ALPN enables the application layer to negotiate which protocols should be used over a secure connection, such as HTTP/1 and HTTP/2\.

When the client initiates an ALPN connection, the load balancer compares the client ALPN preference list with its ALPN policy\. If the client supports a protocol from the ALPN policy, the load balancer establishes the connection based on the preference list of the ALPN policy\. Otherwise, the load balancer does not use ALPN\.Supported ALPN Policies

The following are the supported ALPN policies:

`HTTP1Only`  
Negotiate only HTTP/1\.\*\. The ALPN preference list is http/1\.1, http/1\.0\.

`HTTP2Only`  
Negotiate only HTTP/2\. The ALPN preference list is h2\.

`HTTP2Optional`  
Prefer HTTP/1\.\* over HTTP/2 \(which can be useful for HTTP/2 testing\)\. The ALPN preference list is http/1\.1, http/1\.0, h2\.

`HTTP2Preferred`  
Prefer HTTP/2 over HTTP/1\.\*\. The ALPN preference list is h2, http/1\.1, http/1\.0\.

`None`  
Do not negotiate ALPN\. This is the default\.

**Enable ALPN Connections**  
You can enable ALPN connections when you create or modify a TLS listener\. For more information, see [Add a listener](create-listener.md#add-listener) and [Update the ALPN policy](listener-update-certificates.md#update-alpn-policy)\.