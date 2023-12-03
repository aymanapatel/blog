---
title: "Client Hello - Security all the way down"
seoTitle: "Client Hello - Security all the way down"
datePublished: Sun Oct 01 2023 13:41:23 GMT+0000 (Coordinated Universal Time)
cuid: cln7ig0xs000109l54x6b1xpt
slug: client-hello-security-all-the-way-down
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1696163646532/a386f57b-f55d-466c-ae31-515fdbe34fd8.jpeg
tags: cloudflare, security

---

Cloudflare has been working on a standard to encrypt the TLS connection at all levels. We will dig as to what stuff is still unencrypted, how this poses the threat and the solution **ClientHello**

## Pre-requisites

Before diving into the protocol, it is imperative to understand the historical context of this protocol

Existing TLS implementations contain a couple of parameters/extensions that are sent unencrypted and are used before the handshaking process. These are Server Name Identification(SNI) and Application-Layer Protocol Negotiation (ALPN)

While establishing a TLS connection between client and server, these parameters (SNI and ALPN) are not encrypted. It is after these 2 params are exchanged, that the client and server have enough security information (certificates, cryptographic keys .etc.) to initiate a Secure Connection

### SNI

So what is SNI?

* It is sent by the client indicating which hostname it wants to connect to before starting the TCP handshaking process
    
* Since the ISPs can see which hostname the client wants to connect,it allows them to block websites easily.
    

### ALPN

This provides information on what application-level protocol (HTTP version) should be used once the TLS connection has been established.

Having unencrypted ALPN is an attack vector that can be used to downgrade HTTP version and thus negate all security improvements for every HTTP version upgrade.

Since SNI and ALPN are considered metadata and can contain some sensitive information that is unencrypted, there has been a push to make this also encrypted. But this poses a chicken and egg problem, which is how can the client and server exchange encryption key before the handshaking process when the handshaking process itself is used for the same purpose?

This concept of encrypting before handshake was not considered in earlier TLS versions. But after the Snowden leak and the uproar on global surveillance with only using metadata information, IETF started considering ways to encrypt this information as well.

## TLS Prerequisite

A prerequisite for any TLS connection is the TCP handshake. As it is a topic in itself, you can view it [here](https://www.cloudflare.com/en-in/learning/ssl/what-happens-in-a-tls-handshake/). In simple terms, it does these things

1. Acknowledge both parties participating in TLS
    
2. Very each other
    
3. Establish cryptographic algorithms they will use to securely connect and exchange information with each other.
    
4. Specify which TLS version to use (depending on what is supported by each party)
    
5. Authenticate server via Server’s public key and CA’s signature
    

## Basic TLS

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696160651967/d9825b8b-9a3f-4784-8505-da3e359663b8.png align="center")

The above is the initial SYN/ACK between client and server

After that, there is a ClientHello and ServerHello (refer to the below diagram)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696164036463/efd6fa79-be9b-4700-8901-d5015e0189fa.png align="center")

1. **ClientHello:** Cipher Suite, TLS version client supports, client random, SNI and ALN info
    
2. **ServerHello**: Server’s Sl certificate (with CA who issued it), server’s cipher suite, server random
    

The rest of the steps can be viewed in the Cloudflare blog.

Existing TLS implementations contain a couple of parameters/extensions that are sent unencrypted and are used before the handshaking process. These are Server Name Identification(SNI) and Application-Layer Protocol Negotiation (ALPN)

It is after these 2 params are exchanged, that the client and server have enough security information (certificates, cryptographic keys etc.) to initiate a Secure Connection

### SNI

So what is SNI?

* It is sent by the client indicating which hostname it wants to connect to before starting the TCP handshaking process
    
* Since the ISPs can see which hostname the server wants to connect, it allows them to block websites easily.
    

### ALPN

This provides information on what application-level protocol (HTTP version) should be used once the TLS connection has been established.

Having unencrypted ALPN is an attack vector that can be used to downgrade HTTP version and thus negate all security improvements for every HTTP version upgrade.

Since SNI and ALPN are considered metadata and can contain some sensitive information that is unencrypted, there has been a push to make this also encrypted. But this poses a chicken and egg problem, how can the client and server exchange the encryption key before the handshaking process when the handshaking process itself is used for the same purpose?

This concept of encrypting before handshake was not considered in earlier TLS versions. But after the Snowden leak and the uproar on global surveillance with only using metadata information, IETF started considering ways to encrypt this information as well.

## ESNI

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696164113640/93535cb7-cd46-4a01-81dd-4cf5de0e980e.png align="center")

ESNI was the first version of encrypting SNI. It evolved to what we have today which is **ClientHello**

### ESNI issues

