---
typora-copy-images-to: ./Bilder
---

# Einführung und Konzepte

Linux, primär Java 7

App-Komponenten

* Activies (~ Screens)
* Services
* Content Provider
* Broadcast Receivers

System hat feste Kontrollen über App

* Verwaltung des Lebenszyklus
* Kommunikation zwischen Komponenten
* Apps werden automatisch geschlossen um Speicher zu sparen

Komponten anderer Apps können aufgerufen werden, sowie Systemkomponenten ersetzt werden (andere Kamera oder Browser-App)

Activity: hier passieren User-Eingaben. = GUI + Code

Grundgerüst:

```java
public class MainActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // Hier unser Code
  }
}
```

> Kein `main` oder `new MainActivity`, dies wird vom System instanziert

### Lifecycle

Das System ruft bei Zustandswechseln die jeweilige Methode der Activity-Klasse auf. Der Entwickler überschreibt Callback-Methoden, die ihn interessieren.

![CA93F832-1032-4F66-8925-C8D661742BB8](Bilder/CA93F832-1032-4F66-8925-C8D661742BB8.png)

> Unterschied Paused/Resume: wird die Activity von einer anderen überdeckt (z.B. Notificationleiste heruntergezogen), wird die erste pausiert (`onPause`). Kommt sie wieder in den Vordergrund, wird wieder `onResume` aufgerufen.

Stopped/Started: kommt die Activity wieder in den Vordergrund, weil der User die Applikation nochmal startet oder mit dem Back-Button zurückkommt, wird `onRestart` aufgerufen

Destroyed: die App wird vom System destroyed, oder wenn sie explizit vom User geschlossen wird

> Dies kann auch direkt aus dem `onPaused`-Zustand geschehen (roter Shortcut)

Braucht Android den Speicher wieder, wird die Applikation gekillt. Kann aus

* `onPause`
* `onStop`
* `onDestroy`

geschehen

Tipp: Daten in `onPause` sichern, da man nicht sicherstellen kann dass die anderen beiden aufgerufen werden

> Bei Konfigurationsänderungen (Screengrösse/Ausrichtung) wird die Activity auch neu gestartet

### Best Practices

* GUI beim Start in `onCreate` erstellen
* Auf Benutzereingaben in `onResume` reagieren
* Daten in `onPause` sichern
* Ressourcen in `onStop` freigeben

### Lifecycle komplett

> Nur `onPause` ist garantiert, alles andere nicht

![2D12109F-BC1D-41C9-9D7B-DF42B767C409](Bilder/2D12109F-BC1D-41C9-9D7B-DF42B767C409.png)

## Activity Stack

Activities werden in einem Stack verwaltet.

> Activies eines Stacks können zu verschiedenen Apps gehören

![83A3DDD6-EC12-4010-B9F9-23300150D32C](Bilder/83A3DDD6-EC12-4010-B9F9-23300150D32C.png)

## Tasks

Eine Gruppe von Activities nennt man auch Task

Task lassen sich über den Overview-Screen anzeigen

Activities vs. Task: Browser-Analogie

* Browser: Tabs mit je eigener History beliebiger Seiten
* Android: Tasks mit je eigenem Stack von Activities

## Launch Modes

Android verwaltet Tasks und Activities für uns

Default

* Start der App von Launcher oder Homescreen startet neuen Task
* Back geht zum vorherigen Screen zurück (pop)

Kann geändert werden, aber nur wenn wirklich nötig (Game). Launch-Modes können dieses Verhalten ändern

Buchstaben stehen für Activities, Farben sind Tasks

![4B7E1023-8295-4CCB-9DB6-09295A7A5A66](Bilder/4B7E1023-8295-4CCB-9DB6-09295A7A5A66.png) 

## Activities/Tasks aus Systemsicht

Activities und Ressourcen werden in ein APK gepackt und installiert

Wird eine Activity aktiv, wird pro APK ein Linux-Prozess mit einem Thread gestartet

Dieser Prozess führt alle Activities, die in diesem APK enthalten sind, aus

> Jedes APK wird unter einem eigenen Linux-User installiert

![80459A72-400B-4726-956F-4D1F42E2D9F5](Bilder/80459A72-400B-4726-956F-4D1F42E2D9F5.png)

APK sind JARs mit der Erweiterung `.apk`, die wiederum ZIP-Dateien sind.

Inhalt: Libraries, Ressourcen, Assets, Metadaten, kompilierte Klassen im DEX-Format

## Intents

Intent beschreibt, was gemacht werden soll

