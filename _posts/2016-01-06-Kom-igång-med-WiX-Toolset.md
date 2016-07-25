---
layout: post
title: Kom igång med WiX Toolset
---

Denna guide är tänkt att fungera som ett underlag för den som inte har någon tidigare erfarenhet av WiX, men som är intresserad av lära sig några av grunderna. Texten är däremot inte heltäckande, men i slutet av den så finns referenser vart man kan vända sig för att lära sig mer.

WiX, Windows Installer XML, Toolset är en samling av verktyg för att skapa Windows Installer-paket med hjälp av XML och kan ses som ett lager ovanpå Windows Installer-plattformen.
Projektet startades ursprungligen av Rob Mensching, tidigare anställd hos Microsoft, och den första produkten där WiX användes var Office 2000. WiX har sedan dess använts för en rad olika Microsoft-produkter som Office, Visual Studio och Microsoft SQL Server, men även andra företag drar nytta av WiX. 2004 släpptes WiX publikt och då som Microsofts första Open Source-projekt.
Sedan pågår utvecklingen av projektet fortlöpande med Rob Mensching som fortsatt ledare.

I och med att WiX riktar sig mot Windows Installer-plattformen så får man även tillgång till de fördelar som den bidrar med. Exempelvis så kan en installerad produkt repareras ifall den skulle bli skadad och en transaktionsbaserad installation ger möjlighet rulla tillbaka alla ändringar om något skulle gå fel eller användaren avbryter installationen.

En av många andra fördelar med att använda sig av WiX är att hela installationspaketet som man skapar fram definieras med hjälp av XML. I och med detta så är det även enkelt att versionshantera installationsprojektet tillsammans med övrig kod.

Om du tror att WiX och Windows Installer-plattformen endast går att använda för enklare typer av installationer så behöver du tänka om. Förutom vanliga aktiviteter som att installera filer, skapa genvägar och registrera tjänster så tillhandahåller WiX en mängd tilläggsaktiviteter som innefattar exempelvis att man kan köra SQL-skript, skapa siter i en IIS osv. Det är även fullt möjligt att skapa sina egna tillägg som går att använda i sina installationsprojekt.
<h2>Installera WiX</h2>
&nbsp;

På projektets hemsida, <a href="http://wixtoolset.org/">http://wixtoolset.org/</a>, återfinns länkar till den senaste versionen av WiX. Rekommenderat är att ladda ner EXE-filen för installationen. Denna installerar, förutom verktygen som WiX består av, en integrationen för Visual Studio som ger tillgång till en rad olika projektmallar och Intellisense-stöd.

När själva installationen av WiX är klar så har du tillgång till en rad verktyg som går att köra direkt från kommandoprompten. Detta är en fördel om man vill integrera WiX tillsammans med en byggserver. Rekommenderat är att lägga sökvägen till bin-katalogen, där verktygen ligger, i miljövariabeln PATH så att man alltid har tillgång till dem. De verktyg som installeras är följande
<ul>
	<li style="margin-left: 50px;">Candle</li>
	<li style="margin-left: 50px;">Light</li>
	<li style="margin-left: 50px;">Burn</li>
	<li style="margin-left: 50px;">Heat</li>
	<li style="margin-left: 50px;">Dark</li>
</ul>
Candle och Light kan ses som en klassisk kompilator och länkare där Light kommer att producera det slutgiltiga installationspaketet.

Som vi redan vet så definierar vi vårt installationspaket med hjälp av XML-filer. Dessa har inte ändelsen .xml utan istället .wxs. Dessa WXS-filer kan vi sedan skicka med som argument till en kompilator som följer med WiX som heter Candle.

```bash
candle.exe Product.wxs
```

Det Candle gör är att den tar WXS-filerna och översätter dessa till ett mellanformat som också är uppbyggt med hjälp av XML. Dessa filer får ändelsen .wixobj och kan sedan skickas som argument till verktyget Light för att skapa vårt installationspaket, Product.msi.

