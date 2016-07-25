---
layout: post
title: NUnit 3 och TeamCity Could not load file or assembly 'nunit.framework' or one of its dependencies
---

Under dagen har jag varit sysselsatt med att flytta en installation av TeamCity från en maskin som vi använt för utvärderingssyfte till en virtuell maskin på Azure. Själva flytten var inget problem, men en av byggkonfigurationerna som kör enhetstester kunde inte köras korrekt. Istället misslyckades körningen av testerna med felmeddelandet

```
Could not load file or assembly 'nunit.framework' or one of its dependencies. The system cannot find the file specified.
```

Efter viss felsökning visade sig felet vara lätt avhjälpt. Jag hade nämligen varit för girig i mitt uttryck som angav vilka assemblies som skulle inkluderas vid körningen och även de assemblies som låg i obj-katalogen inkluderas.
