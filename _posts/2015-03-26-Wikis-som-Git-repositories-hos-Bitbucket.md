---
layout: post
title: Wikis som Git-repositories hos Bitbucket
---

Sedan några år tillbaka så använder jag Git för att versionshantera källkoden för både privata och arbetsrelaterade projekt. Till skillnad från många andra så använder jag [Bitbucket](https://bitbucket.org) istället för Github med anledning av att jag vill ha tillgång till privata repositories utan att behöva betala för detta.

Något som jag däremot missat fram till idag är att den wiki som man kan aktivera för varje repository som man har hos Bitbucket ligger i en egen repository. Detta medför att man kan, precis som med andra Git-repositories, göra en clone på denna, genomföra sina ändringar och sedan pusha upp dem till Bitbucket för att få dessa publicerade. Dessutom får man tillgång till alla de fördelar som versionshantering medför.

Vill man inte göra en clone och genomföra alla ändringar lokalt så går det självklart lika bra att göra alla ändringar direkt via webbgränssnittet hos Bitbucket.
