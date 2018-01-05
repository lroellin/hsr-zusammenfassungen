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

## StringFormat

Entspricht dem Format-String im Aufruf an die String.Format-Methode

## Path

Path ist die Standard-Property eines Binding-Ausdrucks. Folgendes ist daher identisch

* `{Binding FirstName}`
* `{Binding Path=FirstName}`

Es kann ebenso Objektsyntax verwendet werden

* `{Binding}` bindet direkt an Datenquelle selbst als Ganzes
* `{Binding Address.Street}` bindet an Property Street der Property Address der Datenquelle
* `{Binding Groups[0].Name}` 

## Mode

Richtung der Datenbindung. Default-Mode je nach gebundener Property unterschiedlich. **Im Zweifelsfall den Mode jeweils explizit angeben**

* OneTime: einmalig (bei erstem Zugriff)
* OneWay: lesend
* TwoWay: lesend/schreibend
* OneWayToSource: schreibend

## IValueConverter

![195A237D-B3CE-4EB6-8E90-F19A3C2EF674](Bilder/195A237D-B3CE-4EB6-8E90-F19A3C2EF674.png)

Beispiel: BooleanToVisibilityConverter (aus dem Standard-Lieferumfang)

> Es ist auch möglich, nur eine der Methoden zu überschreiben und die andere mit NotImplementedException auszustatten, wenn man es selber nicht verwendet

![FFECF7F7-20CA-4E3E-AB0D-6E9FA62D7791](Bilder/FFECF7F7-20CA-4E3E-AB0D-6E9FA62D7791.png)

```c#
public class BooleanToVisibilityConverter : IValueConverter {
  public object Convert(object value, Type targetType, object parameter, CultureInfo culture) {
    return value is bool && (bool)value == true ? Visibility.Visible : Visibility.Collapsed;
  }
  
  public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture) {
    return value as Visibility? == Visiblity.Visible
  }
}
```

## IMultiValueConverter

![B9DC0744-8664-4526-875F-99561E3AD8EA](Bilder/B9DC0744-8664-4526-875F-99561E3AD8EA.png)

![07E6AB5C-1C81-4D37-9C8F-C5D642EB17D3](Bilder/07E6AB5C-1C81-4D37-9C8F-C5D642EB17D3.png)

![3AB2B2EB-816B-4497-A5C3-75C960076E86](Bilder/3AB2B2EB-816B-4497-A5C3-75C960076E86.png)

![14C1C24D-D015-4AAD-B928-7F755DAE6535](Bilder/14C1C24D-D015-4AAD-B928-7F755DAE6535.png)

## Anwenden

Um einen ValueConverter anzuwenden, wird eine Instanz benötigt

* Variante 1: Instanz als Ressource in XAML anlegen

  * (z.B. in App.xaml)

  * ```xml
    <Application.Resources>
    	<local:BooleanToVisibilityConverter x:Key="MyVisiblityConverter" />
    </Application.Resources>
    ```

  * Dann: Nutzung wie gehabt

  * ```xml
    <Label Visiblity="{Binding IsAvailable}, Converter={StaticResource MyVisibilityConverter}"
    ```

  * ​

* Variante 2: Instanz als statische Property in C# bereitstellen

  * zum Beispiel gleich im ValueConverter

  * ![EB218637-81AF-4758-98E4-C9998DA0697B](Bilder/EB218637-81AF-4758-98E4-C9998DA0697B.png)

  * Dann: Nutzung via `{x: Static ...}` anstele von `{StaticResource ...}`

  * ```xml
    Converter="{x:Static local:RgbToColorConverter.Instance} 
    ```

  Gut abwägen, ob sich der Aufwand lohnt!

## INotifyPropertyChanged

In der Vorlesung geht man nun durch einige Varianten, wie man es implementieren kann

![DC5B48DB-0E50-42ED-B706-C0FFBF3FB5D2](Bilder/DC5B48DB-0E50-42ED-B706-C0FFBF3FB5D2.png)

![88C46036-6133-4743-84AB-814226EB2631](Bilder/88C46036-6133-4743-84AB-814226EB2631.png)

![10685645-DF37-4C79-9A0B-3C46985DEE26](Bilder/10685645-DF37-4C79-9A0B-3C46985DEE26.png)

![821D71E6-2907-46CD-86B7-489E1B0CD1A0](Bilder/821D71E6-2907-46CD-86B7-489E1B0CD1A0.png)

Achtung: mit Zwang zum Ableiten sind keine POCOs mehr möglich.