* expliziter Aufruf einer Klasse
  `new Intent(this, CalculateActivity.class)`
* impliziter Aufruf der passenden Komponente
  `new Intent(MediaStore.ACTION_IMAGE_CAPTURE)`
  * System entscheidet, wer zuständig ist
  * ermöglicht lose Kopplung und Austauschbarkeit
* Typischerweise eigene Klassen explizit aufrufen, Systemaktionen implizit auslösen (Bild erstellen, E-Mail senden)
* Apps können Activites zur Verfügung stellen die bestehende Applikationen ersetzen (z.B. Kamera)

### Parameterübergabe

Intent-Klasse kennt `setData`-Methode, welche URI entgegennimmt

* Ziel der ausszuführenden Aktion
* Datei, URL, Kontakt, ...

Zusätzliche Parameter an Activity übergeben: `intent.putExtra(MediaStore.EXTRA_OUTPUT, imageCaptureUri)`

* Key-Value-Paare mit weiteren Daten
* Nur für primitivate Daten, Strings, und alles was serialisierbar ist.

### Activities starten

* Kontrolle übergeben: `startActivity(intent);`

* Rückgabewert erhalten

* ```java
  startActivityForResult(intent, SOME_ID);

  // Callback
  @Override
  protected void onActivityResult(int request, int result, Intent data) {
    if (result == Activity.RESULT_OK && request == SOME_ID) {
      // Verarbeitung
    }
  }
  ```

### Weiteres

* Services starten
  * Hintergrundprozesse ohne UI
* Broadcasts versenden
  * an alle interessierten Apps im System versendet
  * z.B. Batterie leer oder geladen

## View

GUI: alles was der Benutzer sieht und mit dem er interagiert

Deklarativ in XML oder imperativ in Java-Code

Komponenten hierarchisch

* ViewGroup enthält andere Komponenten
* ViewGroup bestimmt das Layout der Kinder
* View ist die Oberklasse aller GUI-Elemente

![135F879D-0477-4477-9114-BC2AA558EB74](Bilder/135F879D-0477-4477-9114-BC2AA558EB74.png)

### Beispiel

![D58BE482-B198-43BA-90D5-945B5A9DC680](Bilder/D58BE482-B198-43BA-90D5-945B5A9DC680.png)

Buttons sind Views. Das Layout drumherum ist eine ViewGroup, aber auch wiederum eine View (Vererbung)

## Manifest

* Komponenten der App
* Metadaten (Name, Icon, Versionsnummer)
* Permissions
* Anforderungen an Geräte-API
* Alle Activities der App

Versionen

* `minSdkVersion` gibt an, welche Version das Gerät mindestens haben muss
* `targetSdkVersion` ist die höchste Version, mit der getestet wurde

Android Studio verwaltet einige Informationen im Gradle-File und baut erst beim Build das reale Manifest.

## Application

Parent unserer Activities im Manifest ist die Application

Ist auch eine Klasse, die den globalen Zustand unserer App hält

Kann durch eigene Application-Subklasser ersetzt werden

Zugriff aus Activity mit `getApplication`, Lifecycle-Methoden: `onCreate`, `onLowMemory`, `onConfigurationChanged`

## Grundlagen der GUI-Programmierung

Android View

* belegt rechteckigen Bereich auf Screen
* ist zuständig, dessen Inhalt zu zeichnen
* ist zuständig, Events zu behandeln

Untergruppen sind Widgets (nicht Homescreen-Widgets) und ViewGroups. Das SDK enthält auch vordefinierte, komplexere Elemente.

Widget = fixfertige Komponenten. Buttons, Images, Checkboxen. 

ViewGroup = Unterklasse von View. ViewGroup ~= Layout

## Layout

Für alle gemeinsam sind die `width` und `height`-Parameter. Zulässige Werte für `android:layout_width=`/`height` sind

* `match_parent`:  so gross wie möglich, also der Parent erlaubt
* `wrap_content`: so klein wie möglich, also die Kinder erlauben
* absolute Werte, aber selten empfehlenswert

## Linear Layout

Elemente werden vertikal oder horizontal angeordnet.

Standardmässig gleich viel Platz, aber selten sinnvoll. Daher kann man mit `android:layout_weight` ein Gewicht vergeben. Kinder ohne Weight bekommen minimalen Platz (Vermutung: `wrap_content`), der restliche Platz wird nach Gewicht aufgeteilt

![051B76F4-D17E-46FE-B088-F1FE97942ABB](Bilder/051B76F4-D17E-46FE-B088-F1FE97942ABB.png)

