---
typora-copy-images-to: ./Bilder
---

# Memory Models

## Lock-freie Programmierung

* Korrekte nebenläufige Interaktionen ohne Locks
* Garantien des Speichermodell nutzen

Einstiegsbeispiel:

![C4AAE602-0FFE-4326-BAD3-040758CD2605](Bilder/C4AAE602-0FFE-4326-BAD3-040758CD2605.png)

(Also: man setzt x = y = 0; DANN lässt man zwei Threads loslaufen)

### Ursachen für Probleme

* Weak Consistency
  * Speicherzugriffe werden in verschiedenen Reihenfolgen auf verschiedenen Threads gesehen
  * Ausnahme: Synchronisation/Speicherbarrieren
* Optimierungen
  * Compiler, Laufzeitsystem und CPUs
  * Instruktionen werden umgeordnet, wegoptimiert

Keine sequentielle Konsistenz bei Nebenläufigkeit!

Die CPU darf Instruktionen eines Threads umordnen, sofern sie erkennt dass sie nicht voneinander abhängen. Wenn ein zweiter Thread davon abhängig ist in welcher Reihenfolge das geschieht, ist das der CPU egal. Im Beispiel oben darf der Thread 2 auch "umgekehrt" ablaufen.

## Memory Model

### Java

Minimale spezifizierte Garantien

* Atomicity (Unteilbarkeit)
* Visibility (Sichtbarkeit)
* Ordering (Reihenfolge)

### Atomicity

Garantien:

Zugriff auf Variable Lesen/schreiben ist atomar für

* Primitive Datentypen bis 32 Bit
* Objekt-Referenzen
* long und double nur mit `volatile` Keyword atomar

Achtung: Unteilbarkeit heisst nicht Sichtbarkeit

* Nach Write sieht ein anderer Thread evtl. noch anderen Wert
* Es heisst nur: immer gültigen Wert; entweder alten oder neuen

Atomar oder nicht?

TODO: genauer analysieren

![DC998850-FB5F-47DF-A831-B8401522C8AA](Bilder/DC998850-FB5F-47DF-A831-B8401522C8AA.png)

### Visibility

Analysebeispiel

```java
class Worker extends Thread {
  private boolean doRun = true;
  
  public void run() {
    while (doRun) {
      ...
    }
  }
  
  public void stopRequest() {
    doRun = false;
  }
}
```

Problem:

Sehe Änderungen eines anderen Threads eventuell nicht oder viel später

Optimierung, z.B. VM hält Variablenwert in Register

```
void run() {
  load doRun to reg1;
  while(reg1) {
    ... (Endlosschleife)
  }
}
```

Garantien: Die Sichtbarkeit ist garantiert bei

* Locks Release & Acquire
  * Änderungen vor Release werden bei Acquire sichtbar
* Volatile Variable
  * Zugriff macht Änderungen anderer Zugreifer sichtbar
* Initialisierung von final Variablen
  * Nach Ende des Konstruktors
* Thread-Start und Join
  * ebenso Task Start und Ende

### Sichtbarkeit mit volatile in Java

Alle Änderungen vor dem volatile Zugriff werden für jeden sichtbar der danach auf dieselbe volatile-Variable zugreift (damit würde man das Problem oben lösen). **Nur so in Java, anders in .NET und C/C++**!

Wo ist die Sichtbarkeit von x == 1 garantiert?

![811359D9-0FEC-48C5-9C8C-B0EDCD45F44B](Bilder/811359D9-0FEC-48C5-9C8C-B0EDCD45F44B.png)

### Java Volatile Keyword

* Atomicity
  * Atomares Lesen und Schreiben auch für long und double
  * Achtung: andere Operationen sind nicht atomar (z.B. `i++`)
* Visibility
  * Änderungen werden anderen Zugreifern propagiert
  * Achtung: kein Sperren im Gegensatz zu Locks
