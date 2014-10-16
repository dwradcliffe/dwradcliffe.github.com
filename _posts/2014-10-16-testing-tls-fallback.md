---
published: true
layout: post
comments: true
related: false
description: How to test TLS_FALLBACK_SCSV using OpenSSL
title: Testing TLS_FALLBACK_SCSV
---

The POODLE is on the loose and we're all trying to kill off SSLv3 and enable this new TLS_FALLBACK_SCSV thing. I had a hard time finding this information but it's actually pretty easy to test TLS_FALLBACK_SCSV.

You will need the latest version of OpenSSL - 1.0.1j. I hacked my homebrew formula to compile, and I'm sure the forumula will be updated shortly.

We will use the `s_client` command to initiate a connection:

    $ openssl s_client -connect www.example.com:443 -fallback_scsv -no_tls1_2

Here's what that means. We use the `-connect` param to specify the server and port we want to connect to. We use the `-fallback_scsv` param to send TLS_FALLBACK_SCSV in the ClientHello. The final param, `-no_tls1_2` tells the client to not use TLSv1.2 and drop the protocol down to TLSv1.1. This assumes your server supports TLSv1.2. If not, you need to prevent the use of the best supported protocol.

When we issue this command, we will get one of two responses. First let's look at an example of a server that supports TLS_FALLBACK_SCSV properly:

    $ openssl s_client -connect www.google.com:443 -fallback_scsv -no_tls1_2
    140735103538016:error:1407743E:SSL routines:SSL23_GET_SERVER_HELLO:tlsv1 alert inappropriate fallback:s23_clnt.c:770:

We see that the connection fails and the `alert inappropriate fallback` is returned.

Now let's look at a server that does not support TLS_FALLBACK_SCSV:

    $ openssl s_client -connect www.twitter.com:443 -fallback_scsv -no_tls1_2
    CONNECTED(00000003)
    ...
    SSL-Session:
    Protocol  : TLSv1.1
    Cipher    : ECDHE-RSA-AES128-SHA

The connection is successfull even though we attempted a 'fallback' connection.