For key distribution, ESNI used DNS. And this keyboard distribution plain-text base-64 encoded for the ESNI Public key which would raise serious flags.

```bash
$ dig _esni.crypto.dance TXT +short
"/wGuNThxACQAHQAgXzyda0XSJRQWzDG7lk/r01r1ZQy+MdNxKg/mAqSnt0EAAhMBAQQAAAAAX67XsAAAAABftsCwAAA="
```

This negates the whole security aspect of SNT as plain-text DNS is easily traceable by ISP. One innovation that helped mitigate this was DNS-over-HTTPS (DoH).

# Final puzzle Piece: ECH

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696164148121/bfc5aac1-ccf1-43d8-b1b9-4448f71a3b0d.png align="center")

What ECH as the final draft proposes is to divide **ClientHello,** into 2 parts:

1. **ClientHelloOuter:** This contains information that is not sensitive such as what cipher suite is used, TLS version and **outer SNI.** This outer-SNI can show CDN-type hostnames which would be common for most sites using CDN for improving edge performance.
    

1. **ClientHelloInner:** This would include inner-SNI which would have an actual server name. This would be encrypted by a public key provided by Cloudflare. This could mean Cloudflare to be a point of vulnerability as they can decrypt by the private key that they possess.
    

ECH uses Hybrid Public Key Encryption (HPKE) for exchanging keys.

## Reference: C Structs used in ECH

```c
opaque HpkePublicKey;
uint16 HpkeKemId;  // Defined in <https://www.ietf.org/archive/id/draft-irtf-cfrg-hpke-05.txt>
uint16 HpkeKdfId;  // Defined in <https://www.ietf.org/archive/id/draft-irtf-cfrg-hpke-05.txt>
uint16 HpkeAeadId; // Defined in <https://www.ietf.org/archive/id/draft-irtf-cfrg-hpke-05.txt>

struct {
    HpkeKdfId kdf_id;
    HpkeAeadId aead_id;
} ECHCipherSuite;

struct {
   opaque public_name; // Entity trusted to update encryption keys
   HpkePublicKey public_key; // Public key to encrypt `ClientHelloInner` 
   HpkeKemId kem_id;  // Identifying public key
   ECHCipherSuite cipher_suites; // Cipher suite for encrypting `ClientHelloInner`
   uint16 maximum_name_length; 
   Extension extensions; // 
} ECHConfigContents;

struct {
   uint16 version; // Version of ECH for which this config is used
   uint16 length;  // Length of next field (in bytes)
   select (ECHConfig.version) {
      // ECHConfigContents string
      case 0xfe08: ECHConfigContents contents; 
   }
  } ECHConfig;

ECHConfig ECHConfigs;
```

# ECH in the real world

## Setting up a flag in your browser

Check if your browser has ECH enabled using this [link](https://defo.ie/ech-check.php)

(Chrome, can be achieved by enabling `chrome://flags` under `encrypted-client-hello`)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696166257636/980ca28c-87be-4c9b-a4a0-29f110230ee6.jpeg align="center")

* Before enabling ECH
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696166283045/46c35d56-2295-4743-ab2d-caca52309c6a.jpeg align="center")

* After enabling ECH
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696166311499/ec19fdff-febe-4db6-94e4-6e2f7189e9e2.jpeg align="center")

# No free lunch

Well this helps with securing exchanging metadata before TLS handshake, there are still some concerns/flaws:

1. Issues for corporate networks to implement firewall rules
    
2. OpenSSL open issue for 5 years [Github Link](https://github.com/openssl/openssl/issues/7482)
    
3. There is still not an official RFC number attached (despite Google bringing this to the browser and Cloudflare releasing their [last puzzle piece to the privacy link](https://blog.cloudflare.com/announcing-encrypted-client-hello/).)
    

## References

* RFCs
    
    * Encrypted Client Hello / ESNI Draft: [https://datatracker.ietf.org/doc/draft-ietf-tls-esni/](https://datatracker.ietf.org/doc/draft-ietf-tls-esni/)
        
    * DNS over HTTPS: [https://datatracker.ietf.org/doc/html/rfc8484](https://datatracker.ietf.org/doc/html/rfc8484)
        
* Cloudflare blogs
    
    * [https://blog.cloudflare.com/announcing-encrypted-client-hello/](https://blog.cloudflare.com/announcing-encrypted-client-hello/)
        
    * [https://www.cloudflare.com/learning/ssl/what-is-encrypted-sni/](https://www.cloudflare.com/learning/ssl/what-is-encrypted-sni/)
        
* YouTube talks
    
    * [Encrypted Client Hello: What does it mean?](https://youtu.be/SB--lW7wNZ0?si=k82z9OwQeuCLiAFg)