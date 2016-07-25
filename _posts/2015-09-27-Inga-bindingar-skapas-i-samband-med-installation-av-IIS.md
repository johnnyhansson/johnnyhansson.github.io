---
layout: post
title: Inga bindingar skapas i samband med installation av IIS
---

Senast jag installerade IIS (IIS 10 på Windows 10) så skapades inga bindningar för HTTP och HTTPS. Detta medförde att jag inte kunde skapa någon ny site i IIS Manager. Jag saknade även standard-siten som installeras tillsammans med IIS.

Om du stöter på detta problem så testa följande steg.
<ol>
	<li>Avinstallera Windows Process Activation Service.</li>
	<li>Avinstallera IIS.</li>
	<li>Starta om datorn.</li>
	<li>Installera Windows Process Activation Service.</li>
	<li>Installera IIS.</li>
</ol>
