---
title: 剖析 SSL 证书
data: 2024-03-01T14:15:10+08:00
categories:
  - favorite
tags:
  - favorite
  - webclipper
  - ssl
  - certificate
  - tls
  - dissecting
origin-link: https://jvns.ca/blog/2017/01/31/whats-tls/
draft: false
---
Hello! In my networking zine (which everyone will be able to see soon), there is a page about TLS/SSL (basically [this tweet](https://twitter.com/b0rk/status/809594614147645440)). But as happens when you write 200 words about a thing on a page, there is a lot more interesting stuff to say. So in this post we will dissect an SSL certificates and try to understand it!

I am not a security person and I am not going to give you security advice for your website (want to know what TLS ciphers you should use? I have no idea!!!). But! I think it’s interesting to know what it means to “issue a SSL certificate” and I can talk about that a little.

### TLS: newer version of SSL

I was confused about what this “TLS” thing was for a long time. Basically newer versions of SSL are called TLS (the version after SSL 3.0 is TLS 1.0). I’m going to just call it “SSL” throughout because that is less confusing to me.

### What’s a certificate?

Suppose I’m checking my email at [https://mail.google.com](https://mail.google.com/)

`mail.google.com` is running a HTTPS server on port 443. I want to make sure that I’m **actually** talking to mail.google.com and not some other random server on the internet owned by EVIL PEOPLE.

This “certificate” business was kind of mysterious to me for a very long time. One day my cool coworker Ray told me that I could connect to a server on the command line and download its certificate!

(If you want to just look at an SSL certificate, you can click on the green lock in your browser and reliably get all the information you need. But this is more fun.)

So, let’s start by looking at mail.google.com’s certificate and deconstruct it a bit.

First, we run `openssl s_client -connect mail.google.com:443`

This is going to print a bunch of stuff, but we’ll just focus on the certificate. Here, it’s this thing:

```
$ openssl s_client -connect mail.google.com:443
...
-----BEGIN CERTIFICATE-----
MIIElDCCA3ygAwIBAgIIMmzfdZnO9pMwDQYJKoZIhvcNAQELBQAwSTELMAkGA1UE
BhMCVVMxEzARBgNVBAoTCkdvb2dsZSBJbmMxJTAjBgNVBAMTHEdvb2dsZSBJbnRl
cm5ldCBBdXRob3JpdHkgRzIwHhcNMTcwMTE4MTg1MjExWhcNMTcwNDEyMTg1MDAw
WjBpMQswCQYDVQQGEwJVUzETMBEGA1UECAwKQ2FsaWZvcm5pYTEWMBQGA1UEBwwN
TW91bnRhaW4gVmlldzETMBEGA1UECgwKR29vZ2xlIEluYzEYMBYGA1UEAwwPbWFp
bC5nb29nbGUuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAiYcr
C9Rn7g9xjsg7khqfRPxUnvpgGyCHqJMXxZGtdf+G02d07cPlMEeaGG12vHyVfRZD
tc/F1ZfwenH6gf0uMobtgw7n2NQa7T7qxuqSUDhZsO1sI1LL/Yqy8QHoooOZQWMz
ytuRA18zti4vQV1dCijADh0+NWI1GDUAKidbaH/fBRrStqBev5Bhq3ZaGj3fDjAO
7CG0Wk3n4Ov2yg44XOdgkLMzjdnbV8l6cZDC7lCK1VsEU1mEd0O0Dw4OcnHLuBPw
IkioZayhPOXDXUS+bhpmtEiCkt8kbHG6jNMC4m8t62Jaf/Si3XNcHhDa4wPCTvid
X//PuuNlRZVg3NjK/wIDAQABo4IBXjCCAVowHQYDVR0lBBYwFAYIKwYBBQUHAwEG
CCsGAQUFBwMCMCwGA1UdEQQlMCOCD21haWwuZ29vZ2xlLmNvbYIQaW5ib3guZ29v
Z2xlLmNvbTBoBggrBgEFBQcBAQRcMFowKwYIKwYBBQUHMAKGH2h0dHA6Ly9wa2ku
Z29vZ2xlLmNvbS9HSUFHMi5jcnQwKwYIKwYBBQUHMAGGH2h0dHA6Ly9jbGllbnRz
MS5nb29nbGUuY29tL29jc3AwHQYDVR0OBBYEFI69aYCEtb2swbJJR3cMOTdcfvZ4
MAwGA1UdEwEB/wQCMAAwHwYDVR0jBBgwFoAUSt0GFhu89mi1dvWBtrtiGrpagS8w
IQYDVR0gBBowGDAMBgorBgEEAdZ5AgUBMAgGBmeBDAECAjAwBgNVHR8EKTAnMCWg
I6Ahhh9odHRwOi8vcGtpLmdvb2dsZS5jb20vR0lBRzIuY3JsMA0GCSqGSIb3DQEB
CwUAA4IBAQAhiqQIwkGp1NmlLq89gjoAfpwaapHuRixxl2S54fyu/4WOHJJafqVA
Tya9J7GTUCyQ6nszCdVizVP26h9TKOs9LJw5jWV9SOnPU2UZKvrNnOUi2FUkCcuD
lsADdKSXNzye3jB88TENrWC/y3ysPdAgPO/sXzyRvNw8SVKl2+RqMDpSRpBptF9e
Lp+WLAM3xKS5SPwCNdCiA332o7qiKRKQm/6bbIWnm7hp/ZnLxbyKaIVytRdiwRNp
O/TTpRv2C708GA3PH6i1pYE86xm3w7lGhN9OiCZpKOJD6ZUH3W20idgPKYPBCO/N
Op2AF3I4iUGeQjXFVLgS6mjUvdLndL9G
-----END CERTIFICATE-----
```

So far, this is unintelligible nonsense. “MIIElDcca… WHAT?!”

It turns out that this nonsense is a format called “X509”, and the `openssl` command knows how to decode it.

So I saved this blob of text to a file called `cert.pem`. You can save it to your computer [from this gist](https://gist.githubusercontent.com/jvns/2c249b8059c0b183c02bb3a71e12e418/raw/b4afd1877471b72f8900b2396c30bc670a5701c9/mail_google_cert.pem) if you want to follow along.

Our next mission is to **parse this certificate**. We do that like this:

```
$ openssl x509 -in cert.pem -text

Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 3633524695565792915 (0x326cdf7599cef693)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=US, O=Google Inc, CN=Google Internet Authority G2
        Validity
            Not Before: Jan 18 18:52:11 2017 GMT
            Not After : Apr 12 18:50:00 2017 GMT
        Subject: C=US, ST=California, L=Mountain View, O=Google Inc, CN=mail.google.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:89:87:2b:0b:d4:67:ee:0f:71:8e:c8:3b:92:1a:
                    9f:44:fc:54:9e:fa:60:1b:20:87:a8:93:17:c5:91:
                    .... blah blah blah ............
                    c2:4e:f8:9d:5f:ff:cf:ba:e3:65:45:95:60:dc:d8:
                    ca:ff
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Alternative Name:
                DNS:mail.google.com, DNS:inbox.google.com

            X509v3 Subject Key Identifier:
                8E:BD:69:80:84:B5:BD:AC:C1:B2:49:47:77:0C:39:37:5C:7E:F6:78
    Signature Algorithm: sha256WithRSAEncryption
         21:8a:a4:08:c2:41:a9:d4:d9:a5:2e:af:3d:82:3a:00:7e:9c:
         1a:6a:91:ee:46:2c:71:97:64:b9:e1:fc:ae:ff:85:8e:1c:92:
         ......... blah blah blah more goes here ...........
```

This is a lot of stuff. Here are the parts of this that I understand

-   `CN=mail.google.com` is the “common name”. Counterintuitively you should ignore this field and look at the “subject alternative name” field instead
-   an **expiry date**: Apr 12 18:50:00 2017 GMT
-   The `X509v3 Subject Alternative Name:` section has the list of domains that this certificate works for. This is mail.google.com and inbox.google.com, which makes sense – they’re both email domains.
-   The `Public Key Info` section tells us the **public key** that we’re going to use to communicate with mail.google.com. We do not have time to explain public key cryptography right now, but this is basically the encryption key we’re going to use to talk secretly.
-   Lastly, the **signature** is a really important thing. Basically anyone could make a certificate for mail.google.com. I could make one right now! But if I gave you that certificate, you would have no reason to believe that it is a real certificate

So let’s talk about certificate signing.

### certificate signing

Every certificate on the internet is basically two parts put together

1.  A certificate (the domain name it’s valid for and public key and other stuff)
2.  A **signature** by someone else. This basically says, “hey, this is okay, Visa says so”

I have a bunch of certificates on my computer in /etc/ssl/certs. Those are the certificates my computer trusts to sign other certificates. For example, I have `/etc/ssl/certs/Staat_der_Nederlanden_EV_Root_CA.pem` on my laptop. Some certificate from the Netherlands! Who knows! If they signed a `mail.google.com` certificate, my computer would be like “yep, looks great, sounds awesome”.

If some random person across the street signed a certificate, my computer would be like “I have no idea who you are”, and reject the certificate.

The mail.google certificate is

-   s:/C=US/ST=California/L=Mountain View/O=Google Inc/CN=mail.google.com
-   which is signed by a “Google Internet Authority G2” certificate
-   which is signed by a “GeoTrust Global CA” certificate
-   which is signed by an “Equifax Secure Certificate Authority” certificate

I have an /etc/ssl/certs/GeoTrust\_Global\_CA.pem file on my computer, which I think is why I trust this mail.google.com certificate. (Geotrust signed Google’s certificate, and Google signed mail.google.com)

### what does getting a certificate issued look like?

So when you get a certificate issued, basically how it works is:

1.  You generate the first half of the certificate (“jvns.ca! expires on X date! This is my public key!”). This part is public.
2.  At the same time, you generate a **private key** for you certificate. You keep this very secret and safe and do not show it to anybody. You’ll use this key every time you establish an SSL connection.
3.  You pay a **certificate authority** (CA) that other computers trust to sign your certificate for you. Certificate authorities are supposed to have integrity, so they are supposed to actually make sure that when they sign certificates, the person they give the cert to actually owns the domain.
4.  You configure your website with your signed certificate and use it to prove that you are really you! Success!


This “certificate authorities are supposed to have integrity thing” I think is why people get so mad when there are [issues like this with Symantec](https://www.symantec.com/page.jsp?id=test-certs-update) where they generated test certificates “to unregistered domains and domains for which Symantec did not have authorization from the domain owner”

### certificate transparency

The last thing we are going to talk about is [certificate transparency](https://www.certificate-transparency.org/what-is-ct). This is a super interesting thing and has a good website and I am almost certainly going to mangle it.

I will try anyway!

So, we said that certificate authorities are “supposed to have integrity”. But there are SO MANY certificate authorities that my computer trusts! And at any time one of them could sign a rogue certificate for mail.google.com. That’s no good.

This isn’t a hypothetical issue – the [certificate transparency](https://www.certificate-transparency.org/what-is-ct) website talks about more than one instance where a CA has been compromised or otherwise has made a mistake.

So, here’s the deal. At any given time, Google **knows** all the valid certificates that are supposed to exist for `mail.google.com` (there is probably only one or something). So certificate transparency is basically a way to make sure that if there is a certificate in circulation for mail.google.com that they DON’T know about, that they can find out.

Here are the steps, as I understand them

1.  Every time any CA signs a certificate, they are supposed to put into a global public “certificate log”
2.  Also the Googlebot puts every certificate it finds on the internet into the certificate log
3.  If a certificate **isn’t** in the log, then my browser will not accept it (or will stop accepting it in the future or something)
4.  Anyone can look at the log at any time to find out if there are rogue certificates in there

So if that CA in the Netherlands signs an evil mail.google.com certificate, they either have to put it in the public log (and Google will find out about their evil ways) or leave it out of the public log (and browsers will reject it).

### setting up SSL stuff is hard

Okay! We have downloaded a SSL certificate and dissected it and learned a few things about it. Hopefully some of you have learned something!

Picking the right settings for your SSL certificates and SSL configuration on your webserver is confusing. As far as I understand it there are about 3 billion settings. [Here is an example of an SSL Labs result for mail.google.com](https://www.ssllabs.com/ssltest/analyze.html?d=mail.google.com&s=216.58.194.165&hideResults=on). There is all this stuff like `OLD_TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256` on that page (for real, that is a real thing.). I’m happy there are tools like SSL Labs that help mortals make sense of all of it.

Someone told me [https://cipherli.st/](https://cipherli.st/) is a way to pick secure SSL configuration if you’re not sure what to do. I don’t know if it’s good or not.

### let’s encrypt is amazing

Also [let’s encrypt](https://letsencrypt.org/) is really cool! They let you have a certificate for your site and make it secure, and you don’t even need to understand all this stuff about how certificates work on the inside! And it’s FREE.
