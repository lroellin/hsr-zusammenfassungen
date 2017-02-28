---
typora-copy-images-to: ./Bilder
---

# Multi-Threading Grundlagen

## Prozesse vs. Threads

* Prozesse laufen parallel zueinander im System. Sie haben einen eigenen Adressraum.

* Threads laufen parallel zueinander innerhalb eines Programms. Sie teilen sich den Adressraum im Prozess.

## Threads

Threads haben jeweils einen Snapshot der Register sowie einen eigenen Stack. Es gibt User-Level Threads, wo sich der Programmierer selber um die Parallelität und Scheduling kümmern muss, und Kernel-Level Threads, wo der Kernel fürs Scheduling zuständig ist.

In Java erstellt die JVM für Java-Threads jeweils einen Kernel-Thread.

## Kontextwechsel

Mit dem Kontextwechsel bezeichnet man den Wechsel zu einem anderen Thread. Es gibt zwei Arten

**Synchron:** Warten auf Bedingung Der Thread wartet auf eine Bedingung, er kennzeichnet sich als Wartend, reiht sich ein und gibt den Prozessor frei.

**Asynchron:** Nach gewisser Zeit soll der Thread den Prozessor freigeben. Damit verhindert man,dass ein Thread dauerhaft den Prozessor belegt.

## Multi-Tasking

**Kooperativ:** Die Threads müssen explizit Kontextwechsel synchron initiieren. Der Scheduler kann einen laufenden Thread nicht unterbrechen

**Preemptiv:** Der Scheduler kann per Timer-Interrupt den laufenden Thread asynchron unterbrechen. Zum Beispiel mit Time-Sliced Scheduling: jeder Thread besitzt den Prozessor für einen maximalen Zeitintervall. Heutzutage wird in der Regel nur preemptiv gearbeitet.

# JVM Threads

Die JVM ist ein Single Process System. Sie erzeugt beim Aufstarten einen Thread, den Main-Thread, welcher die statische Methode ``main()`` aufruft. Der Programmierer kann dann weitere Threads starten.

Gewisse Subsysteme/Laufzeitsysteme starten auch eigene Threads, z.B. AWT oder der Garbage Collector.

Die JVM läuft, solange Threads laufen Ausnahme hierzu sind Threads, die sich als Daemon markiert haben,auf diese wartet Java nicht (wie z.B. der Garbage Collector). **Daemon-Threads brechen bei JVM-Ende unkontrolliert ab**. Die JVM kann andererseits mit `System.exit()/Runtime.exit()` direkt terminiert werden, was aber unsauber ist. Alle Threads werden unkontrolliert abgebrochen.

## Erzeugung eines Threads

```java
class SimpleThread extends Thread { 
    @Override
    public void run() { // Thread implementieren
        // thread behavior
    } 
}
Thread myThread = new SimpleThread(); // Thread instanzieren

myThread.start(); // Thread starten
```

## Start und Ende

Der eigentliche Thread wird erst bei `start()` erzeugt. Er führt die `run()`-Methode des Objektes aus. DerThread endet beim Verlassen von `run()`. Dies aus folgenden Gründen:

* Ende der Methode
* Return-Statement
* Unbehandelte Exception
  * andere Threads laufen weiter!

## Multi-Thread Beispiel

Achtung: dieses Beispiel ist nicht-deterministisch! Threads laufen ohne Vorkehrungen beliebig verzahnt oder parallel. Viele JVMs realisieren System-Outputs ohne Verzahnung oder Thread-Fehler, aber es ist nichts spezifiziert.

```java
class SimpleThread extends Thread { 
    private String label;
    private int nofIt; 
    public SimpleThread(String label, int nofIt) {
        this.label = label; this.nofIt = nofIt;
    } 
    @Override
    public void run() {
        for (int i = 0; i < nofIt; i++) {
            System.out.println(i + " " + label);
        } 
    } 
} 
public class MultiThreadTest {
    public static void main(String[] args) {
        new SimpleThread("A", 10).start();
        new SimpleThread("B", 10).start();
        System.out.println("main finished"); 
    } 
}
```