Problem: berechnete Properties. Änderung sollte immer dann kommuniziert werden, wenn Quellwerte ändern

Lösung: für jeden Wechsel eine eigene Notifikation übermitteln

![9F592F2B-2FFC-4E19-819D-677BFD762EA9](Bilder/9F592F2B-2FFC-4E19-819D-677BFD762EA9.png)

![D4B935FF-1750-491B-BE92-3C7D9F56FFDD](Bilder/D4B935FF-1750-491B-BE92-3C7D9F56FFDD.png)

Variante 4: Fody verwenden.

1. POCO schreiben
2. Mit Attribut `[ImplementPropertyChanged]` markieren

## Binding auf Collections

`ObservableCollection<T>` verwenden, implementiert `INotifyCollectionChanged` und `INotifyPropertyChanged`

Änderungen an den Inhaltsobjekten selbst benötigen weiterhin `INotifyPropertyChanged`

## ObjectDataProvider

Ermöglicht Verwendung einer Factory-Methode in XAML

Beispiel: Enums

![C583A54E-F0DC-41ED-8E4C-A2D62A87A3D1](Bilder/C583A54E-F0DC-41ED-8E4C-A2D62A87A3D1.png)

## ItemTemplate

![41AF3427-9AF9-4E70-9EA0-4B6E8C977B38](Bilder/41AF3427-9AF9-4E70-9EA0-4B6E8C977B38.png)

![0E17B098-2B0E-45FE-AD7D-BA10BA5F5617](Bilder/0E17B098-2B0E-45FE-AD7D-BA10BA5F5617.png)

## DataGrid

Microsoft wollte das ursprünglich nicht mehr in WPF anbieten

![0CA1F1DB-15F4-4ECB-BB9A-C609A53C52AF](Bilder/0CA1F1DB-15F4-4ECB-BB9A-C609A53C52AF.png)

## CollectionViewSource

Ermöglicht Gruppieren, Sortieren, Filtern

```xml
<CollectionViewSource x:Key="ListingDataView" Source="{Binding Source={x:Static Application.Current}, Path=AuctionItems}" />
```

# Details der Benutzerinteraktion

## Events

Publish/Subscribe-Mechanismus

Event in Klasse definieren

Innerhalb der Klasse auslösen (Publish)

Für Benachrichtigungen registrieren (Subscribe)

![EA3F82CE-108A-4E7A-BEC0-A133D911B0FB](Bilder/EA3F82CE-108A-4E7A-BEC0-A133D911B0FB.png)

## Event Args

Von EventArgs abgeleitete Klasse für Übermittlung der Event-Details. Konvention: Klassenname endet auf EventArgs

![D432A7B5-F1B2-44FB-913E-F676C76AC0EA](Bilder/D432A7B5-F1B2-44FB-913E-F676C76AC0EA.png)

Event-Handler:

```c#
private void OnPropertyChanged(object sender, PropertyChangedEventsArgs args)
{
  	// extract the property name out of the event args parameter
  	var propertyName = args.PropertyName
  	// get the new value of the mentioned property via reflection
    var newValue = sender.GetType().GetProperty(args.PropertyName).GetValue(sender);
  	// write to console
  	Console.WriteLine($"The property {propertyName} has changed to {newValue}")
}
```

![DC335332-0441-4631-8C26-CCED47C188E5](Bilder/DC335332-0441-4631-8C26-CCED47C188E5.png)

### Konvention

Name des vordefinierten Event Handlers zu einem Event: "OnEVENTNAME"

Bei virtuellen Eventhandlern: Sender oft weggelassen

Zugriffskategorie meist `protected`

## Varianten für Event Handler

Event

```c#
this.Startup += (o, args) => {}
```

Variante vordefinierter Handler: OnStartup überschreiben

Er muss dann auch nicht mehr registriert werden (das macht WPF)

```c#
protected override void OnStartup(StartupEventArgs e) {}
```

## WPF App Lifecycle

C#/.NET generell: statische Main-Methode festlegen

Falls nur 1 Main-Methode in Assembly: wird automatisch durch Compiler festgelegt. Falls mehrere, Hauptklasse "Startup Object" in Build Settings festlegen.

Wo ist Main in WPF?

![668E51C1-2F04-4790-9642-0B36FFC358BF](Bilder/668E51C1-2F04-4790-9642-0B36FFC358BF.png)

### App Startup