## Relative Layout

Die Kinder werden relativ zueinander angeordnet. Also beispielsweise `android:layout_below="@id/first"`, und `android:layout_toStartOf="@id/first"`. 

Daneben gibt es noch `toRightOf` usw, aber wenn man nicht explizit rechts sondern das Ende meint, nutzt man besser die normalen Werte, dann geht RTL korrekt.

## Weitere

* Framelayout kann Kinder übereinander anordnen
  * Live-Kamerabild mit Auslöser und Hilfslinien
  * Kartenansicht mit Zoom-Buttons
* FlexboxLayout: CSS Flexbox
* ConstraintLayout, mit RelativeLayout verwandt (war in Android Studio immer Standard ab irgendeiner Version)
* WebView um HTML anzuzeigen
  * JavaScript kann aktiviert werden, und Java-Objekte können mit JS angesprochen werden

## Layout und Activity

mit `setContentView(R.layout.activity_main)` setzt man das Layout für die aktuelle Activity.

## Eingabefelder

Mit `inputType` gibt man die erwartete Eingabeform an, z.B. Telefonnummern, Nummern, oder auch E-Mailadressen usw. 

> Diese können auch kombiniert werden: `textCapSentences|textAutoCorrect`

## Eigene Views

Es können auch eigene Views eingebunden werden. Dazu nimmt man als Tag einfach den Klassenamen.

## ID/`R`

Mit `@id/bla` referenziert man das Element mit dieser ID. Damit es das überhaupt gibt, wird es mit `android:id="@+id/bla"` erstellt.

Das Android-Buildsystem sammelt alle diese IDs als Konstanten in der automatisch generierten Klasse `R`.  So können wir auf die in XML deklarierten Elemente zugreifen.

Ebenso liegen Ressourcen in `R`. Die Ordnernamen müssen in Java-Namen umgewandelt werden können, dürfen also z.B. kein `-` enthalten.

Weitere Ressourcentypen

* drawables
* menu
* mipmap - Launcher-Icon der App
* values - Strings und andere Konstanten

## Values Ressourcen

Beispielweise Strings, Farben und Dimensionen

Dimensionen aus `dimens.xml`:

```xml
<RelativeLayout
	android:paddingLeft="@dimen/activity_horizontal_margin"
/>

<!-- dimens.xml -->
<resources>
	<dimen name="activity_horizontal_margin">16dp</dimen>
</resources>
```

## Dimensionen

In density-independent pixels: `dp` oder `dip`

![A5815133-B4C5-475A-AAF1-A89C27584833](Bilder/A5815133-B4C5-475A-AAF1-A89C27584833.png)

Bei Schriften verwendet man scale-independent Pixels `sp`

## Ressourcen-Varianten

Die Auslagerung in XML-Dateien dient auch dazu, dass die Definitionen in untetrschiedlichen Varianten hinterlegt werden können. Also Bilder in verschiedenen Auflösungen, Texte in verschiedenen Sprachen usw.

Android Studio hat dazu einen Wizard.

## Events

Sobald die Lifecycle-Methoden aufgerufen wurden, hat unsere App keine aktive Kontrolle mehr. Das Android-Framework hat einen Event-Loop (Looper). Dieser wartet bis ein Ereignis passiert und bearbeitet dieses dann.

Nur der Main-Thread darf das GUI verändern.

Auslöser

* durch Benutzer (Touch-Geste, Hardwaretaste)
* Sensoren (GPS meldet neue Location)

Um die Events zu empfangen müssen wir Listener beim entsprechenden Objekt registrieren.

Für die unterschiedlichen Typen gibt es unterschiedliche Listener, zum Beispiel:

* OnTouchListener für Touch-Events wie Gesten
* OnClickListener wenn eine View "angeklickt" wird
* OnLongClickListener touch-and-hold
* OnKeyListener Tasten

Für bestimmte Widgets gibt es noch spezialisierte Interfaces, wie z.B. den `TextWatcher` für Eingabefelder.

## GUI-Objekt finden

(mit Kotlin geht das in manchen Fällen automatisch, aber auch nicht immer zuverlässig...)

```java
Button button = (Button) findViewById(R.id.button)
```

`findViewById` der Activity sucht im aktuellen Layout (wahrscheinlich dasjenige von `setContentView`). Rückgabe ist immer die Oberklasse `View`,  das Resultat muss also noch gecastet werden.

## Listener registriern

```java
button.setOnClickListener(new View.OnClickListener() {
  @Override
  public void onClick(View v) {
   // ... 
  }
});
```