* Reordering
  * Keine Umordnung durch Compiler/Laufzeitsystem/CPU
  * Achtung: nicht volatile Variablen werden evtl. umgeordnet

Volatile verhindert Data Race auf Variable

### Ordering

Java Garantien:

* Innerhalb eines Threads: "As-if-Serial" Semantik
  * Sequentielles Verhalten innerhalb eines Threads bleibt
* Zwischen Threads: Reihenfolge nur erhalten für
  * Synchronisationsbefehle
  * Zugriffe auf volatile Variablen
* Keine Umordnung über Synchronisation oder volatile Zugriffe hinweg
  * Memory Barriers / Memory Fences

Welche Umordnungen sind möglich?

![7A8EF8E5-06A0-465A-9A7E-B280A2A65D86](Bilder/7A8EF8E5-06A0-465A-9A7E-B280A2A65D86.png)

### .NET

Unterschied zu Java Memory Model

* Atomarität: long/double nicht mit volatile atomar
* Visibility nicht definiert. Implizit durch Ordering
* Ordering: volatile ist nur partielle Fence

Atomare Instruktionen

* `Interlocked` Klasse

### .NET Volatile Read/Write Fences

* Volatile Read: Acquire Semantik
  * bleibt vor den nachfolgenden Zugriffen
* Volatile Write: Release Semantik
  * bleibt nach den vorherigen Zugriffen

![922A1CFA-CBA1-4DE2-A624-228A76D112AD](Bilder/922A1CFA-CBA1-4DE2-A624-228A76D112AD.png)

### .NET Full Fence

Umordnung in beide Richtungen verbieten: `Thread.MemoryBarrier();`

## Atomare Operationen

* Kein Blockieren oder Warten auf Locks
* Komplexer als nur atomares Lesen und Schreiben
* Java Atomic Variables
* Effizient: atomare Instruktionen des Prozessors
* Garantiert auch Visibility und Ordering

### Eigener Spin-Lock-Yield

Idee: wenn es nicht geht, `yield`en wir

```java
public class SpinLock {
  private volatile boolean locked = false;
  
  public void acquire() {
    
    // Block müsste atomar sein
    while(locked) {
      Thread.yield();
    }
    locked = true;
    
  }
  
  public void release() {
    locked = false;
  }
}
```

Mit atomarer Operation

```java
public class SpinLock {
  private AtomicBoolean boolean locked = new AtomicBoolean(false)
  
  public void acquire() {
    while(locked.getAndSet(true)) { 
      // ^ lese alten Werte und setze neuen Wert atomar Rückgabe = gelesener Wert
      Thread.yield();
    }
  }
  
  public void release() {
    locked.set(false);
  }
}
```

Das Ganze ist etwas tricky zu verstehen:

Angenommen, das Lock sei zu (locked = true) und man macht acquire(), dann wird

1. das Lock auf true gesetzt (bleibt true)
2. Ge-yield()-et (man macht also nichts)

Angenommen, das Lock sei offen (locked = false) und man macht acquire(), dann wird

1. Das Lock auf True gesetzt (es ist jetzt zu)
2. nicht ge-yield-et, man macht also weiter

### Atomares Compare and Set

`boolean CompareAndSet(boolean expect, boolean update)`

* Setzt `update` wenn alter Wert gleich `expect` ist (atomar)
* Retourniert `true` bei erfolgeichem Update

Atomar geschieht das folgende:

```java
if (current == expect) {
  current = update;
  return true;
} else {
  return false;
}
```

### Atomic Klassen

Klassen für Boolean, Integer, Long und Referenzen (auch für Array-Elemente)

Diverse atomare Operatoren

* addAndGet(), getAndAdd(), etc.

* Ab Java 8 sogar mit Lambda

* ```java
  AtomicInteger counter = new AtomicInteger(0);
  counter.updateAndGet( x -> x + 1);
  ```

### Optimistische Synchronisation

