---
Title: How to recover a client cert from chrome
Tags: SSL chrome Linux
Summary: Sometimes after migrating workstations, I realised I forgot to take care of one of my certs. A client cert.
Now the workstation in question has been dealt with and disposed...
---

Sometimes after migrating workstations, I realised I forgot to take care of one of my certs. A client cert. 
Now the workstation in question has been dealt with and disposed (it was a viking funeral, very intense emotionally and with only close friends), but its innards will remain with us forever, in the form of an image of the old HDD.

Now for the meaty bits, aka "how to recover that @#$% client certificate"

Using your package manager of choice, install "libnss3-tools" and do the following:
    certutil -d sql:$OLD-HOME/.pki/nssdb -L

You should get something looking like this:
    Certificate Nickname                                         Trust Attributes
                                                               	SSL,S/MIME,JAR/XPI
    
    <Certificate name>						 u,u,u

Now all you need to do is:
    pk12util -o certfile.p12 -d sql:$OLD-HOME/.pki/nssdb -n "<Certificate name>"


You can now import that certificate in your browser of choice!