```bash
light.exe Product.wixobj
```

Det denna MSI-fil innehåller är den installationsdatabas som beskriver vad vi förväntar oss ska installeras. Hur själva installationen ska genomföras bestäms av Windows Installer-plattformen.
Vi behöver alltså inte berätta hur Windows Installer-plattformen ska utföra sitt arbete genom att kopiera filerna, skapa genvägar och installera tjänster. Bara vad vi förväntar oss att slutresultatet ska vara.

Under processen då installationspaketet, MSI-filen, skapas så genomförs även en rad kontroller för att säkerställa att alla regler för installationspaketet uppfylls. Detta är något som benämns som ICE, Internal Consistency Evaluator.

Använder man de projektmallar som finns i Visual Studio så behöver man inte själv anropa Candle och Light via kommandoprompten utan detta sker automatiskt i samband med att man bygger projektet.
<h2>Skapa ditt första WiX-projekt</h2>
I det här exemplet så ska vi skapa ett installationspaket för en exempelapplikation bestående av en konsolapplikation som skriver ut texten Hello World. Språket som kommer att användas är C# och  Visual Studio 2015.

Öppna Visual Studio och gå till File - New - Project (Ctrl + Shift + N). Välj Templates – Visual C# - Console Application.

<img src="/images/Get-Started-With-WiX-Toolset/NewConsoleApplication.png" alt="NewConsoleApplication" width="660" height="458" />

Klistra in följande kod i Program.cs som skapats i konsolprojektet.

```csharp
using System;

namespace ExampleApplication
{
   class Program
   {
      static void Main(string[] args)
      {
         Console.WriteLine("Hello World!");
         Console.ReadLine();
      }
   }
}
```

Högerklicka på Solution-noden och välj Add – New Project… Den här gången så väljer du Templates – Windows Installer XML – Setup Project.

<img src="/images/Get-Started-With-WiX-Toolset/NewSetupProject.png" alt="NewSetupProject" width="660" height="458" />

När projektet skapats så återfinns en fil som heter Product.wxs med följande innehåll.

```xml
<?xml version="1.0" encoding="UTF-8"?>
    <Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
       <Product Id="*" Name="Setup" Language="1033" Version="1.0.0.0" Manufacturer="" UpgradeCode="5bc57152-0a64-4d42-b709-e1239f3575f6">
          <Package InstallerVersion="200" Compressed="yes" InstallScope="perMachine" />
          <MajorUpgrade DowngradeErrorMessage="A newer version of [ProductName] is already installed." />
          <MediaTemplate />
          <Feature Id="ProductFeature" Title="Setup" Level="1">
             <ComponentGroupRef Id="ProductComponents" />
          </Feature>
       </Product>
       <Fragment>
          <Directory Id="TARGETDIR" Name="SourceDir">
             <Directory Id="ProgramFilesFolder">
                <Directory Id="INSTALLFOLDER" Name="Setup" />
             </Directory>
          </Directory>
       </Fragment>
       <Fragment>
          <ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
             <!-- TODO: Remove the comments around this Component element and the ComponentRef below in order to add resources to this installer. -->
             <!--     <Component Id="ProductComponent"> -->
             <!-- TODO: Insert files, registry keys, and other resources here. -->
             <!--     </Component> -->
          </ComponentGroup>
       </Fragment>
    </Wix>
```

Försöker man bygga projektet så kommer detta att resultera i ett byggfel som lyder ”The Product/@Manufacturer attribute's value cannot be an empty string. If a value is not required, simply remove the entire attribute.”. Med andra ord kan man inte lämna Manufacturer-attributet på Product-elementet tomt. Tillsammans med Manufacturer så krävs attributen Id, Name, Language och Version. Detta är inte ett krav från WiX utan från Windows Installer-plattformen.