```java
do {
  oldValue = var.get(); // lese aktuellen Wert
  newValue = calculateChanges(oldValue);
} while (!var.compareAndSet(oldValue, newValue))
  // ^ schreibe, falls gelesener Wert immer noch aktuell ist
```

Hier kommen wir allerdings zum ABA-Problem

### ABA-Problem

Anderer Thread schreibt unbemerkt dazwischen

![747C4D63-98D9-4A39-A7CB-3B3B41765B06](Bilder/747C4D63-98D9-4A39-A7CB-3B3B41765B06.png)

## Lock-freie Datenstrukturen

Lock-Free Stack

```java
AtomicReference<Node<T>> top = new AtomicReference<>();
...
  
void push(T value) {
  Node<T> newNode = new Node<>(value);
  Node<T> current;
  do {
    current = top.get();
    newNode.setNext(current);
  } while (!top.compareAndSet(current, newNode));
}
```



![C967D903-1F84-49DC-B56D-9E89A645F34D](Bilder/C967D903-1F84-49DC-B56D-9E89A645F34D.png)

Vorgefertigte Lock-freie Datenstrukturen

* `ConcurrentLinkedQueue<V>`
* `ConcurrentLinkedDeque<V>`
* `ConcurrentSkipListSet<V>`
* `ConcurrentHashMap<K, V>`
* `ConcurrentSkipListMap<K, V>`

# Actor Model

## Motivation

Herkömmliche Programmiersprachen sind nicht für Nebenläufigkeit entworfen

* Optimiert für sequentielle/prozedurale Ausführung
* Nebenläufigkeit/Threads sind "Second Class" Features
* Oder gar nicht vorhanden: JavaScript
* By Default: Speicher nicht thread-safe

Korrekte nebenläufige Programme zu schreiben ist daher besonders schwierig

Threads operieren auf Modell von passiven Objekten

Konsequenzen:

* Beschränkter Grad an Nebenläufigkeit
  * Concurrency sehr selektiv eingesetzt
  * Maschinenorientierte Concurrency für Performance
  * Viele unnötlg blockierende Aufrufe
  * Resultat: no free lunch
* Fehleranfälligkeit
  * Race Conditions vorprogrammiert
* Sehr schlecht verteilbar
  * Shared Memory Modell
  * Threads operieren auf gemeinsamem Speicher

## Actor Modell und CSP

Substantiell anderes Programmierkonzept

* Aktive Objekte
  * Objekte haben nebenläufiges Innenleben
* Kommunikation
  * Objekte senden und empfangen Nachrichten
* Kein Shared Memory
  * Nur Austausch von Nachrichten über Kanäle/Mailboxen

## Actor-Modell

Ein Actor kann

* Neue Actors erstellen
* Nachrichten an Actors (andere und sich selber) senden
* Entscheiden, wie die nächste Nachricht behandelt werden soll (Zustandsänderung)

## CSP (Communicating Sequential Processes)

* Prozesse kommunizieren indirekt über Channels miteinander
* Austausch der Nachrichten erfolgt unmittelbar und synchron
* Inspirierte die Entwicklung von Go

Prinzipiell gleiches Modell wie Actors

* Unterschied: Actor hat keine Channels, Senden ist immer asynchron, keine garantierte Reihenfolge des Empfangs

![FCB1BFB8-CF23-4158-9125-71AF1913F855](Bilder/FCB1BFB8-CF23-4158-9125-71AF1913F855.png)

## Klassisch vs. CSP

![064BAD9E-CCC0-4C14-9021-F929F0CB0D69](Bilder/064BAD9E-CCC0-4C14-9021-F929F0CB0D69.png)

## Vorteile: Actor, CSP

* Inhärente Nebenläufigkeit
  * Alle Objekte (Actors) laufen nebenläufig
  * Maschine kann Grad an Nebenläufigkeit ausnutzen
* Keine Race Conditions
  * Kein Shared Memory
  * Nachrichtenaustausch synchronisiert implizit
* Gute Verteilbarkeit
  * Kein Shared Memory
  * Nachrichtenaustausch für Netz prädestiniert

