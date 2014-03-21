---
layout: post
title:  "butterfly now with ssl authentication"
tags: [projects, butterfly, websocket, ssl, X509, certificate, client authentication]
class: butterfly
---


First of all thanks for all your feedback on butterfly, I never expected this to have such success (wow).
Google Analytics says about 50k visits in 2 days scattered among 164 countries. That's a lot and this led to a fair amount of constructive criticism (and obviously a huge amount of destructive criticism too).


## Security

Security was a major topic among all discussions and therefore several security breaches were fixed. That's why starting with the new 1.4.0 release, the SSL secure mode will now be enabled by default. This works by creating a certificate and a client authentication file ([More info](http://en.wikipedia.org/wiki/Transport_Layer_Security)).

This is a boring process so I added some options in butterfly to make your life easier.

NB: Before going into the details, be aware that the previous behavior is still available using the `--unsecure` option.


### Use butterfly > 1.4.0

As you launch `butterfly.server.py` you will see that it can't start without generating certificates. So let's make them!

#### Generating certificates files
In this example I suppose you run the server as root, you want to access it as user `foo` on a local network and your ip address on this network is `192.168.0.1`:


{% highlight bash %}
$ sudo butterfly.server.py --generate-certs --host="192.168.0.1" # Generate the root certificate for running on local network
$ sudo butterfly.server.py --generate-user-pkcs=foo              # Generate PKCS#12 auth file for user foo
{% endhighlight %}


If you run as root the certificates will be placed in `/etc/buttefly/ssl/`, as user in `~/.butterfly/ssl`.

Once generated, you will have to configure your browser to accept your self signed certificate and the PKCS#12 file (which contains a private key for your user to log in).

#### Importing the root certificate in Google Chrome

Under Google Chrome go to *Settings > Show advanced settings... > HTTPS/SSL > Manage certificates... > Authorities > Import...* and specify the `butterfly_ca.crt` file.

#### Importing the root certificate in Firefox

Under Firefox go to *Edit > Preferences > Advanced > View Certificates > Authorities > Import...* and specify the `butterfly_ca.crt` file.

#### Importing the user client authentication file
For the PKCS#12 you will have to change the file ownership before importing it since this file is a secured file:

{% highlight bash %}
$ sudo cp /etc/butterfly/ssl/foo.p12 /tmp # Copy it in /tmp
$ sudo chown foo /tmp/foo.p12             # Make it readable to the browser
{% endhighlight %}

Then go to the *Your Certificates* tab (for both Google Chrome and Firefox) and import it from `/tmp`. If you set a password during the creation you will have to type it otherwise leave it blank. Some browsers don't seem to appreciate blank passwords, if this doesn't work retry `--generate-user-pkcs` with a password.

{% highlight bash %}
$ sudo butterfly.server.py --host="192.168.0.1" # Run the server
{% endhighlight %}

Now point your browser at [**https://**192.168.0.1:57575/](https://192.168.0.1:57575/) and you are done without typing any password (The authentication informations are contained in the PKCS#12 file).

You will have to run `--generate-certs` every time you want to use a different `--host`, the PKCS#12 will still work though since both host and user files comes from the same CA.


#### Other browsers

For Safari under Mac OS X, you will have to go to the Keychain Access application to import the certificate and the user PKCS#12 files. Sadly Safari seems to have some issues with WebSockets and self-signed certificates and I didn't manage to make it work but I'm far from being an Apple tech and if anyone has succeeded in making it work I'm really interested in your procedure, otherwise file a bug to Apple. It works with the previous browsers in Mac OS though.

For other platforms, search for the browser / OS specific way to import certificates and pray for X509 client auth support. If it's unavailable, use the `--unsecure` option and use ssh tunnels.


## Changes

Since the previous post, a lot of effort has been put into making butterfly safer by removing the HTML escapes for now and by checking websocket origin but also into making it run on more operating systems (FreeBSD, Mac OS, Cygwin, ...).

Incoming changes are listed [on the github page](https://github.com/paradoxxxzero/butterfly/issues?labels=enhancement&page=1&state=open). Please feel free to discuss them.


Have fun and if you want to support my free work go to the [about page](/about/) (Encouraged!)