Som för många andra element används GUID-värden som Id för att på så sätt vara garanterat unika.  Istället för att ange Id-värdet, benämns ProductCode, manuellt för Product-elementet så kan man låta WiX automatiskt generera ett nytt varje gång projektet byggs genom att ange en asterisk istället. Detta fungerar däremot endast för Id-attributet på Product-elementet. Produktkoden kommer Windows Installer att använda sig av för att kontrollera om produkten redan är installerad på datorn

Name anger namnet för installationen, Language är den språk-kod som anger det språk som används för installationspaketet och Version anger produktversionen. Trots att fyra nummergrupper ör angivna så används endast de tre första.

Förutom dessa fem attribut så är det även rekommenderat att man även anger UpgradeCode. Detta för att möjliggöra eventuella framtida uppgraderingar av produkten. Till skillnad från Id-attributet så ska detta värde aldrig förändras utan ska bestå över alla versioner av produkten.

Förutom Product-elementet så innehåller WXS-filen elementen Fragement, Directory, Feature och ComponentGroup. Mer om dessa element i nästkommande del av guiden.
<h2>Grundläggande element</h2>
<h3>Package</h3>
Inuti Product-elementet så återfinns Package-elementet. Till skillnad från Product så beskriver Package-elementet själva installationen som sådan och innehåller all information som Windows Installer-plattformen behöver för att kunna utföra en (av)installation.

```xml
<Product ...>
    <Package Id="*" InstallerVersion="200" Compressed="yes" InstallScope="perMachine" Description="Demo application for creating a MSI-package using WiX." Manufacturer="Acme Corp"/>
</Product>
```

Id-attributet på Package-elementet kan utelämnas eller så kan man ange en asterisk för att låta WiX generera ett GUID-värde då man bygger installationsprojektet.

InstallerVersion låter dig ange den version av Windows Installer-plattformen som krävs för installationen. Genom att multiplicera versionsnumret som du önskar med 100 så får du det värde som ska anges för attributet. Skulle den önskade versionen saknas så kommer en dialog att visas för användaren som rekommenderar denne att uppgradera.
<h3>Media/MediaTemplate</h3>
Nästlat inuti Package så återfinns Media-elementet där vi kan definiera vilket medium vi vill installera ifrån. Idag är det ganska ovanligt att behöva dela upp en installation över flera olika media som exempelvis två CD-skivor, men möjligheten tillhandahålls och det går även att ange vilket media som olika filer och/eller komponenter ska placeras på.

```xml
<Product ...>
    <Package ...>
      <Media Id="1" DiskPrompt="#1" EmbedCab="no" Cabinet="Disk1.cab" />
      <Media Id="2" DiskPrompt="#2" EmbedCab="yes" Cabinet="Disk2.cab" />
      <Property Id='DiskPrompt' Value="Insert disk [1]" />
    </Package>
</Product>
```

Med EmbedCab kan vi ange ifall en CAB-fil ska bäddas in i MSI-filen eller om den ska vara fristående. DiskPrompt-attributet gör det möjligt att visa en text under installationen som anger vilket media som ska matas in för att fortsätta installationen.

Sedan WiX 3.6 så finns ett nytt element, MediaTemplate, som är tänkt att förenkla hanteringen av CAB-filer. Är du inte i behov av all den kontroll som Media-elementet ger så bör du istället använda MediaTemplate.
<h3>Directory</h3>
Detta element tillåter oss att ange den katalogstruktur som ska återskapas när själva produkten installeras. Med andra ord är det fullt möjligt, och även det normala förfarandet, att man nästlar Directory-element i flera nivåer.

```xml
<Product ...>
   <Package ...>
      <Directory Id="TARGETDIR" Name="SourceDir">
        <Directory Id="ProgramFilesFolder">
          <Directory Id="Demoware" Name="Demoware">
            <Directory Id="INSTALLFOLDER" Name="Demo.Setup" />
          </Directory>
        </Directory>
      </Directory>
  </Package>
</Product>
```

Ytterst så anger vi källkatalogen för hela installationen. Denna har ett fördefinierat Id som måste vara TARGETDIR.

