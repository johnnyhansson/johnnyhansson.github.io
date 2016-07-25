---
layout: post
title: Kom igång med Jenkins
---

I den här artikeln kommer jag att beskriva hur man kommer igång med att använda Jenkins tillsammans med .NET. Som exempel kommer jag att använda en enkel webbapplikation skriven i ASP.NET MVC 5 och i slutet av artikeln så visar jag även hur man kan publicera denna applikation via FTP.
<h2>Förberedelser</h2>
Innan du påbörjar artikeln så måste vissa program finnas installerade.
Jag förutsätter även att du har grundläggande kunskap i Git och har ett konto hos antingen Bitbucket eller Github.
<h5>Git</h5>
I de fall då man använder en bygg/CI-server så är det brukligt att konfigurera sina byggjobb att monitorera det SCM-system som man använder och vid en förändring starta något av de jobb som ska utföras.

Jag kommer att använda mig av Git i den här artikeln och därför behöver du ha Git installerat. Källkoden i projektet kommer även att hostas på Bitbucket.
<h5>JRE/JDK</h5>
Jenkins är skrivet i Java och för att kunna köra programvaran så behöver du ha en JRE eller JDK installerad. Nödvändiga miljövariabler måste även vara konfigurerade så att kommandoprompten kan hitta Java-kommandot.
<h2>Vad är Jenkins?</h2>
Enkelt beskrivet så är Jenkins en continous integration-server som ger oss möjlighet att automatisera olika typer av uppgifter som att exempelvis bygga våra projekt, köra enhetstester och publicera till test- eller produktionsmiljöer. Dessa typer av uppgifter utförs inte automagiskt utan utförs som en reaktion på en händelse som exempelvis att en utvecklare har checkat in sina ändringar till det SCM-system som Jenkins övervakar.
<h2>Starta Jenkins</h2>
En av många de fördelar som finns med Jenkins är att det är väldigt enkelt att komma igång med. Vill du bara prova Jenkins så behöver du inte ens installera programvaran på din maskin utan det räcker att starta den via kommandotolken. Ska du däremot använda den på en dedikerad byggserver så rekommenderar jag att installera den som en tjänst.

Gå till <a href="https://jenkins-ci.org">https://jenkins-ci.org</a> så finns till höger en rubrik som heter Download Jenkins. Klicka på länken som heter Latest and greatest för att ladda ner den WAR-fil som Jenkins består av.

<img src="/images/Get-Started-With-Jenkins/Download.png" alt="Download" width="331" height="154" />

Öppna en kommandotolk och gå sedan till den katalog där WAR-filen placerades. Skriv sedan följande kommando för att starta Jenkins. Observera att detta enbart kommer fungera om du har en JRE eller JDK installerad samt att nödvändiga miljövariabler är korrekt konfigurerade.

```bash
java -jar jenkins.war
```

Detta kommer starta igång Jenkins på port 8080. Öppna sedan en webbläsare och gå till http://localhost:8080. Skulle den porten av någon anledning vara upptagen på din maskin så kan man ändra den port som ska användas genom att ange växeln --httpPort=XYZ när man startar Jenkins.

```bash
java -jar jenkins.war --httpPort=1234
```

<h2>Installera tillägg</h2>
När du nu startat Jenkins och är inne i webbgränssnittet så möts du av en dashboard som visar dina projekt och deras status.

<img src="/images/Get-Started-With-Jenkins/Dashboard.png" alt="Dashboard" width="824" height="447" />

Innan vi kan gå vidare så måste vi installera några tillägg som behövs senare i artikeln. Tillsammans med Jenkins så kommer några få tillägg förinstallerade, men det finns en stor katalog över tillägg som går att installera i efterhand som gör det möjligt att utöka antalet funktioner som finns tillgängliga.

Klicka på Manage Jenkins följt av Manage Plugins. Välj sedan fliken Available och markera följande tillägg.
<ul>
	<li>GIT plugin</li>
	<li>MSBuild Plugin</li>
	<li>Publish Over FTP</li>
</ul>
Klicka sedan på knappen Install without restart.
<h2>Skapa ett projekt</h2>
För att Jenkins ska ha något att bygga till oss så ska vi sätta upp ett enkelt projekt i Visual Studio. Skapa ett nytt MVC-projekt och döp det till HelloWorldWeb. Klicka sedan på OK.

