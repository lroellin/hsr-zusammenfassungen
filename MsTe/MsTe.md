---
typora-copy-images-to: ./Bilder
---

# MsTe

[TOC]

# Grundlagen

## Bestandteile

* Common Language Runtime CLR
  * ähnlich JVM
  * Common Type System CTS
  * Common Language Specification CTS
* .NET Class Library
  * Basis-Klassen, ADO.NET, XML, ...

## Architektur

![46169A14-E090-415D-8FDA-666083A4CE6D](Bilder/46169A14-E090-415D-8FDA-666083A4CE6D.png)

## Common Language Runtime

Laufzeitumgebung

* JIT für ILC (~Bytecode)
* Class Loader
* GC
* Code Access Security
* Debugging (sprachübergreifend)
* Exceptions
* Type Checking/Code Verifikation des ILC
* Threads
* COM (unmanaged Code)
* Basis-Klassen mit System-Funktionen

![5BC620AF-5E72-4439-A756-2C1BD4AC977C](Bilder/5BC620AF-5E72-4439-A756-2C1BD4AC977C.png)

## MSIL

Assembler-ähnlich

Virtuelle Stack-Maschine ohne Register

Common Type System: komplexe Datentypen/Objekte, Boxing/Unboxing

Vererbung/Polymorphie

Vorteile

* Portabilität
* Typsicherheit

Nachteile

* Laufzeiteffizienz
  * kann durch JIT-Compiler wettgemacht werden, wenn dieser maschinenspezifische Instruktionen ausnutzen kann

![4DC5CB5E-AB56-4D00-84A4-E9F149B45FDC](Bilder/4DC5CB5E-AB56-4D00-84A4-E9F149B45FDC.png)

## Cross Language Development

Objekt-Modell und Bibliotheken in Framework integriert. Sprachwahl sekundär, Konzepte allgemeingültig

Common Language Specification

* Allgemeine Regeln für Cross-Language Development
* CLS-kompatible Bibliotheken können von allen .NET-Sprachen verwendet werden

![23B2DE11-D9F7-47DA-8469-E8FB24A8A34C](Bilder/23B2DE11-D9F7-47DA-8469-E8FB24A8A34C.png)

![53D6CF08-3F10-4162-94C7-DE55E4ACB8E0](Bilder/53D6CF08-3F10-4162-94C7-DE55E4ACB8E0.png)

## Assemblies

Assembly: selbstbeschreibende Komponente mit definierter Schnittstelle

![90998F75-3C06-4AE0-B938-DD93BA5E3A8D](Bilder/90998F75-3C06-4AE0-B938-DD93BA5E3A8D.png)

Kompilation erzeugt Assemblies und Module

* Assembly besteht aus 0 bis Module
* Visual Studio ertellt standardmässig nur 1 Modul pro Assembly

Assembly

* Deployment-/Ausführungseinheit
* exe, dll
* Binärdateien im PEF (ausführbar für Win32/Win64)
* Dynamisch ladbar
* Definiert Typ-Scope
* kleinste versionierbare Einheit
* Einheit für Security Überprüfung

Entspricht ungefähr einem JAR-File

Singlefile/Multifile

![0BC2F95F-B1FF-4036-B3F3-5BFCAB8BB9D2](Bilder/0BC2F95F-B1FF-4036-B3F3-5BFCAB8BB9D2.png)

Kompilation erzeugt ein Modul mit 

* Code/MSIL

Metadaten beschreiben alle Aspekte des Codes ausser Programmlogik

* Klassen-Definitionen
* Methoden-Definitionen
* Feld-Definitionen
* usw

Metadaten lassen sich über Reflection abfragen

![F3CD5276-9978-4EF7-81AF-AEEFEA770361](Bilder/F3CD5276-9978-4EF7-81AF-AEEFEA770361.png)

![D08497AC-BC3C-483F-A876-2D326D790183](Bilder/D08497AC-BC3C-483F-A876-2D326D790183.png)

## Assemblies

Private Assembly: referenziert über Dateipfad, nirgends fix registriert, meist nur von einer Applikation genutzt

Shared Assembly

* eindeutige Bezeichnung (Strong Name)
  * Bezeichnung
  * Versionsnummer
  * Kulturinformation (falls vorhanden)
  * Public Key Token
* Im Global Assembly Cache registriert (gacutil.exe)
* steht allen Applikationen zur Verfügung
* Keine DLL-Hell
  * mehrere Versionen existieren parallel
  * Applikationen referenzieren spezifische Version

## Kompilierung

`csc.exe`für die Kompilierung von C# Quellcode

für jede Sprache separater Compiler

Aufruf aus spezieller Kommandozeile (Developer Command Prompt)

![22FC50EE-5B70-48CB-A731-53F8F628AA16](Bilder/22FC50EE-5B70-48CB-A731-53F8F628AA16.png)

![4EAE4F87-49FA-41AF-9CA7-EE251398DCAF](Bilder/4EAE4F87-49FA-41AF-9CA7-EE251398DCAF.png)

![196C71FB-73AD-44A9-AC30-F31ACE6FEB7A](Bilder/196C71FB-73AD-44A9-AC30-F31ACE6FEB7A.png)

![173C58A1-1CA6-4B6D-A1D5-B032CD91D5CB](Bilder/173C58A1-1CA6-4B6D-A1D5-B032CD91D5CB.png)