Sedan följer själva målstrukturen för installationen och konventionerna är att produkten ska placeras enligt följande katalogstruktur Program Files\Företagsnamn\Produktnamn.

För standardkataloger, som Program Files, så finns det vissa egenskaper (property) som går att använda som Id:en på Directory-elementen. Värdet för dessa sätts upp automatiskt och vi behöver alltså inte själva ange sökvägen till Program Files-katalogen och inte heller något namn med Name-attributet.

Vill vi referera till Program Files (x86) så använder man ProgramFilesFolder som Id annars ProgramFiles64Folder. Det finns flera fördefinierade egenskaper och en länk till dessa återfinns i referensdelen av artikeln [4].

Längst in i strukturen anger vi målkatalogen som ska fungera som bas för vår produkt. Id:et som sätts kommer även att bli tillgängligt som en property vilket sen går att referera till i sitt installationspaket. Här sätter vi rotkatalogen för installationen till INSTALLFOLDER som Id. Förutom att det är möjligt att definiera sina egna properties så tillhandahålls flertalet olika fördefinierade sådana. Även en länk till listan över dessa återfinns i referensdelen av artikeln [3].
<h3>Component</h3>
Nu när vi har vår katalogstruktur för installationen definierad så är det läge att ange vad som ska installeras. En komponent är en sammanhängande enhet och kan innehålla allt ifrån en fil, en genväg eller något helt annat som exempelvis installation av en Windows-tjänst. Huvudsaken är att dessa hör ihop och ska installeras tillsammans som en sammanhängande enhet. Värt att notera är att komponenter inte ska dela filer vilket annars kan orsaka att borttagning av en komponent skulle påverka en helt annan komponent. Som standard brukar man använda sig av regeln ”en fil per komponent”. Anledningen till detta är att endast en fil kan dekoreras med attributet KeyPath i en komponent. Sätts KeyPath till ”yes” för en fil så kommer detta att aktivera möjligheten att reparera filen ifall den skulle bli skadad.

Precis som andra element så behöver en komponent ett Id samt en GUID.
Denna information kommer lagras så att Windows Installer exempelvis kan utföra en reparation av en installation om användaren begär detta eller så att den vet vilka filer som ska tas bort under en avinstallation av produkten.
<h3>File</h3>
När vi ändå är inne på att skapa komponenter så kan det vara intressant att veta hur man associerar en fil med en komponent. Filer är även en av de vanligaste delarna som ett installationspaket kommer att bestå av. En fil tilldelas till en komponent genom att man skapar ett File-element som underelement till Component-elementet. Varje fil måste tilldelas ett id och Source-attributet anger den absoluta eller relativa sökvägen till filen så att WiX kan hitta den för inbäddning i ett kabinett.

En av de vanligaste uppgifterna är att man vill referera till en fil som återfinns i ett projekt i sin solution. För att på ett enkelt sätt referera till dessa filer från sitt installationsprojekt så lägger man till en referens till projektet där filerna finns. I samband med att detta görs så skapas det även upp vissa pre-processor direktiv som kan användas från installationsprojektet. Dessa direktiv har formatet $(var.ProjektNamn.NamnPåDirektiv). Har man skapat en referens till ett projekt med namnet MySoftware och behöver sökvägen till katalogen där Visual Studio placerat alla byggartifakter för projektet använder man TargetDir, $(var.MySoftware.TargetDir). En lista över dessa direktiv återfinns i referensdelen av artikeln [5].

```xml
<Component Id="MySoftware" Guid="DC1EEDA9-94DD-4D35-8CCE-19F289451AA0" Directory="INSTALLFOLDER">
  <File Id="MySoftwareExe" Source="$(var.MySoftware.TargetPath)" KeyPath="yes" />
</Component>
```