## Akka

Actor-Modell implementiert in Skala, zusätzliches Java API

### Konzept

* Actor sind aktive Objekte
  * laufen konzeptionell nebenläufig zueinander
* Privater Zustand
  * aufpassen, dasss per Java Referenzen kein Shared State entsteht
* Eine Mailbox pro Actor
  * ein Buffer für alle Nachrichten, die zu ihm ankommen
  * asynchrones Senden

### Empfangsverhalten

* Reaktion auf ankommende Nachricht
  * Spezielle Behandlungs-Methode wird ausgeführt (ähnlich wie Event Queue bei GUIs)
* Effekte per Behandlung
  * ändere privaten Zustand
  * Sende Nachrichten
  * erzeuge neue Actors
* Intern sequentiell
  * nur eine Nachricht auf einmal bedienbar

### Einfacher Akka-Actor

Empfangsmethode: Run to Completion pro eingehende Nachricht

```java
public class NumberPrinter extends UntypedActor {
  public void onReceive(final Object message) {
    if (message instanceof Integer) {
      System.out.print(message);
    }
  }
}
```

### Erzeugen und Senden

```java
ActorSystem system = ActorSystem.create("System");
ActorRef printer = system.actorOf(Props.create(NumberPrinter.class)); // Erzeugung per Reflection

for (int i = 0; i < 100; i++) {
  printer.tell(i, actorRef.noSender()) // einfaches, asynchrones Senden
}

system.shutdown(); // Gebe End-Signal an alle Actors
```

### Actor-Referenzen

* `ActorRef` Adresse eines Actors
  * Actor bei Fehlverhalten neu startbar (behält Adresse)
* Entkopplung von "Interface" und Instanz
  * Vorbereitet für Verteilung
  * ActorRef ist immutable (in Message verschickbar)
* Verhindert Methodenaufrufe/Variablenzugriffe
  * Reiner Nachrichtenaustausch

### Verteilte Actors-Client (Producer/Consumer)

```java
ActorSystem system = ActorSystem.create("producer");

ActorSelection printer = system.actorSelection(
	"appa.tcp://System@server:2552/user/printer");

printer.tell(123, ActorRef.noSender());
```

### Actor Remoting	

* Remote Lookup
  * `system.actorSelection` mit URL
* ActorSelection
  * leichtgewichtiger als ActoRef
  * kann 0..n Actors umfassen
  * kann zu ActorRef aufgelöst werden
* Remote erzeugen
  * `system.actorOf(…)`
  * `application.conf` spezifiziert wo Actor erstellt wird
  * Keine Codeänderungen!

### Actor Hierarchies

* Hierarchie von Actors
  * Passend zu URL-Adressierungsschema
  * Supervision von Actors (später)
  * Erzeuger ist Parent
* ActorSelection
  * Selektiert Teilbaum
  * Broadcast möglich

### Akka Sender

`tell(msg, sender)`

* Sender der Message wird mitgegeben
* Nützlich bei Antwort an Sender
* Typischerweise getSelf() bzw getSender() bei Forward

Achtung: getSender zeigt immer auf den ursprünglichen Sender, auch beim Forwarding (wenn es mit dem entsprechenden Absender weitergeleitet wurde)

### Akka Synchrones Senden

Actor-Modell ist asynchron

Synchrones Senden-Empfangen möglich mit Futures

`Future<Object> resut = Patterns.ask(actorRef, msg, timeout);`

### Akka Messages

Serializable Classes

Immutable (Value Objects)

* Attribute final
* Collections in `Collections.unmodifiableList` wrappen
* keine Methoden die Seiteneffekte haben

Typischerweise einfache Wrapper-Klassen mit Fokus auf die Attribute

Messages sind viel Schreibaufwand, besser mit der Scala API

### Akka Laufzeitsystem

* Akka verwendet Dispatcher zur Ausführung
* Typischerweise ein Java Fork-Join Thread Pool
* Nicht ein Thread pro Actor!

