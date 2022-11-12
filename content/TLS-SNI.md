+++
title = "SNI in TLS"
date = "2022-08-19"

[taxonomies]
categories=["networking"]
tags=["tls"]
+++

SNI (Server Name Indication) is an extension to the TLS protocol, introduced in 2003. It allows for a single server (or
rather IP address) to host more than one domain and accept encrypted traffic for these.

<!-- more -->

In any modern infrastructure, it's not uncommon for a single server to be a host to a multitude of different domains.
For example, I have a single VPS instance that hosts a numerous amount of different web servers, all accessible via
different subdomains:

-   redwill.se
-   eureka.redwill.se
-   git.redwill.se
-   ...

In my case, these all happen to sit behind the same nginx proxy under the very same IP address. Not only that, these can
only be accessed via HTTPS because the nginx proxy will redirect all HTTP traffic to HTTPS. Regardless of HTTP or HTTPS,
clients will still communicate with the very same IP address for these subdomains, per my intended DNS entries. Using
the same IP address for multiple domains gets a bit problematic for secure HTTPS connection. SNI helps solve this, let's
take a look why:

An SSL certificate (per [X.509][x509]) only applies to a single CN (common name, also known as FQDN - fully qualified
domain name). This effectively means that a distinct server (IP) only can host a single certificate (and by extension a
single domain name capable of encrypted traffic). A TLS connection is not intrinsically aware of domain names - it only
speaks TCP/IP - so by the time the TLS handshake has even started, any domain names involved have already unwrapped to
their underlying IP (often long before the connection is established, due to DNS cache). This leads us to the
introduction of SNI, via [Section 3.1 in RFC 3546][rfc3546]:

SNI (Server Name Indication) solves this by allowing clients, during the very first stages of the TLS handshake, to
provide an SNI in its `ClientHello` message. This gives servers the ability to know exactly which certificate it should
provide in later stages before it even sends its first response (`ServerHello`). Problem solved! Nowadays, to my
knowledge at least, virtually all clients (web browsers, curl, etc.) will always include a `server_name` extension in
the `ClientHello` when a domain name is being requested. Clients that don't support SNI (such as browsers from before
2003) would have a very bad time securely browsing the web today.

The following is an example that highlights this by using the `openssl` CLI to gather the certificate fingerprint. You
can tell that we get a different certificate when providing a different SNI (this is done via the `-servername`
argument). Also note that we provide the IP to connect to in order to further illustrate that TLS only speaks TCP/IP.
The `openssl` client does support providing domain names, however, as mentioned above the domain name is unwrapped
before the TLS connection takes place.

```sh
$ dig +short <my_vps>
188.166.116.169

# -servername redwill.se
$ < /dev/null openssl s_client -servername redwill.se     -connect 188.166.116.169:443 2> /dev/null | openssl x509 -noout -fingerprint
SHA256 Fingerprint=61:DA:3E:65:49:E4:8A:BB:0E:BB:FB:AF:A1:D5:98:03:0E:70:D7:C7:52:D2:85:E7:AF:5D:C2:FD:45:3F:11:8C

# -servername git.redwill.se
$ < /dev/null openssl s_client -servername git.redwill.se -connect 188.166.116.169:443 2> /dev/null | openssl x509 -noout -fingerprint
SHA256 Fingerprint=71:EA:1C:28:40:33:9F:38:25:6E:EC:20:6B:71:E4:2B:62:7B:92:69:E7:80:0C:73:7A:D2:37:D8:93:C9:28:FC

# To further illustrate this, let's project the expiry date
# of the certificate (they're different, although very close to one another)

$ < /dev/null openssl s_client -servername redwill.se     -connect 188.166.116.169:443 2> /dev/null | openssl x509 -noout -dates
notBefore=Jun 24 11:57:02 2022 GMT
notAfter=Sep 22 11:57:01 2022 GMT

$ < /dev/null openssl s_client -servername git.redwill.se -connect 188.166.116.169:443 2> /dev/null | openssl x509 -noout -dates
notBefore=Jun 24 11:56:11 2022 GMT
notAfter=Sep 22 11:56:10 2022 GMT
```

(we do `< /dev/null` to terminate stdin because `openssl s_client` is an interactive command that reads from stdin)

[x509]: https://en.wikipedia.org/wiki/X.509
[rfc3546]: https://www.ietf.org/rfc/rfc3546.txt