![77EB98A1-E648-49D7-A6C4-29C9751404BF](Bilder/77EB98A1-E648-49D7-A6C4-29C9751404BF.png)

![DC73E2C4-F408-403 0-96E2-6BE9F3119645](Bilder/DC73E2C4-F408-4030-96E2-6BE9F3119645.png)

### Window Close

![FA7A22A6-4B15-42A5-A23A-CB300709594E](Bilder/FA7A22A6-4B15-42A5-A23A-CB300709594E.png)

### App Close

![254FF66E-C76E-4D90-B869-ACA7DBF54085](Bilder/254FF66E-C76E-4D90-B869-ACA7DBF54085.png)

## Routed Events = XAML UI Events

Interessante UI-Ereignisse, auf die reagiert werden soll

* Mouse-Ereignis, linke Maustaste gedrückt/losgelassen, Maus bewegt
* Keyboard-Ereignis
* ...

Können abgebrochen werden (`Handled = true`)

Nutzen das C# Event Konzept

Werden abwärts (*Preview*/*Tunneling*) und dann aufwärts (*Bubbling*) durch den **Visual Tree** gesendet

![5881353A-A917-4802-9D05-4774EBF10544](Bilder/5881353A-A917-4802-9D05-4774EBF10544.png)

![59A5B34A-AEDF-4C78-A09E-6B393DED83AE](Bilder/59A5B34A-AEDF-4C78-A09E-6B393DED83AE.png)

### Behandeln

![5C379A1C-12DF-4FBF-94E3-3AC361BE7B12](Bilder/5C379A1C-12DF-4FBF-94E3-3AC361BE7B12.png)

Bei einem Parent-Element

![461002AC-7F40-4678-938A-6081A1AC00DE](Bilder/461002AC-7F40-4678-938A-6081A1AC00DE.png)

![468E06B0-28A6-4274-9A8B-13C67989B09D](Bilder/468E06B0-28A6-4274-9A8B-13C67989B09D.png)

## RoutedEventArgs

* Handled: Flag, ob Event behandelt wurde
* OriginalSource: Quell-Element (Hit-Testing), welches das Event ausgelöst hat
* RoutedEvent: Routed Event, welches mit diesem Objekt verknüpft ist
* Source: Element, welches das Event rapportiert hat

Source vs. OriginalSource

* OriginalSource: u.U. Kind-Element des Source-Elements, per Hit-Testing als 1. Adressat des Events bestimmt
* Source: Element im Logical Tree, welches das Event dann rapportiert

Beispiel: Subscribe auf MouseDown in einem Button. Vom Button rapportiert (Source), aber eigentlich vom Rahmen ausgelöst (OriginalSource)

Dazu gibt es dann abgeleitete Klassen, wie die MouseEventArgs.

* Left/Middle/RightButton (Pressed/Released)
* Delta: Mausradbewegung (120/-120)

Keyboard:

* Key (Enum)
* IsDown/IsUp
* IsRepeat
* SystemKey: gibt bei gedrückter ALT-Taste die zusätzlich gedrückte Taste an (Key ist dann Key.System)

Beispiel: Taste B wird gedrückt und wieder losgelassen

![FB9A6090-4AC5-47D6-B633-B936B1396129](Bilder/FB9A6090-4AC5-47D6-B633-B936B1396129.png)

![F4AB31D0-8081-472B-B09A-B5440C020B31](Bilder/F4AB31D0-8081-472B-B09A-B5440C020B31.png)

In der Vorlesung folgt nun eine Problemstellung, dass man unbedingt den Auslöser überprüfen sollte, wenn man einen Event erhält (kommt der von mir?)

## Drag & Drop

![6278BA13-1505-4BF3-AB06-696272AC4AFA](Bilder/6278BA13-1505-4BF3-AB06-696272AC4AFA.png)

> Spick: Übungslösung übertragen, oder Appendix-Teil

## Hintergrundoperationen

1. visuelles Feedback geben
   1. Button deaktivieren
   2. Spinner anzeigen
      1. Button mit Label und Spinner, Visibility des Spinners an IsEnabled des Buttons binden (eigener Converter nötig da umgedrehte Logik)
2. Starten eines Background-Threads
3. UI-Thread benachrichtigen
   1. `Dispatcher.Invoke(() => {});
   2. Falls man nicht in einem WPF-Control ist: `System.Windows.Application.Current.Dispatcher.Invoke` (statische Instanz)

# Architekturmuster und Diverses

![E5342FAB-DA99-4630-8D82-3B4250BB6C0E](Bilder/E5342FAB-DA99-4630-8D82-3B4250BB6C0E.png)

## Model

Eigentliche Daten, Domain-Objekte

* *Kein* Verhalten
* Nicht zuständig für Darstellung und Formatierung der Daten
* Nicht zuständig für Laden und Speichern
* Abbild der realen Welt, oft ergänzt um Informationen zur Nutzung, Speicherung, Tracking

(die Vorlesung zeigt kurz das Entity Framework und SQLite)

Model: DTO, oder direkt Entity wenn man keine DTO benutzt

## ViewModel

* View-spezifische Objekte, welche die Domain-Objekte um Daten, Berechnungen für die View anreichern
* Enthalten Verhalten
* Zuständig für Darstellung und Formatierung der Daten im Hinblick auf die Darstellung in der View
* Grundlage für stressfreies Data Binding
* Kontroverse: zuständig/nicht zuständig fürs Laden/Speichern

Zwei Varianten: 

* Model als Property kapseln
* Für jede im UI benötigte Property eine gleichnamige VM-Property implementieren
  * Dann Fleissarbeit, oder Tool verwenden (z.B. AutoMapper) um Properties aus Model zu kopieren

```c#
// Initialize Mapper table
Mapper.Initialize(cfg => cfg.CreateMap<Gadget, GadgetVm>());

// copy the properties of gadget into a new instance of type GadgetVm
var vm = Mapper.Map<GadgetVm>(gadget);
```

## Commands

Bindable Methods

Problem: View soll nichts von ViewModel wissen, wie kann ich dann Aktionen im VM auslösen?

Lösung: `ICommand` im ViewModel implementieren, dann via DataBinding auslösen

Begriffe

* The *command* is the action to be executed (Kommando)
* The *command source* is the object which invokes the command (XAML)
* The *command target* is the object, that the command is being executed on (ViewModel)
* The *command binding* is the object which maps the command logic to the command

Beispiel: ein GadgetVM soll sich selbst speichern können. Dies als Reaktion auf einen Button-Klick

Wir schreiben ein eigenes Command `SaveCommand`

* Command führt das Speichern aus
* Command Source ist der Button
* Command Target ist das GadgetVm-Objekt in der Detail View
* Das Command Binding definieren wir auf dem Button, so dass dieser das Kommando beim Klicken auslöst

### Nutzen von Commands

Data Binding im XAML-Code wie gehabt

Voraussetzung: DataContext für Binding korrekt auf ViewModel gesetzt

```xml
<Button Content="Save" Command="{Binding SaveCommand}" />
```

oder mit zusätzlichem Parameter: `CommandParameter="{Binding SelectedGadget}"`

Commands werden beim Data Binding an ein passendes Event eines Controls gebunden. Beim `Button` ist das der `Click`-Event.

Wir schweifen jetzt kurz ab, später kommt dann der konkrete SaveCommand!

Commands haben ein `CanExecute`-Property, das der Button automatisch an das `IsEnabled`-Property bindet.

### Implementierung von Commands

Variante 1: eigene Command-Klasse, Instanz davon als Property im ViewModel

```c#
public class SomeCommand: ICommand {
  	public bool CanExecute(object parameter) { /* return true/false */ }
  	
  	public void Execute(object parameter)
    {
      // irgendwas ausführen
    }
    
  	public event EventHandler CanExecuteChanged;
}

