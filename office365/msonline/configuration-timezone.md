---
title: Configuration de la Timezone dans Office 365
description: Configuration de la Timezone et des paramètres de langue dans Office 365
published: true
date: 2023-08-10T12:16:04.696Z
tags: powershell, office 365
editor: markdown
dateCreated: 2023-08-10T12:16:04.696Z
---

# Introduction
Cette procédure permet de modifier la timezone par défaut des boites emails existantes dans Office 365.


# Affichage de la configuration actuelle

Connectez-vous à Exchange Online avec la commande suivante :
```powershell
Connect-ExchangeOnline -UserPrincipalName admin@tenant.ch
```


Vous pourrez ensuite obtenir la configuration actuelle des boites avec la commande suivante : 

```powershell
Get-Mailbox -ResultSize Unlimited | Get-MailboxRegionalConfiguration
```

![msonline-language-01.png](/generique/office365/msonline/language/msonline-language-01.png)



# Modification de la configuration

Lancez la commande suivante pour passer toutes les boites en français suisse : 
```powershell
Get-Mailbox -ResultSize Unlimited |  Set-MailboxRegionalConfiguration -Language 4108 -TimeZone "W. Europe Standard Time" -DateFormat "dd.MM.yyyy" -LocalizeDefaultFolderName
```

> **4108** = French (Switzerland)
> **W. Europe Standard Time** = (GMT+01:00) Amsterdam, Berlin, Bern, Rome, Stockholm, Vienna 
> **LocalizeDefaultFolderName** = Renomme **Inbox** en **Boite de réception**
{.is-info}


## Language (Locale)
```
Language (Locale) 	Code
Arabic (Algeria) 	5121
Arabic (Bahrain) 	15361
Arabic (Egypt) 	3073
Arabic (Iraq) 	2049
Arabic (Jordan) 	11265
Arabic (Kuwait) 	13313
Arabic (Lebanon) 	12289
Arabic (Libya) 	4097
Arabic (Morocco) 	6145
Arabic (Oman) 	8193
Arabic (Qatar) 	16385
Arabic (Saudi Arabia) 	1025
Arabic (Syria) 	10241
Arabic (Tunisia) 	7169
Arabic (U.A.E.) 	14337
Arabic (Yemen) 	9217
Basque 	1069
Bulgarian 	1026
Catalan 	1027
Chinese (Hong Kong S.A.R) 	3076
Chinese (Macau S.A.R) 	5124
Chinese (People’s Republic of China) 	2052
Chinese (Singapore) 	4100
Chinese (Taiwan) 	1028
Croatian 	1050
Czech 	1029
Danish 	1030
Dutch (Belgium) 	2067
Dutch (Netherlands) 	1043
English (Australia) 	3081
English (Belize) 	10249
English (Canada) 	4105
English (Caribbean) 	9225
English (Ireland) 	6153
English (Jamaica) 	8201
English (New Zealand) 	5129
English (Republic of the Philippines) 	13321
English (South Africa) 	7177
English (Trinidad) 	11273
English (United Kingdom) 	2057
English (United States) 	1033
English (Zimbabwe) 	12297
Estonian 	1061
Filipino (Philippines) 	1124
Finnish 	1035
French (Belgium) 	2060
French (Canada) 	3084
French (France) 	1036
French (Luxembourg) 	5132
French (Principality of Monaco) 	6156
French (Switzerland) 	4108
German (Austria) 	3079
German (Germany) 	1031
German (Liechtenstein) 	5127
German (Luxembourg) 	4103
German (Switzerland) 	2055
Greek 	1032
Hebrew 	1037
Hindi 	1081
Hungarian 	1038
Icelandic 	1039
Indonesian 	1057
Italian (Italy) 	1040
Italian (Switzerland) 	2064
Japanese 	1041
Kazakh 	1087
Korean 	1042
Latvian 	1062
Lithuanian 	1063
Malay 	1086
Norwegian (Bokmål) 	1044
Persian 	1065
Polish 	1045
Portuguese (Brazil) 	1046
Portuguese (Portugal) 	2070
Romanian 	1048
Russian 	1049
Serbian (Cyrillic) 	3098
Serbian (Latin) 	2074
Slovak 	1051
Slovenian 	1060
Spanish (Argentina) 	11274
Spanish (Bolivia) 	16394
Spanish (Chile) 	13322
Spanish (Colombia) 	9226
Spanish (Costa Rica) 	5130
Spanish (Dominican Republic) 	7178
Spanish (Ecuador) 	12298
Spanish (El Salvador) 	17418
Spanish (Guatemala) 	4106
Spanish (Honduras) 	18442
Spanish (Mexico) 	2058
Spanish (Nicaragua) 	19466
Spanish (Panama) 	6154
Spanish (Paraguay) 	15370
Spanish (Peru) 	10250
Spanish (Puerto Rico) 	20490
Spanish (International Sort) 	3082
Spanish (Traditional Sort) 	1034
Spanish (Uruguay) 	14346
Spanish (Venezuela) 	8202
Swedish (Finland) 	2077
Swedish (Sweden) 	1053
Thai 	1054
Turkish 	1055
Ukrainian 	1058
Urdu 	1056
Vietnamese 	1066
```