Bei synchronem Send & Receive Warteabhängigkeiten zwischen Actors

* Thread Pool: keine Lösung (Deadlock bei fixer Anzahl)
* Muss Stack pro Actor instanzieren
* Thread pro Actor => sehr viele Threads

Synchrones Send & Receive daher nicht empfohlen

### Akka Supervision

* Actors können andere Actors überwachen
* Bei Exception wird der Supervisor benachrichtigt
* Parents überwachen by default ihre Kinder

Der Supervisor kann je nach Fehler reagieren. Achtung: Nicht für Programmierfehler, sondern nur für externe Ursachen (Netzwerk, Files), analog Exception vs. Error.

| Resume   | Child macht weiter (behält internen Zustand) |
| -------- | ---------------------------------------- |
| Restart  | Child wird neu gestartet (verliert Zustand) |
| Stop     | Child wird nicht mehr ausgeführt         |
| Escalate | Supervisor gibt auf und meldet selber seinem Supervisor einen Fehler |

Parent von `/user` ist der Root Guardian, zusätzlicher `/system` Actor für Logging und Shutdown

### System Shutdown

Applikation muss Actors selber stoppen

```java
getContext().stop(actorRef); // stoppt nach Bearbeitung der aktuellen Message (Massage)

getContext().stop(getSelf()); // immer rekursiv (?)
getContext().system().terminate();

actor.tell(PoisonPill.getInstance(), sender); // stoppt bei Behandlung der Poison Pill
victim.tell(Kill.getInstance(), sender); // startet Supervision Behandlung
```

### Schwächen

* Protokoll der Actors nur implizit vorhanden
  * Formales Protokoll fehlt
  * Grundsätzliches Problem bei Actors im Gegensatz zur CSP
* Akka: keine Typsicherheit
* Akka: Diskrepanz JVM und Actor Model
  * leicht verletzbare Regeln => Laufzeitfehler

# GPU-Parallelisierung I

GPU ist ein Co-Prozessor des Systems

* CPUs bieten wenig Cores
  * Meist 4, 8, 16, 20
  * Allgemeine, schnelle Prozessore
* GPUs bieten sehr viele Cores
  * 512, 1024, 2880, 5760 Cores
  * Sehr spezifische, langsamere Prozessoren

Ziel: GPU Many Cores für Parallelisierung nutzen

## GPU

GPUs: hohe FLOPS Performance (Floating Point Operations per Second)

Braucht hohe Bandbreite

GPUs verwenden mehr Transisforen für Recheneinheiten

![3C7523D3-0407-45CE-A6C1-EC8DB39365F8](Bilder/3C7523D3-0407-45CE-A6C1-EC8DB39365F8.png)

## Aufbau

![39428029-5D27-4C2A-89C6-AF565E6953C1](Bilder/39428029-5D27-4C2A-89C6-AF565E6953C1.png)

Zum Beispiel 1-30 SM

8-192SP pro SM

## SIMD

Streaming Multiprocessor ist primzipiell SIMD (Single Instruction Multiple Data)

Cores führen dieselbe Instruktion auf unterschiedlichen Daten/Speicherstellen aus

![FDC45B6B-1264-4F58-8BA0-8088F8740090](Bilder/FDC45B6B-1264-4F58-8BA0-8088F8740090.png)

SIMD = Vektorparallelität

![6C5497C7-B7F9-4B93-B2A8-3A8B25FD7987](Bilder/6C5497C7-B7F9-4B93-B2A8-3A8B25FD7987.png)

Cores innerhalb eines SM sind nur mit Vektor-Parallelisierung effizient nutzbar

* Alle Cores führen die gleiche Instruktion aus
* einzelne können auch die Instruktion nicht ausführen (aber nicht eine andere)

## CPU vs GPU

