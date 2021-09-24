---
title: Configurer les enregistrements DKIM, SPF et DMARC
description: Configurer les enregistrements DKIM, SPF et DMARC obligatoires pour l’email
published: true
date: 2021-09-17T07:25:27.538Z
tags: dkim, spf, dmarc
editor: markdown
dateCreated: 2021-08-13T15:37:16.763Z
---

# Introduction

Nous allons voir comment configurer la sainte trinité qui vous permettra d’être copain avec tous les antispam au monde.

Les enregistrements DKIM, SPF et DMARC sont obligatoires de nos jours pour arriver dans la boite de réception et non dans les spams.

 
# L’enregistrement SPF

L’enregistrement **SPF**, pour **Sender Policy Framework**, est un enregistrement tout bête qui permet d’indiquer au niveau des enregistrements DNS quels serveurs mails sont autorisés à envoyer des mails en votre nom. Rien d’autre.

Le principe est très simple, l’enregistrement devrait l’être aussi.

 

Ce que je vous conseille, et que normalement vous avez déjà, c’est d’avoir des enregistrements MX pour chacun de vos serveurs d’envoi d’email.

Si vous avez ça, alors, l’enregistrement SPF sera le plus simple du monde.

 

Vous faites simplement l’enregistrement suivant de type SPF (pour n’importe quel domaine) :

```dns
v=spf1 mx -all
```
 

La première partie, c’est la version de SPF, **mx** sert à indiquer que l’on doit se référer aux MX existants sur le domaine pour avoir le serveur d’envoi et le **-all** permets de rejeter tous les emails qui ne sont pas envoyés de vos serveurs.

Alors ça c’est la version tout le monde il est beau, tout le monde il est content, en production, j’éviterais quand même.

 

Personnellement, j’indique **MX**, au cas où on oublierait de modifier les DNS, mais je rajoute tous les enregistrements **A:** avec les enregistrements de mes serveurs. Aussi, il est possible que les enregistrements MX globaux soit mal traités par l’antispam de destination (problèmes software ou autre).

Dernière chose, je remplace le **-all** par **~all**, ce qui permet de ne pas rejeter tout ce qui ne correspond pas en cas d’erreur légère sur le traitement du SPF (toujours si vous avez un antispam mal foutu de l’autre côté).

 

En gros, l’enregistrement réel pour mon domaine c’est ça :
```dns
v=spf1 mx a:mx1.nicolas-simond.ch a:mx2.nicolas-simond.ch a:mx3.nicolas-simond.ch ~all
```
 
# L’enregistrement DKIM

Je résume vite fait, **DKIM** ajoute une signature chiffrée dans chaque entête d’email sortant (et juste une partie de l’entête, pas la totalité).

Cette signature, lorsque l’on réceptionne l’email permettra de savoir après déchiffrement si l’email a été altéré en cours de route.

 

La mise en place de DKIM se fait dans votre serveur email avant se faire dans le DNS.
- Pour Exchange : https://www.abyssproject.net/2020/04/mettre-en-place-dkim-avec-exchange-2019/
- Pour Office 365 : https://docs.microsoft.com/en-us/microsoft-365/security/office-365-security/use-dkim-to-validate-outbound-email?view=o365-worldwide
- Pour tout ce qui est basé sur Postfix (le reste en gros) : https://wiki.debian-fr.xyz/Opendkim

 

 
# L’enregistrement DMARC

Le dernier concurrent pour la fin. Toujours de façon simple, **DMARC** permets d’indiquer dans vos DNS ce qui doit se passer au cas où un serveur mail de destination aurait un souci avec vos enregistrements SPF ou DKIM, histoire que vous soyez prévenu.

Le côté maboulien du truc, c’est que vous pouvez indiquer depuis vos DNS de traiter tous VOS emails si jamais le destinataire n’arrive pas à valider votre SPF ou votre DKIM par exemple.

 

Si vous ne voulez pas que vos emails arrivent en quarantaine, mais que vous voulez avoir des rapports en cas de soucis, alors créez une règle comme ceci :

```dns
Enregistrement : _dmarc
Type : TXT
Contenu : "v=DMARC1;p=none;sp=none;pct=100;rua=mailto:dmarc@domaine.com"
```
 

Comme pour le SPF, on commence par la version du protocole.

- **P** et **SP** sont respectivement les actions à appliquer pour les emails non conformes aux enregistrements SPF/DKIM venant de votre domaine ou d’un sous-domaine (none, quarantine ou block).
- **PCT** c’est le pourcentage d’email qui tombent sous le coup de DMARC, on indique 100 pour filtrer tous les emails.
- **RUA** c’est l’adresse email qui recevra les rapports en cas de souci de conformité sur SPF et/ou DKIM.

 
# Test

Attendez bien 30 minutes avant de vous lancer dans cette section.

Selon votre hébergeur, cela pourrait même prendre jusqu’à 48h avec les propagations DNS.
 
Rendez-vous sur https://www.mail-tester.com/, le site vous fournira une adresse mail, envoyez-y simplement un email de test depuis n’importe quelle adresse de votre Exchange et cliquez sur « Check your score ».

Étendez la troisième rubrique et regardez tout ce qui est en rouge sur ma capture, vous devriez être pareil pour le DKIM, le SPF et le DMARC :

![dkim-spf-dmarc.webp](/mails/dkim-spf-dmarc.webp)

Si tout le reste est bien fait, vous devriez avoir 10/10. Si vous souhaitez regarder comment faire un Exchange de A à Z, voici les articles à voir :

- https://www.abyssproject.net/2018/06/installation-de-exchange-2016-de-a-a-z-pour-un-nouveau-domaine/