Feinheiten: Wenn man statt ``start()`` ``.run()`` aufruft, wird nur die Methode aufgerufen und folglich sequentiell abgearbeitet.

## Runnable

Runnable sind Alternative zum Vererben von Thread. Runnable ist ein Interface statt eine Klasse, von derman erben muss (wie Thread), so dass man eine andere Basisklasse angeben kann.

```java
class SimpleLogic implements Runnable {
    @Override
    public void run() {
        // thread behavior
    }
}

Thread myThread = new Thread(new SimpleLogic());

myThread.start();
```

## Lambda

Weil ``Runnable`` nur ``run()`` implentiert (nur eine Methode), können wir es mit einem Lambda direkt behandeln.

```java
Thread myThread = new Thread(() -> {
    // thread behavior
});

myThread.start();
```

Achtung: Lambdas sind effectively final. Das bedeutet für den Zugriff auf Variablen/Parameter der umgebendenMethode:

* Variablen/Parameter müssen implizit final sein
* Auch ohne final-Deklaration
  * Ansonsten gibt es einen Compile-Fehler

```java
void startMyThread(String label, int nofIt) { 
    new Thread(() -> {
        for (int i = 0; i < nofIt; i++) {
            System.out.println(i + " " + label);
        } 
    }).start();
}
```

## Passivierung

* Die Thread-Klasse bietet zwei statische Methoden.
  `Thread.sleep(milliseconds)` Der laufende Thread geht in den Wartezustand. Nach Ablauf der Zeit wird er wieder Ready

* `Thread.yield()` Der laufende Thread gibt den Prozessor frei, wird aber direkt wieder ready. Dies provoziert mehr Thread-Wechsel.


## Thread Join: Warten auf Beendigung eines Threads

`t.join()`: blockiert solange bis `t` terminiert ist. Nach Join gilt: `!t.isAlive()`. Ein Zugriff auf Variablen von `t` ist erst nach dem Join sicher, dort könnten z.B. die Resultate des Threads gespeichert sein.

```java
Thread threadA = new SimpleThread("A", 100);
Thread threadB = new SimpleThread("B", 100);
System.out.println("Threads start");
threadA.start();
threadB.start();
try {
    threadA.join();
    threadB.join();
    System.out.println("Threads joined");
} catch (InterruptedException e) { }
```

Feinheiten: falls ein Thread noch nicht gestartet wurde, kehrt ``join()`` sogleich zurück.

## InterruptedException

Ist eine mögliche Exception bei blockierenden Aufrufen wie `sleep()` oder `join()`. Die Threads können von aussen mit `myThread.interrupt()` unterbrochen werden!

`interrupt()` ist für kooperatives Canceling gedacht. Nur verwenden, wenn die Cancel-Policy des Threads bekannt ist!

Sie wird oft zum Aufbrechen von Blockaden missbraucht. Dabei hinterlässt sie meist inkonsistente Zwischen-zustände, oder die Exception wird ignoriert und es wird weiter blockiert.

Achtung: Daemons hören nicht auf ``ÌnterruptedException``s! http://stackoverflow.com/questions/31252227/how-does-daemon-thread-survive-after-jvm-exits

## Weitere Thread-Methoden

* ``staticThreadcurrentThread()`` Gerade ausführendeThread-Instanz
* `void setDaemon(boolean on)` Thread als Daemon markieren, standardmässig false
* `String getName(),void setName(String name)` Thread-Name einstellen.
  * Beim Konstruktor `Thread(String name), Thread(Runnable r, String name)`
  * Für diagnostische Ausgaben: Default: Thread-0, Thread-1, ...

# Synchronisation

Ohne Vorkehrungen laufen Threads beliebig verzahnt oder parallel. Oft muss die Nebenläufigkeit von Threads aber beschränkt werden.

> Synchronisation = Einschränkung der Nebenläufigkeit

## Fälle für Synchronisation