## Timezone

```
Index 	Name of Time Zone 	Time
000 	Dateline Standard Time 	(GMT-12:00) International Date Line West
001 	Samoa Standard Time 	(GMT-11:00) Midway Island, Samoa
002 	Hawaiian Standard Time 	(GMT-10:00) Hawaii
003 	Alaskan Standard Time 	(GMT-09:00) Alaska
004 	Pacific Standard Time 	(GMT-08:00) Pacific Time (US and Canada); Tijuana
010 	Mountain Standard Time 	(GMT-07:00) Mountain Time (US and Canada)
013 	Mexico Standard Time 2 	(GMT-07:00) Chihuahua, La Paz, Mazatlan
015 	U.S. Mountain Standard Time 	(GMT-07:00) Arizona
020 	Central Standard Time 	(GMT-06:00) Central Time (US and Canada
025 	Canada Central Standard Time 	(GMT-06:00) Saskatchewan
030 	Mexico Standard Time 	(GMT-06:00) Guadalajara, Mexico City, Monterrey
033 	Central America Standard Time 	(GMT-06:00) Central America
035 	Eastern Standard Time 	(GMT-05:00) Eastern Time (US and Canada)
040 	U.S. Eastern Standard Time 	(GMT-05:00) Indiana (East)
045 	S.A. Pacific Standard Time 	(GMT-05:00) Bogota, Lima, Quito
050 	Atlantic Standard Time 	(GMT-04:00) Atlantic Time (Canada)
055 	S.A. Western Standard Time 	(GMT-04:00) Caracas, La Paz
056 	Pacific S.A. Standard Time 	(GMT-04:00) Santiago
060 	Newfoundland and Labrador Standard Time 	(GMT-03:30) Newfoundland and Labrador
065 	E. South America Standard Time 	(GMT-03:00) Brasilia
070 	S.A. Eastern Standard Time 	(GMT-03:00) Buenos Aires, Georgetown
073 	Greenland Standard Time 	(GMT-03:00) Greenland
075 	Mid-Atlantic Standard Time 	(GMT-02:00) Mid-Atlantic
080 	Azores Standard Time 	(GMT-01:00) Azores
083 	Cape Verde Standard Time 	(GMT-01:00) Cape Verde Islands
085 	GMT Standard Time 	(GMT) Greenwich Mean Time: Dublin, Edinburgh, Lisbon, London
090 	Greenwich Standard Time 	(GMT) Casablanca, Monrovia
095 	Central Europe Standard Time 	(GMT+01:00) Belgrade, Bratislava, Budapest, Ljubljana, Prague
100 	Central European Standard Time 	(GMT+01:00) Sarajevo, Skopje, Warsaw, Zagreb
105 	Romance Standard Time 	(GMT+01:00) Brussels, Copenhagen, Madrid, Paris
110 	W. Europe Standard Time 	(GMT+01:00) Amsterdam, Berlin, Bern, Rome, Stockholm, Vienna
113 	W. Central Africa Standard Time 	(GMT+01:00) West Central Africa
115 	E. Europe Standard Time 	(GMT+02:00) Bucharest
120 	Egypt Standard Time 	(GMT+02:00) Cairo
125 	FLE Standard Time 	(GMT+02:00) Helsinki, Kiev, Riga, Sofia, Tallinn, Vilnius
130 	GTB Standard Time 	(GMT+02:00) Athens, Istanbul, Minsk
135 	Israel Standard Time 	(GMT+02:00) Jerusalem
140 	South Africa Standard Time 	(GMT+02:00) Harare, Pretoria
145 	Russian Standard Time 	(GMT+03:00) Moscow, St. Petersburg, Volgograd
150 	Arab Standard Time 	(GMT+03:00) Kuwait, Riyadh
155 	E. Africa Standard Time 	(GMT+03:00) Nairobi
158 	Arabic Standard Time 	(GMT+03:00) Baghdad
160 	Iran Standard Time 	(GMT+03:30) Tehran
165 	Arabian Standard Time 	(GMT+04:00) Abu Dhabi, Muscat
170 	Caucasus Standard Time 	(GMT+04:00) Baku, Tbilisi, Yerevan
175 	Transitional Islamic State of Afghanistan Standard Time 	(GMT+04:30) Kabul
180 	Ekaterinburg Standard Time 	(GMT+05:00) Ekaterinburg
185 	West Asia Standard Time 	(GMT+05:00) Islamabad, Karachi, Tashkent
190 	India Standard Time 	(GMT+05:30) Chennai, Kolkata, Mumbai, New Delhi
193 	Nepal Standard Time 	(GMT+05:45) Kathmandu
195 	Central Asia Standard Time 	(GMT+06:00) Astana, Dhaka
200 	Sri Lanka Standard Time 	(GMT+06:00) Sri Jayawardenepura
201 	N. Central Asia Standard Time 	(GMT+06:00) Almaty, Novosibirsk
203 	Myanmar Standard Time 	(GMT+06:30) Yangon Rangoon
205 	S.E. Asia Standard Time 	(GMT+07:00) Bangkok, Hanoi, Jakarta
207 	North Asia Standard Time 	(GMT+07:00) Krasnoyarsk
210 	China Standard Time 	(GMT+08:00) Beijing, Chongqing, Hong Kong SAR, Urumqi
215 	Singapore Standard Time 	(GMT+08:00) Kuala Lumpur, Singapore
220 	Taipei Standard Time 	(GMT+08:00) Taipei
225 	W. Australia Standard Time 	(GMT+08:00) Perth
227 	North Asia East Standard Time 	(GMT+08:00) Irkutsk, Ulaanbaatar
230 	Korea Standard Time 	(GMT+09:00) Seoul
235 	Tokyo Standard Time 	(GMT+09:00) Osaka, Sapporo, Tokyo
240 	Yakutsk Standard Time 	(GMT+09:00) Yakutsk
245 	A.U.S. Central Standard Time 	(GMT+09:30) Darwin
250 	Cen. Australia Standard Time 	(GMT+09:30) Adelaide
255 	A.U.S. Eastern Standard Time 	(GMT+10:00) Canberra, Melbourne, Sydney
260 	E. Australia Standard Time 	(GMT+10:00) Brisbane
265 	Tasmania Standard Time 	(GMT+10:00) Hobart
270 	Vladivostok Standard Time 	(GMT+10:00) Vladivostok
275 	West Pacific Standard Time 	(GMT+10:00) Guam, Port Moresby
280 	Central Pacific Standard Time 	(GMT+11:00) Magadan, Solomon Islands, New Caledonia
285 	Fiji Islands Standard Time 	(GMT+12:00) Fiji Islands, Kamchatka, Marshall Islands
290 	New Zealand Standard Time 	(GMT+12:00) Auckland, Wellington
300 	Tonga Standard Time 	(GMT+13:00) Nuku’alofa 
```