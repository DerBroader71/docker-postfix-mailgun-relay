# docker-postfix-mailgun-relay
A docker image that uses postfix as a relay through mailgun. Useful to link to other images.

NOTE: works with domains verified by mailgun

## Configurables

```
SYSTEM_TIMEZONE = UTC or Europe/Berlin
MYNETWORKS = "10.0.0.0/8 172.0.0.0/8 192.168.0.0/16"
EMAIL = postmaster@mg.{YOURDOMAIN}
EMAILPASS = password (is turned into a hash and this env variable is removed at boot)
```

## Example

```bash
docker run -i -t --rm \                                                        
    --name postfix-mailgun-relay \
    -p 9025:25 \
    -e SYSTEM_TIMEZONE="Europe/Berlin" \
    -e MYNETWORKS="10.0.0.0/8 172.0.0.0/8 192.168.0.0/16" \                    
    -e EMAIL="postmaster@mg.{YOURDOMAIN}" \
    -e EMAILPASS="{PASSWORD}" \
    ralfherzog/docker-postfix-mailgun-relay
```

or via docker-compose.yml

```yaml
version: '3'

services:
 postfix-mailgun-relay:
  build: .
  image: "ralfherzog/docker-postfix-mailgun-relay"
  container_name: "postfix-mailgun-relay"
  environment:
   SYSTEM_TIMEZONE: "Europe/Berlin"
   MYNETWORKS: "10.0.0.0/8 172.0.0.0/8 192.168.0.0/16"
   EMAIL: "postmaster@mg.{YOURDOMAIN}"
   EMAILPASS: "{PASSWORD}"
  ports:
  - "9025:25"
```

## Testing

```bash
telnet localhost 9025
```
```text
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 7e9ec6cde4d1 ESMTP
HELO somedomain
250 7e9ec6cde4d1
MAIL FROM:<sender@{YOURDOMAIN}>
250 2.1.0 Ok
RCPT TO:<recevier@{SOMEDOMAIN>
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
From: <sender@{YOURDOMAIN}>
To: <recevier@{SOMEDOMAIN>
Subject: Testmail
Date: Fri, 31 Mar 2017 22:00:00 +0200
 
Testmessage
.
250 2.0.0 Ok: queued as A0F95AAB
QUIT
221 2.0.0 Bye
```

## Troubleshooting

##### Mailgun does not send my mails but I am totally sure my credentials are correct.

Check if the connection is accepted by mailgun. Therefore log in into the container and have a look at the postfix log 
file.

```bash
docker-compose exec postfix-mailgun-relay cat /var/log/mail.log
```

If you see something like
```text
Mar 31 21:13:43 37412245dacc postfix/smtp[153]: connect to smtp.mailgun.org[54.149.68.173]:587: Connection timed out
```
you might be using a blacklisted ip or your internet connection is simply disturbed. You can check this by connecting
to mailgun with telnet.

```bash
telnet smtp.mailgun.org 587
```