**Gegenseitiger Ausschluss (mutual exclusion:** nur einer darf die Kaffeemaschine auf einmal benutzen, nur eine Vorlesung zur gleichen Zeit im gleichen Raum

**Warten auf Bedingungen:** Post erst abholen, wenn sie angeliefert worden ist, Telefon: erst reden wenn Gegenstelle abgenommen hat 

## Synchronized

Mit dem Java ``synchronized``-Keyword können Methoden mit gegenseitigem Ausschluss abgesichert werden. Der Body der Methode ist dann ein kritischer Abschnitt. 

```java
class BankAccount {
  private int balance = 0;
  
  public synchronized void deposit(int amount) {
    this.balance += amount;
  }
}
```

Jedes Objekt erbt von ``Object`` den Monitor-Lock. Maximal ein Thread kann denselben Lock haben. Der ``synchronized``-Block belegt den Lock des Objekts. 

* Bei Eintritt
  * Falls frei: besetzen
  * Falls besetzt: warten bis frei
* Bei Austritt: wieder freigeben

### Mehrere Methoden

Das Keyword ``synchronized`` kann auch bei mehreren Methoden stehen. Dabei gilt weiterhin, dass der Lock nur einmal pro Objekt existiert. Das Beispiel von vorhin erweitert:

```java
class BankAccount {
  private int balance = 0;
  
  public synchronized void deposit(int amount) {
    this.balance += amount;
  }
  
  public synchronized boolean withdraw(int amount) {
    if(amount > this.balance) {
      this.balance -= amount;
      return true
    } else { // keine Kreditbank
      return false;
    }
  }
}
```



In diesem Code schliessen sich zwei gleichzeitige Aufrufe von ``deposit()`` und ``withdraw()`` in jeglichen Kombinationen aus.

### Funktion vs. Block

``synchronized`` kann auch in der Block-Variante verwendet werden. Dort kann angegeben werden, welches Objekt gelockt werden soll. Es ist dann auch nicht die ganze Methode gelockt, sondern nur dieser Block:

```java
public void deposit(int amount) {
  synchronized(this) {
    this.balance += amount;
  }
}
```

#### Äquivalenzen

```java
public class Test {
  synchronied void f() {...} // Object Lock
  static synchronized void g() {...} // Class Lock
}
```

$$
\iff
$$

```java
public class Test {
  void f() {
    synchronized(this) {...} // Object Lock
  }
  static void g() {
    synchronized(Test.class) {...} // Class Lock
  }
}
```

### Exit aus Synchronzed Block

Der Lock wird bei jedem Exit freigegeben

* Ende des Blocks
* Return-Statements
* Unbehandelte Exception

### Rekursive Locks

Gleicher Thread kann denselben Lock mehrfach beziehen, z.B. durch geschachtelte Aufrufe. Der Lock wird erst beim letzten Release freigegeben. Realisiert wird dies intern durch einen Counter. Beispiel:

```java
synchronized void limitedDeposit(int amount) {
  if(amount + balance <= limit) {
    deposit(amount)
  }
}
synchronized void deposit(int amount) {...}
```

## Monitor - Warten auf Bedingungen

Angenommen, unsere ``withdraw()``-Methode wäre gutgläubig und geduldig. Anstatt einen Fehler zurückzugeben, wartet sie ab, bis genügend Geld vorhanden ist.

```java
public synchronized void withdraw(int amount) throws InterruptedException {
  while (amount > this.balance) {
    Thread.sleep(1);
  }
  this.balance -= amount;
}
```

Das Problem ist, dass dieser Ansatz sehr naiv ist. Wenn nun eine Methode im ``withdraw()`` wartet, hat sie ja bereits den Lock des Objekts - das bedeutet, niemand kann die ``deposit()``-Methode aufrufen. Es kann also gar nie Geld ankommen.

``sleep()`` und ``yield()`` geben den Lock nicht frei. Zudem ist das Pollen in Zeitabständen ineffizient. Was lernen wir daraus? 

> ``synchronized`` ist nicht für Wartebedingungen.

Wir benötigen den *Wait & Signal*-Mechanismus des Monitors. 

### Monitor

Ein Monitor ist ein Objekt mit internem, gegenseitigen Ausschluss. Nur ein Thread operiert zur gleichen Zeit im Monitor. Nicht-private Methoden sind alle ``synchronized``, alle Variablen privat.

Im Wait & Signal-Mechanismus können Threads im Monitor auf Bedingung warten. Threads können die Bedingung signalisieren und wartende Threads aufwecken.

![F40951E8-DECF-486F-B4FF-C976C8911563](Bilder/F40951E8-DECF-486F-B4FF-C976C8911563.png)

### Beispiel

```java
class BankAccount {
  public int balance = 0;
  
  public synchronized void withdraw(int amount) throws InterruptedException {
    while (amount > balance) {
      wait(); // warte bis geweckt und prüfe erneut
    }
    balance -= amount;
  }
  
  public synchronized void deposit(int amount) {
    balance += amount;
    notifyAll(); // wecke alle
  }
}
```

### Details

Wieso funktioniert das? ``wait`` gibt den Monitor-Lock temporär frei, damit ein anderer Thread die Bedingung im Monitor erfüllen kann.

``wait``macht:

1. In Warteraum gehen
2. Monitor freigeben
3. (Inaktiv bis zum Wecksignal)
4. Monitor neu beziehen

Wecksignal: signalisieren einer Bedingung im Monitor

* ``notify()``weckt *einen* **beliebigen** wartenden Thread im Monitor
* ``notifyAll()`` weckt alle im Monitor wartende Threads
* kein Effekt, falls kein Thread wartet

Beim Aufruf dieser Methoden: *Signal and continue*

1. weckt man (alle) Thread(s) in ``wait``
2. Behält man den Monitor und macht weiter

Für den Moment kennen wir nur pauschales Wait & Signal. Ich muss selber schauen, ob nach dem Signal die Bedingung nun erfüllt ist.

``wait()``, ``notify()`` und ``notifyAll()`` sind nur im ``synchronized``-Block verwendbar, ansonsten gibt es eine ``IllegalMonitorStateException``. Bei ``wait`` mit rekursiven Locks, werden alle gehaltenen Locks auf diesem Obkekt temporär freigegeben.

Gründe zum Aufwachen aus ``wait()``: 

* ``notifyAll()``, ``notify()``
* InterruptedException
* Spurious Wakeup (eher akademisch)

### Funktionsweise des Monitors

Der Monitor unterscheidet den äusseren und den inneren Warteraum. Ein Thread kommt erstmal am äusseren Warteraum an, bis er den Monitor betreten kann (den Lock besetzen). Falls ein Thread den Lock besitzt, ist er im Monitor. Ruft er dprt ``wait`` auf, wandert er in den inneren Warteraum und gibt den Lock frei. Nun kann ein anderer Thread den Lock besetzen und potentiell die Wartenden benachrichtigen (im inneren Warteraum). Diese wandern dann in den äusseren Warteraum, wo sie (wie alle anderen Threads) auf den Wiedereintritt in den Monitor warten.

![5803F5B5-C973-4540-9148-C794A1DD799E](Bilder/5803F5B5-C973-4540-9148-C794A1DD799E.png)

### Typische Fallen/Antipattern (bei Wartebedingungen)

**Bedingung prüfen mit ``if``:** das Signal sagt nur, dass man nun nochmals prüfen soll, nicht dass es nun erfüllt ist. Darum:

> Immer mit ``while`` prüfen

**Bei Wartebedingung nur mit ``notify()`` signalisieren:** es kann sein, dass sich für beide Bedingungen eines Werts (z.B. *nicht leer* und *nicht voll*) mehrere Threads anstauen. ``notify()`` weckt aber nur *irgendeinen* Thread auf. Falls dieser Thread auch auf irgendeine Bedingung wartet, wird kein anderer Thread aufgeweckt, der weitermachen kann.

> Immer ``notifyAll()`` verwenden

### Fairness-Probleme

In Java gibt es keine garantierte FIFO-Warteschlange. Einige Threads könnten so nie drankommen. Bei Signal-and-Continue kommen aufgeweckte Threads in den äusseren Warteraum und können kontinuierlich überholt werden. Es ist aber auch bei FIFO-Warteschlangen (.NET) nicht garantiert fair, da *hinten* eingereiht wird (es kommen erst die bereits aussen wartenden Threads dran)