* GPU: Video Gaming

  * Extrem hohe Datenparallelität
  * Wenig Verzweigungen
  * kein beliebiges Warten bei Parallelität
  * Einfache, aber viele Cores
  * Kleine Caches pro Core

  Ziel: hoher Gesamtdurchsatz

* CPU: General Purpose

  * Niedrige Datenparallelität
  * Viel Verzweigungen
  * Beliebige Thread-Synchronisation
  * Wenige, aber mächtige Cores
  * Grössere Caches in Chip

  Ziel: niedrige Latenz pro Thread

## NUMA-Modell

Non-Uniform Memory Access

* kein gemeinsamer Hauptspeicher zwischen GPU und CPU
  * explizites Übertragen
* Unterschiedlicher Instruktionssatz/Architektur
  * Code für GPU kompilieren/designen

## CUDA

Proprietär für Nvidia-Karten, API/Compiler für C/C++

Computer Unified Device Architecture

General Purpose Programming Model

* Parallelisierung mit sehr vielen (vektorparallelen) Threads
* Video-Computing und CUDA teilen sich GPUs
* API und Compiler für Programmiersprache C/C++

## Vektoraddition

Sequentiell:

```c
void VectorAdd(float *A, float *B, float *C, int N)  {
  for (int i = 0; i < N; i++) {
    C[i] = A[i] + B[i]; 
  }
}

Parallelisieren: pro Thread C[i] = ...
```

CUDA

```c
// kernel definition, GPU (Device)
__global__
void VectorAddKernel(float *A, float *B, float *C) {
  int i = threadIdx.x;
  C[i] = A[i] + B[i];
}

// CPU (Host)
int main() {
  // ...
  // kernel invocation
  VectorAddKernel<<<1, N>>>(A, B, C);
  // ...
}
```

der gleiche Kernel wird von mehreren Threads ausgeführt.

`threadIdx.x` => Thread-ID (0..N-1)

## SIMT: Single Instruction Multiple Threads

![93E390E8-AAA6-46F4-B8D3-D88475DF3F79](Bilder/93E390E8-AAA6-46F4-B8D3-D88475DF3F79.png)

## CUDA Blocks

Threads sind in Blöcke gruppiert

* Ein Block => gleicher SM
* wird im Programmiermodell sichtbar
* Threads können *innerhalb Block* interagieren

## CUDA Ausführungsmodell

* Thread = virtueller Skalarprozessor
* Block = virtueller Multiprozessor
* Blöcke müssen unabhängig sein
  * Run to Completion
  * Beliebige Ausführungsreihenfolge
    * sequentiell oder parallel
  * Blocks in Thread Pool (Thread Execution Manager)

## CUDA Thread Pool Abstraktion

* Grad der Parallelität durch GPU bestimmt


* Automatische Skalierung

![7A9C66DA-E49B-475E-A268-7E99282DD24D](Bilder/7A9C66DA-E49B-475E-A268-7E99282DD24D.png)

## Thread Hierarchie

* Kernel wird auf *Grid* ausgeführt
  * Blöcke mit ID
    * Thread mit ID

![5780CC24-E7CA-4A02-850B-208121684018](Bilder/5780CC24-E7CA-4A02-850B-208121684018.png)

## Datenaufteilung

Jede Kernel-Ausführung bestimmt seinen Datenteil

* `threadIdx.x` Nummer des Threads innerhalb Block
* `blockIdx.x` Nummer des Blocks
* `blockDim.x` Blockgrösse
* Weitere Dimensionen y, z nutzbar
* Der Programmierer modelliert die Datenaufteilung selber

![B7C98FCB-58D3-4232-9C6E-B4F0FEDD06AD](Bilder/B7C98FCB-58D3-4232-9C6E-B4F0FEDD06AD.png)

## Aufteilung in Blöcke

Rechnung:

* `blockIdx.x * blockDim.x` eigenen Block finden
* `+ threadIdx.x` Offset innerhalb Block