Vill man att filen får ett annat namn på målsystemet i samband med installationen kan man dra nytta av attributet Name. KeyPath som vi tidigare nämnt kan anges på File-elementet för att möjliggöra reparation av filen om installationen skulle skadas. Utelämnar man detta attribut så kommer den första filen i en komponent automatiskt tilldelas detta. Det är även möjligt att ange ifall en installation ska fortsätta eller inte genom om filen inte skulle gå att placera på målsystemet genom attributet Vital.
<h3>ComponentGroup</h3>
Med ComponentGroup-elementet så är det möjligt att gruppera ihop flera komponenter. Detta är smidigt då man vid behov kan referera till en grupp med komponenter istället för att referera till varje enskild komponent. Detta görs då genom att använda ComponentGroupRef-elementet och ange Id:et till den komponentgrupp som man vill referera till. ComponentGroup-elementet låter dig även ange Directory-attributet på själva gruppen istället för på varje enskild komponent, vilket minskar behovet av att upprepa sig.

```xml
<ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
  <Component Id="ProductComponent" Guid="C4156160-6AB9-4092-8DA1-1B37C72D7CBB">
    <File Id="DemoAppExe" Source="$(var.Demo.App.TargetPath)" KeyPath="yes" DiskId="1" />
    <File Id="DemoAppExeConfig" Source="$(var.Demo.App.TargetDir)\Demo.App.Exe.config" DiskId="2" />
  </Component>
</ComponentGroup>

...

<Feature Id="ProductFeature" Title="Demo.Setup" Level="1">
  <ComponentGroupRef Id="ProductComponents" />
</Feature>
```

<h3>Feature</h3>
Feature-elementet i installationspaketet beskriver de olika delarna av en produkt som användaren har möjlighet att installera. När du installerat någon form av applikation känner du säkerligen igen att du fått välja om du exempelvis vill inkludera dokumentationen för applikationen eller vilka insticksmoduler som ska installeras. Detta är exempel på uppdelning som man kan göra med Feature-elementet.

I en installation så måste det finnas minst ett Feature-element och inuti detta så anger man de komponenter eller den grupp av komponenter som ska ingå. Alla komponenter i ett installationspaket måste ingå i en feature. Feature-element går även att nästla för att skapa undergrupper.

Varje feature måste tilldelas ett id och kan även ges en titel, Title, samt beskrivning, Description, som kan visas under själva installationen om ett grafiskt gränssnitt tillhandahålls. Vill man förhindra att användaren väljer att avmarkera installationen av en specifik feature så kan man göra detta genom att sätta attributet Absent till ”disallow”. För att dölja valet ”Feature will be installed when required” för en feature så anger man attributet AllowAdvertised till ”no”.

Komponenterna som ska ingå i en feature går att specificera direkt under elementet, men föredra istället att använda ComponentGroupRef för att inkludera en hel grupp av komponenter. Detta kommer underlätta arbetet med själva installationspaketet.

```xml
<Feature Id="ProductFeature" Title="Demo.Setup" Level="1" Absent="no" AllowAdvertised="yes">
  <ComponentGroupRef Id="ProductComponents" />
</Feature>
```

<h3>Fragement</h3>
När man skapar ett nytt installationsprojekt så hamnar alla olika element i en och samma fil, nämligen Product.wxs. Har man en liten produkt med få features och komponenter så är det normalt inga problem med detta upplägg. Har man däremot en större produkt så kan det vara värt att dela upp installationspaketet i mindre och mer hanterbara delar. Det är här Fragement-elementet kommer in.

Detta element kan ses som en generisk container där andra typer av element kan placeras och låter dig dela upp installationspaketet i olika delar. Något som kommer att underlätta hanteringen av installationsprojektet. Normalt så brukar man gruppera ihop element och koncept som hör ihop och placera dessa i ett och samma fragment. Med andra ord så kan man placera alla kataloger i ett fragment, alla komponenter i ett annat osv. Fragmenten placeras också i sina egna filer såsom Directories.wxs, Components.wxs etc.

För att skapa en ny wxs-fil med ett Fragement-element i så högerklickar man på sitt installationsprojekt, väljer Add och sedan New Item… Markera sedan valet Installer File och ge den nya filen ett passande namn, exempelvis Directories.wxs.