Alternativ dazu kann auch die Activity das Interface implementieren (`setOnClickListener(this`))

Sie können auch im XML deklariert werden. Das eignet sich dann, wenn die Activity sonst nicht viel macht, ansonsten ist es schöner in einer eigenen Klasse

`android:onClick="onButtonClicked"`

die Activity muss dann

`public void onButtonClicked(View view)`

implementieren. Mittels Reflection wird diese Methode dann gefunden.

Ein Listener lässt sich auch bei mehreren Views registrieren. Mit dem übergebenen View-Parameter kann man unterscheiden (Referenz-Vergleich `==`) welches Element den Event ausgelöst hat.

## Texteingabe-Events

Das `TextWatcher`-Interface implementiert 3 Methoden

* `beforeTextChanged` wird aufgerufen bevor der Text geändert wird, wir sehen also noch den alten Text
* `onTextChanged` wird aufgerufen, sobald der Text geändert hat (meistens interessiert uns nur diese Methode)
* `afterTextChanged` nachdem der Texte geändert wurde, hier haben wir noch die Chance den Text anzupassen (Achtung vor Endlosschleife)

## Inputvalidierung

EditTexts bieten eine einfache Möglichkeit, Inputvalidierung anzuzeigen. Mit `setError` kann eine Nachricht gesetzt werden, diese wird bei jeder Änderung resettet.

```java
final EditText password = findViewById(R.id.password);
password.addTextChangedListener(new TextWatcher() {
   @Override
   public void afterTextChanged(Editable s) {
       String pw = s.toString();
       if (pw.length() < 8) {
           password.setError("Passwort muss mindestens 8 Zeichen lang sein.");
       }
   }
});

```

# Strukturierung und Navigation

Zurück

* zum hierarchischen Parent: Ancestral Navigation
* zum vorherigen Element: Temporal Navigation

Ancestral-Navigation geschieht über den Up- oder Home-Button

Temporal Navigation über den Back-Button

### Screen-Map Vorgehen

1. Domainmodell entwerfen
2. Screens ableiten
3. Screens in Beziehung bringen und gruppieren (Handy Master-Detail, Tablet 1 Screen)
4. Navigation zwischen den Screens festlegen
5. Wireframe/Storyboard für die Gesamtübersicht erstellen
6. Usability-Test mit einem Paper-Prototypen des Wireframes (HCID)

## Fragments

Wiederverwendbare Teil-Activities

Fragment kann in mehreren Activities eingebunden werden

Activity kann mehrere Fragments beinhalten

Eigener Lebenszyklus! (es rastet uus)

![D59D83FA-CA12-4F3C-9FC9-7E47A1B22036](Bilder/D59D83FA-CA12-4F3C-9FC9-7E47A1B22036.png)

Zusätzliche Zustände

* onAttach: Fragment wird einer Activity hinzugefügt
* onCreateView: UI des Fragments erstellen
* onActivityCreated: wenn Activity onCreate fertig ist
* onDestroyView: Gegenstück zu onCreateView
* onDetach: Gegenstück zu onAttach

## Fragment instanzieren

`LayoutInflater` nimmt XML und instanziert die View-Klassen

`ViewGroup container` ist Layout in der Parent-Activity

![CCBE7775-366B-49BD-993F-20F176BB4ED6](Bilder/CCBE7775-366B-49BD-993F-20F176BB4ED6.png)

Fragments können statisch oder dynamisch in die Activity eingebunden werden.

Statisch:

```xml
<LinearLayout>
	<fragment 
     android:name="com.example.myfragmentapplication.MainActivityFragment"
     tools:layout="@layout/fragment_main" />
</LinearLayout>
```

In der Activity dann wie gewohnt das äussere Layout setzen.

Dynamisch:

```xml
<LinearLayout>
	<FrameLayout android:id="@+id/fragment_container"/>
  	<!-- Frame Layout hat sich so eingebürgert -->
</LinearLayout>
```

```java
public class MainActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    FragmentManager fragmentManager = getFragmentManager();
    FragmentTransaction fragmentTransaction = fragmentManager.beginTransation();
    
    MainActivityFragment fragment = new MainActivityFragment();
    fragmentTransaction.add(R.id.fragment_container, fragment);
    fragmentTransaction.commit();
  }
}
```

## Activity <==> Fragment-Kommunikation

Activity bei `onAttach` in Field speichern. Die Fragments sind wiederverwendbare Komponenten und sollten keine Abhängigkeit auf Activity haben. Daher Best Practice: Callback-Interface für Fragment => Activity-Kommunikation