<img src="/images/Get-Started-With-Jenkins/NewProject.png" alt="NewProject" width="955" height="660" />

Välj mallen Empty och markera MVC. Klicka på OK.

<img src="/images/Get-Started-With-Jenkins/SelectTemplate.png" alt="SelectTemplate" width="770" height="575" />

Skapa en ny controller och döp denna till HomeController.

```csharp
using System;
using System.Collections.Generic;
using System.Configuration;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace HelloWorldWeb.Controllers
{
    public class HomeController : Controller
    {
        public ActionResult Index()
        {
            return View();
        }
    }
}
```

Skapa en vy för den Index-action som finns i HomeController.

```html
@{
    ViewBag.Title = "Index";
}

<h1>MVC Application Deployed with Jenkins!</h1>
```

<h2>Skapa en publiceringsprofil för projektet</h2>
Innan vi kan lämna Visual Studio och påbörja konfigurationen av Jenkins så ska vi skapa en publiceringsprofil för projektet. Denna kommer vi att använda oss av när Jenkins bygger projektet.

Högerklicka på projektet i Visual Studio och välj Publish. Välj sedan Custom.

<img class="alignnone size-full wp-image-65" src="/images/Get-Started-With-Jenkins/PublishTarget.png" alt="PublishTarget" width="720" height="565" />

Namnge sedan profilen och klicka på Next.

<img src="/images/Get-Started-With-Jenkins/PublishProfileName.png" alt="PublishProfileName" width="354" height="149" />

Välj File System som Publish method och ange till vilket katalog som publicering ska ske. När Jenkins genomför publiceringen så kommer vi att skriva över det angivna värdet med sökvägen till en katalog som kommer placeras i arbetskatalogen för Jenkins-projektet. Klicka sedan på Next.

<img src="/images/Get-Started-With-Jenkins/PublishTargetType.png" alt="PublishTargetType" width="720" height="565" />

Markera vilken byggkonfiguration som ska användas vid publiceringen. Som standard är Release angiven. Markera även Delete all existing files prior to publish. Klicka på Next.

<img src="/images/Get-Started-With-Jenkins/PublishSettings.png" alt="PublishSettings" width="720" height="565" />

Klicka på Close i sista steget och välj att du vill spara ändringarna som du gjort.

Gör en commit på alla ändringar och pusha sedan dessa till Bitbucket/Github.
<h2>Konfigurera tillägget MSBuild</h2>
Innan vi kan skapa ett nytt projekt i Jenkins så behöver tillägget MSBuild konfigureras. Gå till Manage Jenkins och sedan Configure System. När tillägget installerades så tillkom även ett nytt val som heter MSBuild. Klicka på
MSBuild installations... och sedan Add MSBuild.

Ange ett namn och sedan sökvägen där Jenkins kan hitta MS Build. Spara sedan konfigurationen genom att klicka på Save.

<img src="/images/Get-Started-With-Jenkins/AddMsBuild.png" alt="AddMsBuild" width="1160" height="291" />
<h2>Skapa ett byggprojekt</h2>
Nu är det dags att skapa ett nytt byggprojekt. Detta kommer innehålla konfiguration för var Jenkins kan hitta källkoden till projektet, hur och hur ofta Jenkins ska kontrollera om förändringar inträffat osv.

Klicka på New Item i huvudmenyn och ange ett projektnamn. Som projekttyp väljer du Freestyle project och klickar sedan på OK vilket tar dig till konfigurationen för projektet.

<img src="/images/Get-Started-With-Jenkins/CreateProject.png" alt="CreateProject" width="1131" height="374" />

Markera valet Git under sektionen Source Code Management och skriv in URL:en till repositoryn.

<img src="/images/Get-Started-With-Jenkins/GitRepository.png" alt="GitRepository" width="836" height="270" />Klicka på knappen Add för att lägga till de uppgifter som ska användas för att komma åt repositoryn.

Bläddra ner till sektionen Build Triggers och markera valet Poll SCM. I fältet Schedule så fyller du in fem stycket *-tecken. Det vi konfigurerar här är när Jenkins ska påbörja ett nytt bygge av projektet och vi har angivit att Jenkins varje minut ska kontrollera om några förändringar har gjorts i den repository som övervakas. Om du inte känner igen syntaxen sedan tidigare så är den av samma sort som används för CRON-job i exempelvis Linux-system.

