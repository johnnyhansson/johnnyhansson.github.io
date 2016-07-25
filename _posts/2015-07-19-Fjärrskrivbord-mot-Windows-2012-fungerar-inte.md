---
layout: post
title: Fjärrskrivbord mot Windows 2012 fungerar inte
---

Här kommer ett tips för att felsöka varför fjärrskrivbord inte fungerar mot Windows Server 2012.

En av de första sakerna jag brukar göra efter att ha installerat en ny maskin med Windows Server är att aktivera åtkomst till den via fjärrskrivbord (Remote Desktop). Den senate gången var inget undantag, men jag kunde ändå inte ansluta till den och Server Manager-verktyget indikerade att fjärrskrivbord var aktiverat.

I just detta fallet hade enbart brandväggsreglerna som tillät anslutning om man var ansluten via ett domännätverk aktiverats. Med andra ord så blockerade fortfarande Windows-brandväggen mina anslutningsförsök, men efter att aktiverat reglerna som gällde Public-profilen så fungerade fjärrskrivbord.