![D18227DA-2747-46BE-92A7-967127911309](Bilder/D18227DA-2747-46BE-92A7-967127911309.png)

## Master-Detail

Um das sauber zu unterscheiden, baut man zwei Einstiegs-Activities. Eine Default (enthält nur Master, für Phone) und eine grösser 600dp, also `sw600dp`, die Master und Detail enthält. Ferner baut man die beiden Fragments, und schlussendlich noch eine Detail-Activity fürs Phone.

Dann hat sich als Best Practice eingebürgert:

1. `boolean twoPane` zum Merken, ob wir im Tablet-Modus sind
2. Wenn `findViewById(R.id.item_detail_container) != null`, `twoPane = true`.

Dann in einem Callback, das von der Masterliste aufgerufen wird, checken in welchem Modus man sich befindet, und entweder eine Activity starten oder das Fragment austauschen.

## Verschachtelung

Ab Android 4.2, API-Level 17, können Fragments verschachtelt werden. Dann muss man mit dem `getChildFragmentManager` gearbeitet werden, z.B. beim ViewPager.

## Support Library

Fragments sind erst später hinzugekommen, daher braucht man für ältere APIs die Support-Library. Man muss aber alles mit oder ohne Support-Library machen, die Java-Typen sind unterschiedlich.

## Menüs

In `onCreateOptionsMenu(Menu menu)` erstellen (oder in XML bauen und mit `getMenuInflater().inflate(R.menu.menu_main, menu)` verlinken), dann in `onOptionsItemSelected(MenuItem item)` behandeln. Die Erkennung funktioniert dort über Konstanten, oder über die ID.

> `true` bedeutet behandelt

Von Fragments aus können auch Menü-Optionen hinzugefügt werden. Dazu gibt es auch dort `onCreateOptionsMenu(Menu menu, MenuInflater inflater)`, dort wird der Inflater also bereits mitgegeben.

>  In `onCreate` muss unbedingt `setHasOptionsMenu(true)` gesetzt werden, sonst wird wahrscheinlich das Callback nie aufgerufen

## Action Bar

Welche Actions in der Action-Bar angezeigt werden, hängt vom

*  verfügbaren Platz ab
* der Menü-Konfiguration ab (`showAsAction`)
* Dazu gibt es sogenannte ActionViews, wie z.B. ein Plus-Button der expandiert

## Split Action Bar / Tabs

Je nach Gerät und Theme können die Buttons auch unten angehängt werden. Es gibt auch eine Tab-Bar.

> Die Action-Bar ist seit Android 5 deprecated.

## Toolbar

Seit Lollipop gibt es die neue ToolBar, die auch einen ActionBar-Modus hat um z.B. das Options-Menü nachzubauen.

Zum Verwenden, im onCreate:

```java
Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
setSupportActionBar(toolbar);
```

## Navigation Drawer

Platzsparend, aber der Hamburger wird auch kritisch gesehen

![F9831A48-5303-478E-AA4C-E0D023A77BFF](Bilder/F9831A48-5303-478E-AA4C-E0D023A77BFF.png)

### Implementierung

![8AE6D09B-5AA5-4357-9BEB-A2DDD534196D](Bilder/8AE6D09B-5AA5-4357-9BEB-A2DDD534196D.png)

In der Activity kann dann ein Listener registriert werden, welcher auf die Menüpunkte des Drawers reagiert

![E1F82F48-4BBB-46A5-9FE0-2C9F15D0C6E6](Bilder/E1F82F48-4BBB-46A5-9FE0-2C9F15D0C6E6.png)

## Bottom Navigation View

3-5 Elemente, von iOS bekannt

## Support Library

NavigationView ist nicht Teil von Android, aber über die Design Support Library erhältlich.

## Toasts

kleine Feedback-Nachrichten. Länge `LENGTH_SHORT` oder `LENGTH_LONG`, und Position mit `setGravity` (kombinierbar mit `|`).

## Snackbar

Löst den Toast ab. Flexibler, da auch eine Action angegeben werden kann.

![6721442B-4813-4A6F-BEF4-3930DCB613E2](Bilder/6721442B-4813-4A6F-BEF4-3930DCB613E2.png)

## Settings

Wird in XML deklariert und mit einer PreferenceActivity Subklasse eingebunden

![4F95871A-ED20-4BA4-ABFA-D20C7702BACC](Bilder/4F95871A-ED20-4BA4-ABFA-D20C7702BACC.png)