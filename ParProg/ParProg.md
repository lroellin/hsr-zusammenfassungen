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

Die JVM läuft, solange Threads laufen Ausnahme hierzu sind Threads, die sich als Daemon markiert haben,auf diese wartet Java nicht (wie z.B. der Garbage Collector). Daemon-Threads brechen bei JVM-Ende unkontrolliert ab. Die JVM kann andererseits mit `System.exit()/Runtime.exit()` direkt terminiert werden, was aber unsauber ist. Alle Threads werden unkontrolliert abgebrochen.

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

Jetzt kann man aber gleich weitergehen und dies mit einem Lambda abhandeln. Wir implementieren das Runnable-Interface mit run():

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

## InterruptedException

Ist eine mögliche Exception bei blockierenden Aufrufen wie `sleep()` oder `join()`. Die Threads können von aussen mit `myThread.interrupt()` unterbrochen werden!

`interrupt()` ist für kooperatives Canceling gedacht. Nur verwenden, wenn die Cancel-Policy des Threads bekannt ist!

Sie wird oft zum Aufbrechen von Blockaden missbraucht. Dabei hinterlässt sie meist inkonsistente Zwischen-zustände, oder die Exception wird ignoriert und es wird weiter blockiert.

## Weitere Thread-Methoden

* ``staticThreadcurrentThread()`` Gerade ausführendeThread-Instanz
* `void setDaemon(boolean on)` Thread als Daemon markieren, standardmässig false
* `String getName(),void setName(String name)` Thread-Name einstellen.
  * Beim Konstruktor `Thread(String name), Thread(Runnable r, String name)`
  * Für diagnostische Ausgaben: Default: Thread-0, Thread-1, ...