<img src="/images/Get-Started-With-WiX-Toolset/NewFragement.png" alt="NewFragement" width="660" height="458" />

När den nya filen skapats så placerar man de element som man vill flytta inuti Fragment-elementet. I exemplet nedan så visas hur katalogstrukturen för installationspaketet flyttats till sin egen fil. Trots att deklarationerna flyttats utanför Product.wxs så kommer projektet att bygga normalt.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
  <Fragment>
    <Directory Id="TARGETDIR" Name="SourceDir">
      <Directory Id="ProgramFilesFolder">
        <Directory Id="INSTALLFOLDER" Name="My Software" />
      </Directory>
    </Directory>
</Fragment>
</Wix>
```

<h3>Property</h3>
I installationen så går det att definiera Property-element som fungerar som variabler, vilket senare går att använda i sitt installationsprojekt. Vi kommer framöver att använda ordet egenskap istället för property. Värdet för dessa egenskaper är alltid en sträng och en egenskap anses vara definierad om den har ett värde.

En egenskap tilldelas ett Id. Detta id blir också det namn som man kan referera till när den ska användas i installationsprojektet. En egenskap med ett id som endast innehåller versaler blir publik och åtkomlig under hela installationen. I annat fall blir egenskapen privat och åtkomsten blir begränsad. Värdet på en egenskap kan loggas under själva installationen och det är därför inte rekommenderat att placera känslig data i en publik egenskap.

Förutom att det är möjligt att definiera sina egna egenskaper så finns det även många fördefinierade. En länk som listar många av de egenskaper som finns går att hitta under referenssektionen [2].

Ett av användningsområdena för egenskaper är tillsammans med Condition-elementet som tas upp härnäst.
<h3>Condition</h3>
Ett Condition-element definierar ett uttryck som antingen kan vara sant eller falskt och därvid bestämmer ifall en viss åtgärd ska utföras. Dessa uttryck sätts ofta upp med hjälp av egenskaper som man antingen själv definierat eller som finns fördefinerade.

Condition-elementet kan användas tillsammans med olika typer av andra element som Feature och Component. I fallet då dessa används tillsammans med Feature-elementet så är det möjligt att välja om en feature ska visas eller inte. Andra sammanhang då dessa går att använda är för Launch Conditions och UI-kontroller, men det är inget som vi kommer att ta upp i denna guide.

Uttrycken som sätts upp innanför Condition-elementet definieras inom ett CDATA-block och dessa uttryck går även kombinera med olika typer av operatorer.
<ul>
	<li style="margin-left: 50px;">=</li>
	<li style="margin-left: 50px;">&lt;&gt;</li>
	<li style="margin-left: 50px;">&gt;</li>
	<li style="margin-left: 50px;">&lt;</li>
	<li style="margin-left: 50px;">&gt;=</li>
	<li style="margin-left: 50px;">&lt;=</li>
	<li style="margin-left: 50px;">&gt;&lt;</li>
	<li style="margin-left: 50px;">&lt;&lt;</li>
	<li style="margin-left: 50px;">&gt;&gt;</li>
	<li style="margin-left: 50px;">AND</li>
	<li style="margin-left: 50px;">OR</li>
	<li style="margin-left: 50px;">XOR</li>
	<li style="margin-left: 50px;">EQV</li>
	<li style="margin-left: 50px;">IMP</li>
	<li style="margin-left: 50px;">NOT</li>
</ul>
Vill man exempelvis sätta upp en Launch Condition som säkerställer att användaren är administratör när installationen körs så kan man använda följande Condition-element som underelement till Product-elementet.

```xml
<Condition Message="You must run the installation as an administrator!">
  <![CDATA[Installed OR Privileged]]>