```c
__global__
void VectorAddKernel(float *A, float *B, float *C, int N) {
  int i = blockIdx.x * blockDim.x + threadIdx.x;
  C[i] = A[i] + B[i];
}

// kernel invocation
VectorAddKernel<<<4, 512>>>(A, B, C, 2048) // 2048 = 4 * 512
```

`VectorAddKernel<<<BLÖCKE, THREADS>>>`

Hier haben wir aber noch ein Problem: wir haben jetzt die Anzahl Blöcke und die Anzahl Threads selber fix bestimmt. So ist das nicht sehr dynamisch.

## Allgemeinere Aufteilung

Um das dynamisch zu machen verwenden wir das folgende. Hier ist die Bedingung wichtig, sonst schreiben wir über Memory hinaus.

```c
__global__
void VectorAddKernel(float *A, float *B, float *C, int N) {
  int i = blockIdx.x * blockDim.x + threadIdx.x;
  if (i < N) {
	  C[i] = A[i] + B[i];    
  }
}

int blockSize = 512; // Anzahl Threads pro Block
int gridSize = (N + blockSize - 1) / blockSize; // Anzahl Blöcke pro Grid
VectorAddKernel<<<4, 512>>>(A, B, C, 2048) // 2048 = 4 * 512
```

Zur Rechnung von gridSize:

Angenommen, $N$ ist 512 (512 Elemente), und man hat eine $blockSize$  von 512. Intuitiv reicht ja hier ein Block.

Wenn man nun aber $\frac{N + blockSize}{blockSize}$ rechnet, erhält man 1024 / 512 = 2 Blöcke.

Mit dem -1 stellt man sicher, dass der Nenner so rutscht, dass man mit der Teilung und Integer-Abrundung auf den richtigen Wert kommt.

Wichtig: `blockDim` im Kernel entspricht der `blockSize`.

## CUDA Ausführung

1. Auf GPU allozieren
   `cudaMalloc` 
2. Daten zu GPU transferieren
   `cudaMemCpy(HostToDevice)`
3. Kernel ausführen
   `<<<gridId, blockId>>>`
4. Daten von GPU zu CPU transferieren
   `cudaMemcpy(DeviceToHost)`
5. Auf GPU deallozieren
   `cudaFree`

## Programmgerüst

```c
void CudaVectorAdd(float* A, float* B, float* C, int N) { 
  size_t size = N * sizeof(float);
  float *d_A, *d_B, *d_C;
  
  cudaMalloc(&d_A, size);
  cudaMalloc(&d_B, size);
  cudaMalloc(&d_C, size);

  cudaMemcpy(d_A, A, size, cudaMemcpyHostToDevice);
  cudaMemcpy(d_B, B, size, cudaMemcpyHostToDevice);
  
  int blockSize = 512;
  int gridSize = (N + blockSize - 1) / blockSize;
  VectorAddKernel<<<gridSize, blockSize>>>(d_A, d_B, d_C, N);
  
  cudaMemcpy(C, d_C, size, cudaMemcpyDeviceToHost);
  
  cudaFree(d_A);
  cudaFree(d_B);
  cudaFree(d_C);
}
```

## Speicher-Funktionen

* `cudaMalloc`
  * Alloziert Objekte in Device Global Memory
  * Parameter: Pointer-Adresse, Size (Bytes)
* `cudaFree`
  * Dealloziert Objekte in Device Global Memory
  * Parameter: Pointer-Adresse
* `cudaMemcpy`
  * Kopiert Speicher zwischen CPU/GPU
  * Target Pointer, Source Pointer, Size, Destination (zu oder von GPU)

## Fehlerbehandlung

Return von CUDA-Funktion beachten. Fehler falls Code != `cudaSuccess`

Am besten Hilfsfunktion verwenden oder schreiben, und um jeden Methodenaufruf wrappen

## Verbessertes Gerüst