## Common Type System

Single-rooted Typsystem

> Alle Typen sind von System.Object abgeleitet (auch int, float usw.)

2 Kategorien von Typen: Reference/Value Types

Boxing/Unboxing automatisch

Polymorphismus mit `System.Object` unterstützt daher auch int usw.

![6B8DE186-61FA-4819-B8B6-ED094ED9BA32](Bilder/6B8DE186-61FA-4819-B8B6-ED094ED9BA32.png)

|                          | Reference (Class) | Value (Struct)     |
| ------------------------ | ----------------- | ------------------ |
| Speicherort              | Heap              | Stack              |
| Variable enthält         | Referenz          | Wert               |
| Nullwerte                | Möglich           | Nie                |
| Default Value            | null              | 0 \| false \| `\0` |
| Zuweisung/Methodenaufruf | Kopiert Referenz  | Kopiert Wert       |
| Ableitung möglich        | Ja                | Nein (sealed)      |

## Reference Types

> Objekt hat Referenz auf seine Typbeschreibung!

![194896B2-4270-4195-BC27-01AC17457AEF](Bilder/194896B2-4270-4195-BC27-01AC17457AEF.png)

![3520031D-4BFA-4A92-88FD-45CE30F8A2F4](Bilder/3520031D-4BFA-4A92-88FD-45CE30F8A2F4.png)

## Value Types

Keine Garbage Collection nötig, da auf Stack!

![8208B894-873D-4EE5-B0E0-73599CB938EE](Bilder/8208B894-873D-4EE5-B0E0-73599CB938EE.png)



![FC553396-3B6E-4E9B-9316-C91F8824F224](Bilder/FC553396-3B6E-4E9B-9316-C91F8824F224.png)



## Vergleich

* Reference Types: Objekt-Referenz wird kopiert
* Value-Types: Wert wird kopiert

## Boxing/Unboxing

Boxing automatisch (Up Cast), Unboxing Casting nötig (Down Cast)

```c#
System.Int32 i1 = 123;
System.Object obj = i1; // Boxing
System.Int32 i2 = (System.Int32)obj; // Unboxing
```

![EAA7BB9C-14D9-4689-82EF-D4F962A47285](Bilder/EAA7BB9C-14D9-4689-82EF-D4F962A47285.png)

## CLS

Die CLS definiert eine Teilmene von Typen, die in interoperablen Schnittstellen verwendet werden können. Zum Beispiel sind unsigned-Typen verboten, oder Pointer-Typen und jagged-Arrays sind auch verboten.

Falls man Sprach-Interoperabilität möchte, muss CLS eingehalten werden

Beispiel:

* Server-Komponente in C#
* Client in VB.NET oder JScript

![3BE2BF51-3889-4CA1-88C7-74B58042F77B](Bilder/3BE2BF51-3889-4CA1-88C7-74B58042F77B.png)

# C# Grundlagen

## Zahlen

Unicode

Case-sensitive

Wenn man Schlüsselwörter als Identifier verwenden möchte: `@` davorstellen (`@if`)

Unicode-Escape Sequenzen erlaubt

![46F4FA29-326E-41A4-9F2F-6FA779D3C425](Bilder/46F4FA29-326E-41A4-9F2F-6FA779D3C425.png)

![49DFC199-69C6-40C5-B2A7-85630C4E3BD8](Bilder/49DFC199-69C6-40C5-B2A7-85630C4E3BD8.png)

Ab C#7.0 auch `_` zur optischen Strukturierung von Zahlen. Wird vom Compiler schlichtweg entfernt.

## Zeichen/Zeichenketten

![3830C3C5-F8F2-40C4-A480-8E789CF321B9](Bilder/3830C3C5-F8F2-40C4-A480-8E789CF321B9.png) 

![96E21524-D07D-456F-B164-014B21B897B1](Bilder/96E21524-D07D-456F-B164-014B21B897B1.png)

## Kompatibilität

![1DBF1405-BBB7-42D7-B501-86231D63BA63](Bilder/1DBF1405-BBB7-42D7-B501-86231D63BA63.png)

## Sichtbarkeit

| Attribut           | Beschreibung                             |
| ------------------ | ---------------------------------------- |
| public             | Überall sichtbar                        |
| private            | Innerhalb des jeweiligen Typen (Klasse / Struct) sichtbar |
| protected          | Innerhalb des jeweiligen Typen (Klasse / Struct) oder abgeleiteter Klasse sichtbar |
| internal           | Innerhalb des jeweiligen Assemblies sichtbar |
| protected internal | Innerhalb des jeweiligen Typen (Klasse / Struct) oder abgeleiteter Klasse sichtbaroderInnerhalb des jeweiligen Assemblies sichtbar |

| Typ       | Standard | Zulässig (Top-Level)* | Standard für Members | Zulässig für Members                     |
| --------- | -------- | --------------------- | -------------------- | ---------------------------------------- |
| class     | internal | public / internal     | private              | public<br />protected<br />internal<br />private<br />protected internal |
| struct    | internal | public / internal     | private              | public<br />internal<br />private        |
| enum      | internal | public / internal     | public               | –                                        |
| interface | internal | public / internal     | public               | –                                        |
| delegate  | internal | public / internal     | –                    | –                                        |

`*` gilt nicht für Nested Types