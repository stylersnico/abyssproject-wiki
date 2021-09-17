---
title: Configure DKIM, SPF and DMARC
description: Configure mandatory DKIM, SPF and DMARC records for email
published: true
date: 2021-09-17T07:40:41.373Z
tags: dkim, spf, dmarc
editor: markdown
dateCreated: 2021-09-17T07:40:41.373Z
---

# Introduction

We are going to see how to configure to holy trinity that will allow you to be friend with all the antispam in the world.

The DKIM, SPF and DMARC records are mandatory today to get into the inbox of your contacts.

 
# SPF record

The **SPF** record, for **Sender Policy Framework**, is a simple record that indicate which email servers are allowed senders for your domain.

My tip, is to use your MX record to build automatically the spf, simple as that.


You can use the following (for any domain):

```dns
v=spf1 mx -all
```
 
The first part is the SPF version that we use, **mx** indicate that the server must read the MX records in your domain to grab the list of the allowed servers. 
The **-all** reject all emails not sent from your servers. 

That the simple one, but in case of the remote server is buggy and doesn't read correctly the **mx** record, I add the **A:** records off all my servers like this.

You can also replace **-all** by **~all**, to not reject everything that is not recognized correctly by the antispam.

The real record will look like this:
```dns
v=spf1 mx a:mx1.nicolas-simond.ch a:mx2.nicolas-simond.ch a:mx3.nicolas-simond.ch ~all
```
 
# DKIM record

Simply, **DKIM** add an encrypted signature to the header of all your sent emails

With this, we can check during the reception of the email if it as been altered during the transport.

 

You should setup dkim in your email server before setting up the dns:
- For Exchange: https://www.abyssproject.net/2020/04/mettre-en-place-dkim-avec-exchange-2019/
- For Office 365: https://docs.microsoft.com/en-us/microsoft-365/security/office-365-security/use-dkim-to-validate-outbound-email?view=o365-worldwide
- For postfix-based emails: https://wiki.debian-fr.xyz/Opendkim

 

 
# DMARC record

The last one, **DMARC** allow you to indicate how the remote antispam should proccess your emails if it doesn't respond to your security policy.

If you don't want your emails to go in quarantine but you want to receive reports that indicate a problem, you should configure it like this:

```dns
Enregistrement : _dmarc
Type : TXT
Contenu : "v=DMARC1;p=none;sp=none;pct=100;rua=mailto:dmarc@domaine.com"
```
 

Like SPF, we begin protocol version:

- **P** and **SP** are the action to apply if the email fail spf or dkim (none, quarantine or block).
- **PCT** is the percentage of email to filter (always 100%).
- **RUA** is the address where you want to get your reports.

 
# Test

Wait at least 30 minutes before getting into the test.
epending on your DNS server, please allow 48 hours.
 
Go on https://www.mail-tester.com/, the website will provide you an email address, simply send a real email to it.

Extent the third part and check for SPF, DKIM and DMARC like in this screenshot :

![dkim-spf-dmarc.webp](/mails/dkim-spf-dmarc.webp)

If everything is fine, you should have a 10/10.
