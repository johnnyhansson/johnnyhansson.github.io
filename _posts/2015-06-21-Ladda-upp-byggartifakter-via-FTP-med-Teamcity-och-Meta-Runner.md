---
layout: post
title: Ladda upp byggartifakter via FTP med Teamcity och Meta-Runner
---

Ibland uppstår behovet att publicera sina byggartifakter via FTP och då kommer här ett tips för er som använder Teamcity 8 eller nyare.

Från och med Teamcity 8 så finns en ny funktion inbyggd som heter
Meta-Runner. Vad denna funktion möjliggör är att bryta ut viss information från en byggkonfiguration och skapa en build runner av dem som sedan kan återanvändas i andra byggkonfigurationer. Detta går att göra direkt via webbgränssnittet i Teamcity. Från och med version 9 har Jetbrains även lagt in funktionen att kunna installera Meta-Runners direkt via webbgränssnittet.

Hur kommer då publicering via FTP in i bilden? På GitHub återfinns något som heter <a href="https://github.com/jetbrains/meta-runner-power-pack" target="_blank">Meta-Runners Power Pack</a> vilket är en samling av Meta-Runners som går att installera i sina projekt för att utföra olika typer av uppgifter. En av dessa är ftp-upload som möjliggör uppladdning av filer via FTP. Innan jag gick över till att använda ftp-upload så provade jag pluginen Deployer som finns till Teamcity.

Mina erfarenheter av ftp-upload är att jag upplever denna mer stabil än Deployer samtidigt som överföringen sker väsentligt snabbare. Står valet mellan att använda Deployer eller ftp-upload så skulle jag välja den senare.

Är du intresserad av att veta mer om Meta-Runners så finns det en del information i <a href="https://confluence.jetbrains.com/display/TCD9/Working+with+Meta-Runner" target="_blank">dokumentationen</a> för Teamcity.
