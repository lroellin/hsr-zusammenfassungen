---
typora-copy-images-to: ./Bilder
---

# WPF

WPF: Trennung von Darstellung (XAML) und Code

Begriffe

* Markup: Design, Elemente, Layout (XAML)
* Code Behind: Programmcode

Pixel in WPF sind device-independent Pixel (1/96") (in Windows Forms sind Pixel = Bildschirmpixel)

XAML-Elemente werden 1:1 zu erzeugten CLR-Objekten. Attribute werden 1:1 zu Properties und Events der erzeugten CLR-Objekte.

> Alles was in XAML implementiert werden kann, kann auch in Code (C#) ausgedrückt werden

![5F3930AA-74E2-41B1-AEB9-3191F37917BE](Bilder/5F3930AA-74E2-41B1-AEB9-3191F37917BE.png)

Faustregel: was in XAML geht, in XAML machen, da oft leichtgewichtiger, kürzer, und auch mit einem Designer definierbar.

## Logical Tree

![76453EEF-38BC-40D9-8CC7-97BE55E11FC8](Bilder/76453EEF-38BC-40D9-8CC7-97BE55E11FC8.png)

XAML-Elemente

* Entspricht der Struktur der XAML-Elemente
* Beschreibt die Beziehungen zwischen verschiedenen Elementen des UIs

Zuständig für

* Dependency Properties erben
* Dynamische Ressourcen-Referenzen auflösen
* Elementnamen für Datenbindung nachsclagen
* Routed Events weiterleiten

## Visual Tree

Dargestellte UI-Elemte und Dekorationen

* Entspricht der grafischen Repräsentation
* Beinhaltet alle dargestellten Elemente gemäss der Vorlage des Controls

Zuständig für

* Visuelle Darstellung
* Vereben der Transparenzeinstellungen, Transformationen, `IsEnabled`-Property
* Hit-Testing

## Routed Events

![D6583F5F-EC6B-4514-91E3-FC3475A34FA1](Bilder/D6583F5F-EC6B-4514-91E3-FC3475A34FA1.png)

## Attribute Syntax/Property Element Syntax

```xml
<Button
        Height="50"
        Width="200"
        Content="Watch Now"
        />

<Button
        Width="120"
        Height="50">
  <Button.Content>Watch Now</Button.Content>
</Button>
```

Die Property Element Syntax ist so natürlich relativ sinnlos. Aber man kann mit ihr zusammengesetzte Dinge bauen:

![C1597ADC-FDB3-4E54-9A3D-C86D46034A25](Bilder/C1597ADC-FDB3-4E54-9A3D-C86D46034A25.png)

## Attached Properties

Element setzt Eigenschaften, die das Parent-Element betreffen.

```xml
<Button DockPanel.Dock="Top" />
```

=

```c#
DockPanel.SetDock(element, Dock.Top)
```

## Markup Extensions

Verkürzte Notation um komplexe Ausdrücke unter Verwendung der "Attribute Syntax" eingeben zu können

```xml
<TextBox Text="{Binding Path=FirstName}" />
<!-- Kurzform von -->
<TextBox><TextBox.Text><Binding.Path="FirstName" /></TextBox.Text></TextBox>
```

## Type Converters

Konvertieren String-Werte automatisch in einen passenden Datentyp

```xml
<Button Background="Aqua" />
<!-- Kurzform von -->
<Button><Button.Background><SolidColorBrush Color="Aqua" /></Button.Background></Button>
```

## Events

![910A3EF6-327A-4D02-827D-ED57CCA7EEB5](Bilder/910A3EF6-327A-4D02-827D-ED57CCA7EEB5.png)

![517DF439-C295-4EC1-B8C1-CC6A5A37E700](Bilder/517DF439-C295-4EC1-B8C1-CC6A5A37E700.png)

![1F62D9B8-4E20-49FC-92C5-5546937569F7](Bilder/1F62D9B8-4E20-49FC-92C5-5546937569F7.png)

# Layout und Controls

Falls nicht anders angegeben: Einheiten in `px`. 

Priorität bei Platzprobleme

1. MinWidth
2. MaxWidth
3. Width

## Alignment

Left/Center/Right, Top/Center/Bottom, sowie `Stretch` **als Standard**: füllt die ganze verfügbare Breite/Höhe im Parent-Element

> Tiefere Priorität als explizite Width/Height-Angaben!

## Ränder/Rahmen

* l, t, r, b
* l,t => l = links/rechts, t = oben/unten
* x = alle

Bei CornerRadius: 

* tl, tr, br, bl

## Box

Width = Inhalt + Border + Padding (CSS: `box-sizing: border-box`)

## Sonstiges

* IsEnabled: UI interagierbar (Standard = True)
* SnapsToDevicePixels: rundet Pixelangaben auf physische Gerätepixelwerte (Rendering)
* Visibility: Collapsed, Hidden, Visible (Standard)

## Brush

* SolidColorBrush
* LinearGradientBrush
* RadialGradientBrush
* ImageBrush
* DrawingBrush
* VisualBrush

Vordefinierte Pinseltypen: benannte Farben, UI-Farben (`MenuColor`)

Opacity: Transparenzwert 0..1

Rendering effizienter als wenn Opacity des Elements gesetzt wird

## Clipping

`ClipToBounds`: Sollen Child Controls an den Rändern des Parent Controls abgeschnitten werden? (Standard = False)

## Stack Panel

Child Elemente werden horizontal/vertikal aufgelistet. Standard vertikal

> Restlicher Platz wird nicht ausgefüllt

## WrapPanel

Wie StackPanel, aber mit Zeilen-/Spaltenumnbruch

In der Praxis zu Layoutzwecken nur selten anwendbar

![DB5007C1-CBCE-48E8-A265-195FA591FAB5](Bilder/DB5007C1-CBCE-48E8-A265-195FA591FAB5.png)

## Dock Panel

Es gibt ein Top, Bottom, Left, Right. Die Reihenfolge der Elemente ist wichtig!

Wird z.B. erst ein Top und dann ein Left spezifiziert, so belegt das Top den ganzen oberen Rand und Left nur den restlichen, linken Platz.

Gibt man nichts an, wandert es ins Center.

## Grid

Zeilen/Spalten explizit deklarieren. Dort gibt es eine Breiten/Höhenangabe:

* Ganzzahlen: fixe Breite/Höhe
* `Auto`: automatisch aus verfügbarem Platz hergeleitet
* `*`: nutzt ganze Breite/Höhe entsprechend dem verfügbaren Platz, proportional aufgeteilt
* `2*`: verfügbarer Platz wird gewichtet aufgeteilt
* Dazu MinWidth/MinHeight

Mit `Grid.Column`/`Row` wird der Platz im Grid angegeben. Mit `RowSpan`/`ColumnSpan` kann gespannt werden.

Spezialfall 1x1 Grid: Elemente werden in der Zelle gestapelt, mit Alignment/Margin flexibles Layout erstellbar

![ACDD9FFD-B642-4F0F-83FF-204B43B6D12C](Bilder/ACDD9FFD-B642-4F0F-83FF-204B43B6D12C.png)

Prozentangaben grundsätzlich nicht möglich. Trick: Gewichtung, z.B. 20, 40, 40

## Canvas

Absolute Positionierung, keinerlei Layout-Logik

Mit Canvas.Left/Top/Right/Bottom Abstand zum jeweiligen Rand angeben, dazu noch Canvas.ZIndex für Z-Position

![35B1368C-DBEF-4949-8BAA-39F819686A8A](Bilder/35B1368C-DBEF-4949-8BAA-39F819686A8A.png)

## Shapes

![B9D115C1-0F0A-428B-AD15-03EDD306065C](Bilder/B9D115C1-0F0A-428B-AD15-03EDD306065C.png)

![FE731DB0-8453-4302-BB5E-40A597E6D231](Bilder/FE731DB0-8453-4302-BB5E-40A597E6D231.png)

![E1C0C77D-5E16-4BD9-9D11-D1E3C1C931A0](Bilder/E1C0C77D-5E16-4BD9-9D11-D1E3C1C931A0.png)

Intern verwenden sie leichtgewichtige Model-Klassen für Geometrie-Berechnungen, die man auch zum Definieren einer Clipping-Form verwenden kann

## ViewBox

Skaliert einzelnes Child Control (inkl. Text), um den verfügbaren Platz auszunutzen. Funktioniert mit Transformation. Optionen `Stretch=`:

![B4C9A723-56AC-4F0D-AE8F-FF72B68391F2](Bilder/B4C9A723-56AC-4F0D-AE8F-FF72B68391F2.png)



## Image

Attribut `Source` für Angabe der Datenquelle, verhält sich ansonsten wie ViewBox Stretch. Übergrösse wird bei Alignment berücksichtigt.

## Border

Zeichnet Rahmen um ein Child Control. Da `Border` nur ein Kind haben können, können sie mit Panels (z.B. StackPanel) kombiniert werden.

## Eigene Controls

UserControl: Komposition

* wiederverwendbare Zusammenstellung mehrerer Controls als Gruppe
* Besteht aus einem XAML und einem Code Behind-File
* Kann nicht mit Styles/Templates umgehen, dies muss vom umgebenden Fenster kommen

CustomControl: Ableitung

* Erweitert ein bestehendes Control um neue Funktionen
* Besteht aus einem Code-File und allenfalls einem Standard-Style
* Kann mit Styles-Templates umgehen

## Dialogfenster

Vorgehen

1. Dialogfenster erstellen
2. Dialogfenster mit `ShowDialog` anzeigen
3. Spezialvariable `DialogResult` setzen. true = OK, false = Abgebrochen

> Setzen der Variable: Dialogfenster wird automatisch geschlossen (Seiteneffekt)

4. Abrufen allfälliger Daten aus der Dialogfenster-Instanz

### Beispiel

* Default-Button: `IsDefault=true`
* Cancel-Button: `IsCancel=true`

## Fenster mit Spezialformen

1. `AllowTransparency`= True
2. `WindowStyle = None`
3. dazu `ResizeMode = NoResize` (sinnvollerweize)

Form in Window.Clip mittels eines Geometry-Elements festlegen

```xml
<Window.Clip>
	<RectabngleGeometry RadiusX="8" RadiusY="8" Rect="0, 0, 800, 600"
</Window.Clip>
```

(Ergibt Rechteck mit abgerundeten Rändern)

Beliebige Formen mit PathGeometry möglich. Fenstergrösse muss natürlich auf Geometrie abgestimmt sein.

## Testing

In der Vorlesung wird nun gezeigt, wie ein UI-Test gemacht wird

# UI Design

## LinearGradientBrush

![0BBB7382-9694-4019-B398-8BE29362D3B7](Bilder/0BBB7382-9694-4019-B398-8BE29362D3B7.png)

## Ressourcen

Physische Ressourcen: z.B. Bilder `<Image Source="Desert Landscape.jpg">`

`Resource`: beliebiges Objekt (= Instanz), das in XAML definiert werden kann. Wird mit dem KeyAttribut aus dem `x`-Namespace benannt:

![50C1BA8C-8B28-4BC5-B6EE-B50C3133D51E](Bilder/50C1BA8C-8B28-4BC5-B6EE-B50C3133D51E.png)

### ResourceDictionary

Behälter, um Ressourcen zu speichern. Indexiert nach dem Ressourcen-Namen. 

Zugriff auf Resource:

1. Key wird im Element und in allen Parent-Nodes gesucht (Logical Tree)
2. Key wird in `Application.Resources`gesucht
3. Key wird in SystemRessourcen gesucht

![7D4EA66C-31CC-4E23-B0C1-451E583C4585](Bilder/7D4EA66C-31CC-4E23-B0C1-451E583C4585.png)

### Zugriff auf statische Werte

```xml
<Button Background="{:Static SystemColors.ControlBrush}" Content="Save" />
```

System-Ressourcen:

* im Namespace `System.Windows`
* `SystemColors`
* `SystemFonts`
* `SystemParameters`

### Statisch/dynamisch

* `StaticResource`: Statische Bindung beim Laden (1x)
  * Compile Time Check (findet Fehler früh)
* `DynamicResource`: dynamische Bindung zum Zugriff (jedes Mal)
  * Runtime Check (lässt dynamisch erzeugte und geladene Ressourcen zu)

## Ressourcen in C#

```c#
var okText = (string)FindResource("OkText");
var bgBrush = FindResource("DarkBrush") as Brush;
```

## Markup Extension

![643430D1-F8AB-4276-99EA-FD8BDDB262E5](Bilder/643430D1-F8AB-4276-99EA-FD8BDDB262E5.png)

## Eigene ResourceDictionaries

Separates xaml-File. In andere ResourceDictionaries einbindbar. XML-Root-Node ist `ResourceDictionary`.

```xml
<ResourceDictionary <!-- Namespace-Quatsch --> >
	<!-- Hier Ressourcen angeben -->
</ResourceDictionary>
```

Zum Einbinden:

```xml
<Application.Resources>
	<ResourceDictionary>
      <!-- Mixen möglich -->
      <SolidColorBrush x:Key="MyButtonBackground" Color="#EEEEE" />
      <ResourceDictionary.MergedDictionaries>
      	<ResourceDictionary Source="Colors.xaml" />
        <ResourceDictionary Source="Brushes.xaml" />
      </ResourceDictionary.MergedDictionaries>
      <!-- Wiederverwendung möglich (definiert in Colors.xaml -->
      <SolidColorBrush x:Key="ButtonBgBrush" Color="{StaticResource ThemeColor1}" />
	</ResourceDictionary>
</Application.Resources>
```

## Externe Ressourcen: externe Assembly

![D6278557-D5D5-4046-B8B1-CF43517FADF1](Bilder/D6278557-D5D5-4046-B8B1-CF43517FADF1.png)

![21E9E228-3A77-4E2B-A7E6-C20E8EF26582](Bilder/21E9E228-3A77-4E2B-A7E6-C20E8EF26582.png)

### absolute Pfadangabe

Ressourcen im Filesystem. Nicht zu empfehlen, da zu starke Bindung

```xml
<Image Source="C:\Program Files\X-App\media\pix\open.png" />
```

### Package-URI

Kolleg...

![63563C09-11F6-4FE0-A317-EB9A731CF733](Bilder/63563C09-11F6-4FE0-A317-EB9A731CF733.png)

Randnotiz:

> `/BasePicLib/...` hat einen Default-Prefix: `pack://application:,,,`

## Styles definieren

### explizite Styles

```xml
<Style x:Key="MyButtonStyle">
	<Setter Property="Button.Foreground" Value="#000000" />
	<Setter Property="Button.Background" Value="#FFFFFF" />
	<!-- auch komplexes möglich -->
  	<Setter Property="Button.Background">
      <Setter.Value>
        <LinearGradientBrush StartPoint="0,0" EndPoint="0,1">
        	<!-- ... -->
        </LinearGradientBrush>
      </Setter.Value>
  	</Setter>
</Style>
```

Verwenden mit:

```xml
<Button Style="{StaticResource MyButtonStyle}" Content="Cancel" />
```

Man kann auch einen `TargetType="Button"` einfügen. Der Klassenname (`Button.Foreground`) kann dann weggelassen werden (`Foreground`).

Lässt man den Key weg, **gilt der Style für alle Elemente des Controls**

Man kann auch ableiten, mit `BasedOn={StaticResource MyButtonStyle}`.

Möchte man aber den Grundstil als Standard-Stil haben, muss man einen Sentinel einfügen, der den Key weglässt und nur BasedOn ist (der Grundstil muss einen Key haben, damit man davon ableiten kann)

```xml
<Style TargetType="Button" x:Key="BaseButton"><!-- Nutzung über Key -->
  <!-- verschiedene Setter -->
</Style>
<!-- Sentinel, gilt für alle Button -->
<Style TargetType="Button" BasedOn="{StaticResource BaseButton}" />
<!-- spezifischer Stil, Nutzung über Key -->
<Style TargetType="Button" x:Key="DisabledButton" BasedOn="{StaticResource BaseButton}">
	<!-- verschiedene Setter -->
</Style>
```

## ControlTemplate

XAML-Struktur, die Inhaltsdarstellung eines Controls festlegt

Ermöglicht starke Wiederverwendung der Basis Control-Logik bei sehr flexibler Darstellung

![41F3E9BA-8B57-4C9D-93F6-1488732AC257](Bilder/41F3E9BA-8B57-4C9D-93F6-1488732AC257.png)

TemplateBinding: nur innerhalb Control Template anwendbar. Kann Wert einer Dependency Property im Control (oder Style) abrufen

## Trigger

Style anhand unterschiedlicher UI-Zustände anpassen

![4A04F87C-B8F8-491D-82C5-AB59EC6CD9B9](Bilder/4A04F87C-B8F8-491D-82C5-AB59EC6CD9B9.png)

## Visual State Manager

![E6B2858F-4121-4791-8BB7-7C31244CDF9C](Bilder/E6B2858F-4121-4791-8BB7-7C31244CDF9C.png)

![8FD93486-2529-46A4-93AE-1C931E2B8A99](Bilder/8FD93486-2529-46A4-93AE-1C931E2B8A99.png)

## Transformationen

`LayoutTransform`: werden vor der Layout-Phase angewandt

`RenderTransform`: werden nach der Layout-Phase angewandt

![0125C8CC-B2EC-431C-B60C-5E52EFED8C7D](Bilder/0125C8CC-B2EC-431C-B60C-5E52EFED8C7D.png)

![C62232DE-F949-428F-92E0-82B06EAA1426](Bilder/C62232DE-F949-428F-92E0-82B06EAA1426.png)

## GUI Design Principles

![58B37D57-42C3-4B1A-941E-A7E710ECEB65](Bilder/58B37D57-42C3-4B1A-941E-A7E710ECEB65.png)

In der Vorlesung wird ab S. 77 beschrieben, wie Dynamic Layout mit WPF erzielt werden kann

# Umgang mit Daten

## Markup Extensions

![C89BDAC6-3012-42A9-9301-A01879D4EB13](Bilder/C89BDAC6-3012-42A9-9301-A01879D4EB13.png)

## Binding

![F7D42A89-6D92-4902-B909-DE1BB217856B](Bilder/F7D42A89-6D92-4902-B909-DE1BB217856B.png)

![220C6199-31C1-46D0-A0CA-DF659FADD405](Bilder/220C6199-31C1-46D0-A0CA-DF659FADD405.png)

### DataContext

Datenbindungsausdrücke sind relativ zum DataContext zu formulieren

Data Binding Auswertung: es wird im Control + dessen Parents nach `DataContext != null` gesucht

![9956FF18-7B96-4842-99D5-09CA4E24941E](Bilder/9956FF18-7B96-4842-99D5-09CA4E24941E.png)

![B473787C-1DF8-4013-916A-5B8D8A2757C6](Bilder/B473787C-1DF8-4013-916A-5B8D8A2757C6.png)

![9CBC3E79-1E00-4ADA-8901-2506C7E6251E](Bilder/9CBC3E79-1E00-4ADA-8901-2506C7E6251E.png)

Und auf einzelnes Objekt **unüblich**. Bei jedem UI-Wechsel Anpassung im Code Behind nötig. Stattdessen mit separaten ViewModels arbeiten und nur DataContext für Fenster setzen.

![F049E1E1-3C52-45EA-B35C-6B187A04E047](Bilder/F049E1E1-3C52-45EA-B35C-6B187A04E047.png)

## Designer

Kennt DataContext nicht => by default kein IntelliSense für DataBinding

Abhilfe: Design-Time Data Context setzen

Variante 1: DesignInstance

![88EB3931-70FB-4F4C-BC08-DAB9712509A4](Bilder/88EB3931-70FB-4F4C-BC08-DAB9712509A4.png)

Variante 2: eigene, statische Design Instanz

![658BA8CF-3868-4147-AACC-C4AB515461CD](Bilder/658BA8CF-3868-4147-AACC-C4AB515461CD.png)

## Source/RelativeSource/ElementName

Source:

DataContext für einzelne Elements übersteuern: Datenquelle mittels Source explizit angeben

![D9D897BF-E8C1-4630-B8E2-7388350B8BD8](Bilder/D9D897BF-E8C1-4630-B8E2-7388350B8BD8.png)

RelativeSource:

ermöglicht die Angabe einer relativen Datenquelle im **Visual Tree**

Eigene Markup Extension dafür: RelativeSource

![2BA77C60-ACB3-45CA-89BA-8B6C4B2D5DCE](Bilder/2BA77C60-ACB3-45CA-89BA-8B6C4B2D5DCE.png)

ElementName

Nutzt direkt anderes, benanntes XAML-Element als Datenquelle. Namen müssen im gleichen Namensraum vorliegen. 

![278265D5-E59D-4A11-B9E9-2836BE04060D](Bilder/278265D5-E59D-4A11-B9E9-2836BE04060D.png)