</Condition>
```

Skulle uttrycket vara falskt så kommer ett meddelande att visas för användaren när installationen körs och denna avbryts. Installed är en fördefinierad egenskap som är sann om produkten redan är installerad. Denna kan anges om man inte vill riskera att ett visst Condition-element ställer till problem vid exempelvis en avinstallation. Om Installed skulle vara sann så kommer hela uttrycket att bli sant i vårt exempel då vi använder oss av operatorn OR.
<h2>Användargränssnitt</h2>
Om vi skulle bygga ett installationsprogram med de element som nämnts ovan så hade inte användaren som startat installationen fått några val. Istället hade installationen fortlöpt automatiskt.
Windows Installer-plattformen innehåller inga fördefinierade dialoger som vi kan använda oss av, men istället för att bygga vårt egna gränssnitt så ska vi använda oss av de dialoguppsättningar som tillhandahålls av WiX.

För att möjliggöra detta så behöver man utföra två saker.
<ol>
	<li>Lägga till en referens i installationsprojektet till WixUIExtensions.dll. Denna återfinns i bin-katalogen där du installerade WiX och innehåller de uppsättningar av dialoger som vi vill använda.</li>
</ol>
<img src="/images/Get-Started-With-WiX-Toolset/AddReference.png" alt="AddReference" width="660" height="539" />

Högerklicka på References i installationsprojektet och välj Add Reference…

Bin-katalogen kommer automatiskt vara vald när dialogen visas så det räcker att bläddra ner till WixUIExtensions.dll och dubbelklicka på den. Klicka sedan på OK.
<ol>
	<li>Inuti Product-elementet lägga till ett UIRef-elementet med Id-attributet satt till den dialoguppsättning som ska användas. WiX tillhandahåller inte enbart enskilda dialoger som vi kan använda utan även hela uppsättningar av fördefinierade installationsflöden.</li>
</ol>
De dialoguppsättningar som finns har följande namn.
<ul>
	<li style="margin-left: 50px;">WixUI_Minimal</li>
	<li style="margin-left: 50px;">WixUI_Advanced</li>
	<li style="margin-left: 50px;">WixUI_FeatureTree</li>
	<li style="margin-left: 50px;">WixUI_InstallDir</li>
	<li style="margin-left: 50px;">WixUI_Mondo</li>
</ul>
Många av egenskaperna för dialoguppsättningarna (texter för licensavtal, bilder osv.) går att anpassa genom att definiera WixVariable-element i installationsprojektet. Instruktioner om hur man gör detta återfinns i referensdelen av artikeln (1).

```xml
<UIRef Id="WixUI_Mondo" />
```

Om du nu bygger installationsprojektet och startar installationen så kommer du att mötas av ett grafiskt gränssnitt där stegen varierar beroende på vilken uppsättning av dialoger som du angav i UIRef-elementet.
<h2>Exempelkod</h2>
Jag har satt ihop ett enkelt WiX-projekt i Visual Studio 2015 som visar hur man använder sig av features, genvägar, ikoner. Projektet återfinns på <a href="https://github.com/johnnyhansson/WixDemo" target="_blank">Github</a>.
<h2>Referenser</h2>
[1] "Customizing Built-in WixUI Dialog sets", <a href="http://wixtoolset.org/documentation/manual/v3/wixui/wixui_customizations.html">http://wixtoolset.org/documentation/manual/v3/wixui/wixui_customizations.html</a>

[2] "Property Reference",
<a href="https://msdn.microsoft.com/en-us/library/aa370905(v=vs.85).aspx">https://msdn.microsoft.com/en-us/library/aa370905(v=vs.85).aspx</a>

[3] "WiX Tutorial", <a href="https://www.firegiant.com/wix/tutorial/">https://www.firegiant.com/wix/tutorial/</a>

[4] "System folder properties", <a href="https://msdn.microsoft.com/en-us/library/aa372057.aspx">https://msdn.microsoft.com/en-us/library/aa372057.aspx</a>

[5] "Using Project References and Variables", <a href="http://wixtoolset.org/documentation/manual/v3/votive/votive_project_references.html">http://wixtoolset.org/documentation/manual/v3/votive/votive_project_references.html</a>