```c
void handleCudaError(cudaError error) { 
  if (error != cudaSuccess) {
  	fprintf(stderr, "CUDA: %s!\n",
    	cudaGetErrorString(error));
  	exit(EXIT_FAILURE); 
  }
}

handleCudaError(cudaMalloc(&d_A, size));
handleCudaError(cudaMalloc(&d_B, size)); 
handleCudaError(cudaMalloc(&d_C, size));

handleCudaError(cudaMemcpy(d_A, A, size, cudaMemcpyHostToDevice)); handleCudaError(cudaMemcpy(d_B, B, size, cudaMemcpyHostToDevice));

int blockSize = 512, gridSize = (N + blockSize - 1) / blockSize; VectorAddKernel<<<gridSize, blockSize>>>(d_A, d_B, d_C, N); handleCudaError(cudaGetLastError());

handleCudaError(cudaMemcpy(C, d_C, size, cudaMemcpyDeviceToHost));

handleCudaError(cudaFree(d_A));
handleCudaError(cudaFree(d_B));
handleCudaError(cudaFree(d_C));
```

## Unified Memory seit CUDA 6

Automatischer Transfer CPU <==> GPU

Keine expliziten Memory Copies mehr

Dafür andere Regeln

Siehe V10 S. 39

# GPU Parallelisierung II

## Launch Configuration

* Maximale Anzahl Threads per Block
  * Abhängig von GPU, z.B. 512 oder 1024
* Blockgrösse als Vielfaches von 32 (sonst ineffizient)
* Überflüssige Threads vermeiden
  * 2 Blöcke à 1024 Threads, wenn man 1500 braucht: 548 unnütze Threads
* Streaming Multiprocessor ausschöpfen
  * Limite für Resident Blöcke und Threads, z.B. 8 und 1536
* Grosse Blockgrösse hat Vorteile
  * Threads können nur in Block interagieren

## Resident Threads & Blocks

Kennzahl der GPU

* Anzahl Blöcke/Threads die in GPU geladen sein können
  * Davon sind zu einer Zeit nur wenige effektiv in Ausführung
  * Falls eine Berechnung blockiert, wechselt die GPU zu anderen Resident Block/Thread
* Analoges Konzept wie CPU Hyperthreading

### Beispielrechnung

Wir wollen einen Vektor mit **1500** Elementen parallel bearbeiten

Wir haben eine Grafikkarte mit folgenden Kennzeichen

* maximale Anzahl *Threads per Block*: 1024
* maximale Anzahl *Resident Blocks*: 8
* maximale Anzahl *Resident Threads*: 1536

![427767B6-C620-4B87-9DA3-AD37F5B36893](Bilder/427767B6-C620-4B87-9DA3-AD37F5B36893.png)

Probleme:

1. Limite bei Resident Blocks: 8 * 128 = 1024
2. Limite bei Resident Blocks: 8 * 160 = 1280

Die anderen sind alle Ok, ausser dem Letzten. Limite bei Threads per Block: Hier bringen wir nur 1024 Threads in den Block, nicht die 1500 benötigten.

Von denen, bei denen "Threads in SM" = alle ist:

1. Priorität: am wenigsten unnütze Threads
2. Priorität: am meisten Threads per Block

## 3D Thread Hierarchie

(x, y, z) für `threadIdx` und `blockIdx`. 

* 1D falls y, z ungenutzt
* 2D falls z ungenutzt

![141F1ECB-B01B-4CB8-8C6E-E7AE2322A631](Bilder/141F1ECB-B01B-4CB8-8C6E-E7AE2322A631.png)

## C Limitation

Mehrdimensioale Arrays nicht direkt unterstützt



```c
float *matrix = 
  (float *)malloc(NofRows * NofCols * sizeof(float));
matrix[row * NOfCols + Col]
```

## Matrix Multiplikation

Sequentiell


$$
C_{i,j} = \sum_{0}^{K-+}A{i,k} * B_{k, j}
$$

```c
float sum = 0;
for (int k = 0; k < K; k++) {
  sum += A[i, k] * B[k, j];
}
C[i, j] = sum;
```