public class GadgetVM : BindableBase
{
	public SomeCommand MyCommand { get; set; } // kann auch ICommand MyCommand sein
	
	public GadgetVm()
    {
    	MyCommand = new SomeCommand();
    }
}
```

Variante 2: Command als innere Klasse im ViewModel implementieren

Vorteil: Zugriff auf private Member des ViewModels

Variante 3: RelayCommand implementieren (selber)

```c#
public class RelayCommand : ICommand
{
	private readonly Action _execute;
	private readonly Func<bool> _canExecute;
	
	public RelayCommand(Action execute, Func<bool> canExecute = null)
    {
    	if (execute == null) throw new ArgumentNullException("execute");
    	
    	_execute = execute;
    	_canExecute = canExecute;
    }
    
    public bool CanExecute(object parameter) => _canExecute?.Invoke() ?? True
    public void Execute(object parameter) => _execute();
}
```

Oder auch mit generischem <T>

```c#
public class RelayCommand<T> : ICommand
{
	private readonly Action<T> _execute;
	private readonly Predicate<T> _canExecute;
	
	public RelayCommand(Action<T> execute, Predicate<T> canExecute = null)
    {
    	if (execute == null) throw new ArgumentNullException("execute");
    	
    	_execute = execute;
    	_canExecute = canExecute;
    }
    
