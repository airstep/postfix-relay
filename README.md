# postfix-relay
Postfix SMTP relay docker image. Useful for sending email without using an
external SMTP server.

Default configuration is an open relay that relies on docker networking for
protection. So be careful to not expose it publicly.

## Usage
`docker pull mwader/postfix-relay` or clone/build it yourself. 

All postfix [configuration options](http://www.postfix.org/postconf.5.html)
can be set using `POSTFIX_<name>` environment
variables. See [Dockerfile](Dockerfile) for default configuration. You probably
want to set `POSTFIX_myhostname` (the FQDN used by 220/HELO).

Note that `POSTFIX_myhostname` will change the postfix option
[myhostname](http://www.postfix.org/postconf.5.html#myhostname).

#### Using docker run
```
docker run -e POSTFIX_myhostname=smtp.domain.tld mwader/postfix-relay
```

#### Using docker-compose
```
app:
  # use hostname "smtp" as SMTP server

smtp:
  image: mwader/postfix-relay
  restart: always
  environment:
    - POSTFIX_myhostname=smtp.domain.tld
    - OPENDKIM_DOMAINS=smtp.domain.tld
```

## SPF
When sending email using your own SMTP server it is probably a good idea
to setup [SPF](https://en.wikipedia.org/wiki/Sender_Policy_Framework) for the
domain you're sending from.

## DKIM
To enable [DKIM](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail),
specify a whitespace-separated list of domains in the environment variable
`OPENDKIM_DOMAINS`. The default DKIM selector is "mail", but can be changed to
"`<selector>`" using the syntax `OPENDKIM_DOMAINS=<domain>=<selector>`.

At container start, RSA key pairs will be generated for each domain unless the
file `/etc/opendkim/keys/<domain>/<selector>.private` exists. If you want the
keys to persist indefinitely, make sure to mount a volume for
`/etc/opendkim/keys`, otherwise they will be destroyed when the container is
removed.

DNS records to configure can be found in the container log or by running `docker exec <container> sh -c 'cat /etc/opendkim/keys/*/*.txt` you should see something like this:
```bash
$ docker exec 7996454b5fca sh -c 'cat /etc/opendkim/keys/*/*.txt'

mail._domainkey.smtp.domain.tld. IN	TXT	( "v=DKIM1; h=sha256; k=rsa; "
	  "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0Dx7wLGPFVaxVQ4TGym/eF89aQ8oMxS9v5BCc26Hij91t2Ci8Fl12DHNVqZoIPGm+9tTIoDVDFEFrlPhMOZl8i4jU9pcFjjaIISaV2+qTa8uV1j3MyByogG8pu4o5Ill7zaySYFsYB++cHJ9pjbFSC42dddCYMfuVgrBsLNrvEi3dLDMjJF5l92Uu8YeswFe26PuHX3Avr261n"
	  "j5joTnYwat4387VEUyGUnZ0aZxCERi+ndXv2/wMJ0tizq+a9+EgqIb+7lkUc2XciQPNuTujM25GhrQBEKznvHyPA6fHsFheymOuB763QpkmnQQLCxyLygAY9mE/5RY+5Q6J9oDOQIDAQAB" )  ; ----- DKIM key mail for smtp.domain.tld
```

## License
postfix-relay is licensed under the MIT license. See [LICENSE](LICENSE) for the
full license text.