<img src="/images/Get-Started-With-Jenkins/BuildTrigger.png" alt="BuildTrigger" width="386" height="216" />

Det sista vi behöver göra innan vi kan testa vår konfiguration är att lägga till en uppgift som ska utföras när ett bygge startas. Antalet uppgifter som ska utföras kan vara en eller flera och innefatta allt från att bygga projektet, köra olika typer av tester eller publicera projektet till en server.

Välj Build a Visual Studio project or solution using MSBuild som visas när du klickar på knappen Add build step.

<img src="/images/Get-Started-With-Jenkins/BuildStepMsBuild.png" alt="BuildStepMsBuild" width="525" height="235" />

Välj sedan den MS Build-version som du tidigare konfigurerade och ange namnet på den solution eller det projekt som ska skickas som parameter till MS Build. Jenkins kommer automatiskt att sätta aktuell katalog projektets workspace-katalog eller roten för Git-repositoryn när MS Build körs. Varje projekt har ett eget workspace och är den katalog dit Jenkins kommer att klona Git-repositoryn när den startar ett nytt bygge.

Som parametrar till MS Build så anger vi även att den ska använda den publiceringsprofil som vi satt upp för projektet och till vilken katalog som publiceringen ska ske. Anledningen till att vi publicerar till en lokal katalog är för att underlätta senare då vi ska ladda upp resultatet av bygget via FTP.

<img src="/images/Get-Started-With-Jenkins/BuildStepMsBuildConfiguration.png" alt="BuildStepMsBuildConfiguration" width="730" height="216" />

Klicka nu på Save för att spara alla inställningar för Jenkins-projektet.
<h2>Testa konfigurationen</h2>
Nu när vi gjort de grundläggande inställningarna så vill vi såklart kontrollera så att allt fungerar som det ska. Gå tillbaka till dashboarden och klicka på ikonen med play-symbolen och klockan som syns till höger om projektet för att starta ett nytt bygge. När bygget startar så kommer Jenkins att göra en lokal klon av Git-repositoryn och bygga vårt MVC-projekt som vi skapade tidigare. Om alla inställningar är korrekta så kommer en blå ikon att visas och till höger om denna en stor sol. Skulle något däremot inte fungera så håll muspekaren över namnet på projektet och klicka på den svarta pilen som visas. Klicka sedan på Configure för att ändra eventuella inställningar.
<h2>Publicera projektet via FTP</h2>
Nu när Jenkins kan bygga vårt projekt så vill vi publicera resultatet via FTP till exempelvis något webbhotell. Tidigare installerade vi tillägget Publish Over FTP som vi nu ska dra nytta av, men innan vi kan göra detta så måste konfigurera tillägget.

Gå till Manage Jenkins och sedan Configure System. Bläddra sedan ner till sektionen Publish Over FTP och klicka på Add för att lägga till en ny FTP-server. Namnge FTP-konfigurationen tillsammans med adressen till servern och vilka användaruppgifter som ska användas när publiceringen sker. Avsluta med att klicka på Test Configuration. Fungerar allt så sparar du konfigurationen.

Gå tillbaka till konfigurationen för projektet och bläddra ner till sektionen
Post-build Actions. Klicka på Add post-build action och välj Send build artifacts over FTP.

<img src="/images/Get-Started-With-Jenkins/PostBuildStepFtp.png" alt="PostBuildStepFtp" width="315" height="268" />

Välj konfigurationen för FTP-servern som du skapade tidigare och ange vilken katalog där filerna som ska laddas upp ligger. Alternativet Remove prefix anger den del i sökvägen som inte ska skapas på den FTP-server som vi laddar upp filerna till. Anger vi exempelvis att filerna som ska laddas upp ligger i katalogen output och vi inte till att den katalogen inte ska skapas när filerna laddas upp så anger vi output i Remove prefix. Remote directory anger sökvägen på

FTP-servern där de uppladdade filerna ska placeras.

<img src="/images/Get-Started-With-Jenkins/FtpConfiguration.png" alt="FtpConfiguration" width="733" height="280" />

Spara nu konfigurationen för projektet och testa sedan att denna fungerar.