    public bool CanExecute(object parameter) => _canExecute?.Invoke((T)parameter) ?? True
    public void Execute(object parameter) => _execute((T)parameter);
}
```

Die Klasse geht aber jeweils noch weiter:

```c#
// Event an CommandManager delegieren (Benachrichtigung erfolgt so immer dann 
// wenn WPF denkt, dass sich etwas am Ausführungsstatus geändert hat,
// z.B. bei Key- oder Mouse-Button-Klick)
public event EventHandler CanExecuteChanged
{
	add { CommandManager.RequerySuggested += value; }
	remove { CommandManager.RequerySuggested += value }
}
```

![CBD9750F-FBCE-41AC-8894-ECBCA5595DED](Bilder/CBD9750F-FBCE-41AC-8894-ECBCA5595DED.png)

Verwendung:

```c#
public class GadgetVm : BindableBase
{
	public ICommand SaveCommand { get; set; }
	
	public GadgetVm()
    {
    	SaveCommand = new RelayCommand(
    		() => this.Save(), // Methode des ViewModels
    		() => this.CanSave // Property des ViewModels
    	);
    }
    
    public CanSave => ; // some condition
    public void Save() {} // some method
}
```

Besser: Initialisierung direkt bei Definition:

```c#
public ICommand SaveCommand { get; } = new RelayCommand(() => this.Save(), () => this.CanSave);
```

Oder lazy

```c#
// Backing-Field benötigt
private ICommand _saveCommand;
public ICommand SaveCommand =>
  _saveCommand ?? (_saveCommand = new RelayCommand(() => this.Save(), () => this.CanSave));
```

## Events

View soll ViewModel nicht kennen, aber bei Auftreten von Events Command anstossen

Bekannte Lösung: mit Event Handling/Code Behind

Es gibt aber Alternativlösungen auf Nuget, womit man Events auf Kommandos mappen kann

## ViewModel vs. View

Werden beide "von aussen" erzeigt

Das ViewModel wird "von aussen" als DataContext für die View gesetzt

## ViewModel vs. Services

Immer mit Referenz auf service-Interface, und via Dependency Injection angeben.

## Die View

Möglichst wenig (am besten kein) Zugriff auf die UI-Controls aus dem Code Behind

Zustände via Data Binding zwischenspeichern

Steuerung der Anzeige via Flags des ViewModels binden

## Erzeugen des ViewModels

Es werden einige Varianten vogestellt, eine davon ist die Dependency Injection via Konstruktor

```c#
// onStartup
var vm = new GadgetVm();
MainWindow = new GadgeothekView(vm);
MainWindow.Show();

public GadgeothekView(GadgetVm vm)
{
  InitializeComponent();
  DataContext = vm
}
```

Dazu noch eine Variante direkt in XAML (das geht anscheinend...)

```xml
<Window ...>
	<Window.DataContext>
      <local:GadgetVm />
  	</Window.DataContext>
</Window>
```

```c#
public GadgeothekView()
{
  InitializeComponent(); // Code Behind also "leer"
}
```

So erreicht man, dass das ViewModel cross-plattform läuft und testbar ist. Zudem ist so 100% der Anwendung auch ohne UI testbar.

## Internationalisierung

Internationalisierung und Lokalisierung in CH nötig!

# Xamarin

X-Platform Native App Development C#

* Native User Interfaces
  * sehen aus und verhalten sich wie native Apps
* Native API Access
  * 100% der plattformspezifischen Device APIs
* Native Performance
  * Nutzen die volle Geschwindigkeit der Geräte

Klassischer Xamarin-Ansatz

* Native UI X-mal bauen
* Shared C# Backend

Xamarin Forms

* Native UI Tweaks builden lassen
* Shared C# Backend

Von C# zu nativem Code: Windows

![30382A54-5ACB-42D7-8461-DB42834FAA0F](Bilder/30382A54-5ACB-42D7-8461-DB42834FAA0F.png)

iOS/Android

![DD1FB9C6-BE1B-45A9-B042-DD827891FB4B](Bilder/DD1FB9C6-BE1B-45A9-B042-DD827891FB4B.png)

Xamarin.Forms: alle XAML-Konzepte X-Plattform nutzbar, Controls werden pro Plattform auf Native Controls gemappt.

