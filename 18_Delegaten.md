<!--

author:   Sebastian Zug & André Dietrich
email:    zug@ovgu.de   & andre.dietrich@ovgu.de
version:  0.0.1
language: de
narrator: Deutsch Female

import: https://raw.githubusercontent.com/LiaTemplates/Rextester/master/README.md
import: https://raw.githubusercontent.com/LiaTemplates/WebDev/master/README.md
-->

# Vorlesung Softwareentwicklung - 18 - Delegaten

--------------------------------------------------------------------
Link auf die aktuelle Vorlesung im Versionsmanagementsystem GitHub

https://github.com/liaScript/CsharpCourse/blob/master/18_Delegaten.md

Die interaktive Form ist unter diese Link zu finden ->
[LiaScript Vorlesung 18](https://liascript.github.io/course/?https://raw.githubusercontent.com/liaScript/CsharpCourse/master/18_Delegaten.md#1)

---------------------------------------------------------------------

**Wie weit sind wir schon gekommen?**

c# Schlüsselwörter:

| abstract    | as       | base     |`bool`      |`break`     |`byte`     |
|`case`       |`catch`   | char     |`checked`   |`class`     | const     |
|`continue`   |`decimal` | default  | delegate   |`do`        |`double`   |
|`else`       |`enum`    | event    | explicit   | extern     |`false`    |
|`finally`    | fixed    |`float`   |`for`       |`foreach`   |`goto`     |
|`if`         | implicit | in       |`int`       | interface  |`internal` |
| is          | lock     |`long`    |`namespace` |`new`       | null      |
| object      | operator |`out`     | override   |`params`    |`private`  |
| protected   |`public`  | readonly |`ref`       |`return`    |`sbyte`    |
| sealed      |`short`   | sizeof   | stackalloc |`static`    |`string`   |
|`struct`     |`switch`  |`this`    |`throw`     |`true`      |`try`      |
| typeof      |`uint`    |`ulong`   |`unchecked` | unsafe     |`ushort`   |
|`using`      | virtual  |`void`    | volatile   |`while`     |           |


Auf die Auführung der kontextabhängigen Schlüsselwörter wie `where` oder
`ascending` wurde hier verzichtet.

---

## Kontrollfragen

*1. Hier stehen jetzt Ihre Fragen ...*

---------------------------------------------------------------------
## 1. Motivation und Konzept

Ihre Aufgabe besteht darin folgendes Code-Fragment so umzuarbeiten, so dass
unterschiedliche Formen der Nutzer-Notifikation (neben Konsolenausgaben auch
Emails, Instant-Messager Nachrichten, Tonsignale) möglich sind. Welche Ideen
haben Sie dazu?

```csharp           Notification
using System;
using System.Reflection;
using System.Collections.Generic;

namespace Rextester
{
    public class VideoEncodingService{

      private string userId;
      private string filename;

      public VideoEncodingService(string filename, string userId){
         this.userId = userId;
         this.filename = filename;
      }

      public void StartVideoEncoding(){
         Console.WriteLine("The encoding job takes a while!");
         NotifyUser();
      }

      public void NotifyUser(){
          Console.WriteLine("Dear user {0}, your encoding job {1} was finished",
                            userId, filename);
      }
    }

    public class Program{
      public static void Main(string[] args){
         VideoEncodingService myMovie = new VideoEncodingService("007.mpeg", "12321");
         myMovie.StartVideoEncoding();
      }
    }
}
```
@Rextester.eval(@CSharp)

Gegen das Hinzufügen weiterer Ausgabemethoden in die Klasse `VideoEncodingService` spricht
die Tatsache, dass dies nicht deren zentrale Aufgabe ist. Eigentlich sollte
sich die Klasse gar nicht darum kümmern müssen, welche Art der Notifikation
genutzt werden soll, dies sollte dem Nutzer überlassen bleiben.

Folglich wäre es sinnvoll, wenn wir `StartVideoEncoding` eine Funktion als
Parameter übergeben könnten, die wir unabhängig von der eigentlichen Klasse
definiert haben.

### Grundidee

> Merke: Ein Delegat ist eine Methodentyp und dient zur Deklaration von Variablen,
> die auf eine Methode verweisen.

Für die Anwendung sind drei Vorgänge nötig:

1. Anlegen des Delegaten (Spezifikation einer Signatur)
2. Instanzierung (Zuweisung einer signaturkorrekten Methode)
3. Aufruf der Instanz

```csharp
// Schritt 1
//[Zugriffsattribut] delegate Rückgabewert DelegatenName(Parameterliste);
public delegate int Rechenoperation(int x, int y);

static int Addition(int x, int y){
  return x + y;
}

static int Modulo(int dividend, int divisor){
  return divident % divisor;
}

// Schritt 2 - Instanzieren
Rechenoperation myCalc = new Rechenoperation(Addition);
// oder
Rechenoperation myCalc = Addition;

// Schritt 3 - Ausführen
myCalc(7, 9);
```

Lassen Sie uns dieses Konzept auf unsere `VideoEncodingService`-Klasse anwenden.

```csharp           Notification
using System;
using System.Reflection;
using System.Collections.Generic;

namespace Rextester
{
    // Schritt 1
    public delegate void NotifyUser(string userId, string filename);

    public class VideoEncodingService{

      private string userId;
      private string filename;

      public VideoEncodingService(string filename, string userId){
         this.userId = userId;
         this.filename = filename;
      }

      public void StartVideoEncoding(NotifyUser notifier){
         Console.WriteLine("The encoding job takes a while!");
          // Schritt 3
          notifier(userId, filename);
      }
    }

    public class Program{

      // Die Notifikationsmethode ist nun Bestandteil der "Nutzerklasse"
      public static void NotifyUserByText(string userId, string filename){
        Console.WriteLine("Dear user {0}, your encoding job {1} was finished",
                          userId, filename);
      }

      public static void Main(string[] args){
         VideoEncodingService myMovie = new VideoEncodingService("007.mpeg", "12321");
         // Schritt 2
         NotifyUser notifyMe = new NotifyUser(NotifyUserByText);
         myMovie.StartVideoEncoding(notifyMe);
      }
    }
}
```
@Rextester.eval(@CSharp)

### Was passiert hinter den Kulissen?

Was wird anhand des Aufrufes

```
NotifyUser notifyMe = new NotifyUser(NotifyUserByText);
```

deutlich? Handelt es sich bei `notifyMe` wirklich nur um eine Methode?


                                           {{1}}
*******************************************************************************
Delegattypen werden von der `Delegate`-Klasse im .NET Framework abgeleitet.

https://docs.microsoft.com/de-de/dotnet/api/system.delegate?view=netframework-4.8

Die Delegattypen sind versiegelt, es ist nicht möglich benutzerdefinierte Klassen von `Delegate` Klasse abzuleiten. Dies ermöglicht es einer Methode, einen Delegaten als Parameter zu akzeptieren und den Delegaten zu einem späteren Zeitpunkt aufzurufen. Dies wird als asynchroner Rückruf bezeichnet und ist eine häufig verwendete Methode, um einen Aufrufer darüber zu benachrichtigen, dass ein langer Prozess abgeschlossen wurde. Wenn ein Delegat auf diese Weise verwendet wird, benötigt der Code, der den Delegaten verwendet, keine Kenntnisse über die Implementierung der verwendeten Methode. Die Funktion ähnelt den bereitgestellten Kapselungsschnittstellen.

Entsprechend der Codezeile `delegate int Transformer(int x);` generiert der
Compiler eine spezielle `sealed class Transformers`

```csharp
sealed class Transform : System.MulticastDelegate
{
  public int Invoke(int x);
  public IAsyncResult BeginInvoke(int x,
    AsyncCallback cb, object state);
  public int EndInvoke(IAsyncResult resut);
}
```
*******************************************************************************

                               {{2}}
*******************************************************************************
Seit C# 2.0 ist die Syntax für die Zuweisung einer Methode an eine Delegate-Variable
vereinfacht. Statt

```
NotifyUser notifyMe = new NotifyUser(this.NotifyUserByText);
```

kann nunmehr auch

```
NotifyUser notifyMe = this.NotifyUserByText;
NotifyUser notifyMe = NotifyUserByText;
```

verwendet werden.

*******************************************************************************

### Multicast Delegaten

Sollten wir uns mit dem Aufruf einer Methode zufrieden geben?

```csharp           MultiCast
using System;
using System.Reflection;
using System.Collections.Generic;

namespace Rextester
{
    public class Program{

      delegate int Transformer(int x);

      static int Square(int x){
          Console.WriteLine("This is method Square(int x)");
          return x*x;
      }

      static int Square(int x, int y){
          Console.WriteLine("This is method Square(int x, int y)");
          return x * x + y * y;
      }

      static int Cube(int x){
          Console.WriteLine("This is method Cube(int x)");
          return x * x * x;
      }

      public static void Main(string[] args){
        // alte Variante
        Transformer transform1 = new Transformer(Square);
        // neue Variante:
        Transformer transform2 = Square;

        Transformer transformer = Square;
        transformer += Cube;
        transformer += Cube;
        transformer -= Cube;
        transformer -= Square;

        Console.WriteLine("Zahl von eingebundenen Delegates {0}",
                          transformer.GetInvocationList().GetLength(0));

        transformer(5);
      }
    }
}
```
@Rextester.eval(@CSharp)

> Merke: Der Rückgabewert des Aufrufes entspricht dem der letzten Methode.


### Schnittstellen vs. Delegaten

```csharp    DelegatesVsInterfaces
void delegate XYZ(int p);

interface IXyz {
    void doit(int p);
}

class One {
    // All four methods below can be used to implement the XYZ delegate
    void XYZ1(int p) {...}
    void XYZ2(int p) {...}
    void XYZ3(int p) {...}
    void XYZ4(int p) {...}
}

class Two : IXyz {
    public void doit(int p) {
        // Only this method could be used to call an implementation through an interface
    }
}
```

Sowohl Delegaten als auch Schnittstellen ermöglichen einem Klassendesigner,
Typdeklarationen und Implementierungen zu trennen. Eine bestimmte Schnittstelle
kann von jeder Klasse oder Struktur geerbt und implementiert werden. Ein Delegat
kann für eine Methode in einer beliebigen Klasse erstellt werden, sofern die
Methode zur Methodensignatur des Delegaten passt.  

> Merke: In beiden Fällen kann die Schnittstellenreferenz oder ein Delegat kann
> von einem  Objekt verwendet werden, das keine Kenntnis von der Klasse hat, die
> die  Schnittstellen- oder Delegatmethode implementiert.

Wann ist welche der beiden Varianten vorzuziehen? Verwenden Sie einen Delegaten
unter folgenden Umständen:

+ Ein Ereignis-Entwurfsmuster wird verwendet.
+ Es ist wünschenswert, eine statische Methode einzukapseln.
+ Der Aufrufer muss nicht auf andere Eigenschaften, Methoden oder Schnittstellen des Objekts zugreifen, das die Methode implementiert.
+ Eine einfache Zusammensetzung ist erwünscht.
+ Eine Klasse benötigt möglicherweise mehr als eine Implementierung der Methode (siehe oben).

Verwenden Sie eine Schnittstelle wenn:

+ Es gibt eine Gruppe verwandter Methoden, die aufgerufen werden können.
+ Eine Klasse benötigt nur eine Implementierung der Methodesignatur.
+ Die Klasse, die die Schnittstelle verwendet, möchte diese Schnittstelle in andere Schnittstellen- oder Klassentypen umwandeln.
+ Die implementierte Methode ist mit dem Typ oder der Identität der Klasse verknüpft, z. B. mit Vergleichsmethoden.

Ein Gegenbeispiel einer Einzelmethodenschnittstelle anstelle eines Delegaten ist
`IComparable` oder die generische Version `IComparable<T>`. `IComparable`
deklariert die `CompareTo`-Methode, die eine Ganzzahl zurückgibt, die eine
Beziehung angibt, die kleiner, gleich oder größer als zwei Objekte desselben
Typs ist. Damit kann `IComparable` Grundlage für einen Sortieralgorithmus
verwendet werden. Obwohl die Verwendung einer Delegatenvergleichsmethode als
Grundlage eines Sortieralgorithmus gültig wäre, ist dies nicht ideal. Da die
Fähigkeit zum Vergleichen zur Klasse gehört und sich der Vergleichsalgorithmus
zur Laufzeit nicht ändert, ist eine Einzelmethodenschnittstelle ideal.

(aus https://docs.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/ms173173(v=vs.100))

## 2. Erweiterung

Neben dem Basiskonzept der Delegaten können in C# spezifischere Realisierungen
umgesetzt werden, die die Anwendung flexibler bzw. effizienter machen.

### Anonyme / Lambda Funktionen

Entwicklungshistorie von C# in Bezug auf Delegaten

| Version | Delegatendefinition |
| ------- | ------------------- |
| <2.0    | benannte Methoden   |
| >= 2.0  | anonyme Methoden    |
| ab 3.0  | Lambdaausdrücke     |

Dabei lösen Lambdaausdrücke die anonymen Methoden als bevorzugten Weg zum Schreiben von Inlinecode ab. Allerdings bieten anonyme Methode eine Funktion, über die Lambdaausdrücke nicht verfügen. Anonyme Methoden ermöglichen das Auslassen der Parameterliste. Das bedeutet, dass eine anonyme Methode in Delegaten mit verschiedenen Signaturen konvertiert werden kann.

**Anonyme Methoden**

Das Erstellen anonymer Methoden verkürzt den Code, da nunmehr ein Codeblock als Delegatparameter übergeben wird.

```csharp
// Declare a delegate pointing at an anonymous function.
Del d = delegate(int k) { /* ... */ };
```
Das folgende Codebeispiel illustriert die Verwendung. Dabei wird auch deutlich, wie
eine Methodenreferenz durch einen anderen ersetzt werden kann.

```csharp           AnonymouseDelegate
using System;
using System.Reflection;
using System.Collections.Generic;


// Declare a delegate.
delegate void Printer(string s);

namespace Rextester
{
    public class Program{

      static void DoWork(string k)      {
          System.Console.WriteLine(k);
      }

      public static void Main(string[] args){

          // Anonyme Deklaration
          Printer p = delegate(string j)
          {
              Console.WriteLine(j);
          };
          p("The delegate using the anonymous method is called.");

          // Der existierende Delegat wird nun mit einer konkreten Methode
          // verknüpft
          p = DoWork;
          // alternativ könnte man auch einen neuen Delegaten anlegen
          //Printer p1 = new Printer(DoWork);

          p("The delegate using the named method is called.");

      }
    }
}
```
@Rextester.eval(@CSharp)


**Lambda Funktionen**

Ein Lambdaausdruck ist ein Codeblock, der wie ein Objekt behandelt wird. Er kann als Argument an eine Methode übergeben werden und er kann auch von Methodenaufrufen zurückgegeben werden.

```
(<Paramter>) => { expression or statement; }

(int a) => { return a * 2; }; // Anweisungsblock
(int a) => a * 2; // einzelner Ausdruck
(int a) => { }; // leerer Anweisungsblock
```

```csharp           LambdaDelegate
using System;
using System.Reflection;
using System.Collections.Generic;


namespace Rextester
{
    public class Program{

      public delegate int Del( int Value);

      public static void Main(string[] args){
          Del obj = (Value) => {
                int x=Value*2;
                return x;
          };
          Console.WriteLine(obj(5));
      }
    }
}
```
@Rextester.eval(@CSharp)

Anonyme Methoden ermöglichen das Auslassen der Parameterliste. Das bedeutet, dass eine anonyme Methode in Delegaten mit verschiedenen Signaturen konvertiert werden kann. Dies ist bei Lambdaausdrücken nicht möglich.

### Generische Delgaten

Delegaten können auch als Generics realisiert werden. Das folgende Beispiel
wendet ein Delegate "Transformer" auf ein Array von Werten an. Dabei stellt
C# sicher, dass der Typ der übergebenen Parameter in der gesamten Verarbeitungskette übernommen wird.


```csharp           GenericDelegates
using System;
using System.Reflection;
using System.Collections.Generic;

namespace Rextester
{
    // Schritt I - Generisches Delegat
    delegate T Transformer<T>(T x);

    class Utility{
        public static void Transform<T>(ref T[] values, Transformer<T> trans)
        {
            for (int i = 0; i < values.Length; ++i)
                values[i] = trans(values[i]);
        }
    }

    public class Program{

      // Schritt II - Spezifische Methode, die der Delegatensignatur entspricht
      static int Square(int x){
          Console.WriteLine("This is method Square(int x)");
          return x*x;
      }

      static float Square(float x){
          Console.WriteLine("This is method Square(float x)");
          return x*x;
      }

      static void printArray<T>(T[] values){
          foreach(T i in values)
              Console.Write(i + " ");   
          Console.WriteLine();
      }

      public static void Main(string[] args){
        int[] values = { 1, 2, 3 };
        printArray<int>(values);

        //
        Utility.Transform<int>(ref values, Square);
        // gleichermaßen aber auch mit impliziter Typauswahl möglich
        // Utility.Transform(values, Square);
        printArray(values);
      }
    }
}
```
@Rextester.eval(@CSharp)

Beachten Sie, dass Sie alle expliziten Benennungen des Datentypen in

```
printArray<Type>(values)
Utility.Transform<Type>(ref values, Square);
```

entfernen können, die Ausführungsumgebung ordnet den generischen Delegaten eine
passende Methode zu.

### Action / Func

Der generischen Idee entsprechend kann man auf die explizite Definition von
eigenen Delegates vollständig verzichten. C# implementiert dafür
zwei Typen vor:

```csharp
delegate TResult Func<out TResult>();
delegate TResult Func<in T1, out TResult>(T1 arg1);
delegate TResult Func<in T1, in T2, out TResult>(T1 arg1, T2 arg2);
delegate TResult Func<in T1, in T2, in T3, out TResult>(T1 arg1, T2 arg2, T3 arg3);

delegate void Action();
delegate void Action<in T1>(T1 arg1);
delegate void Action<in T1, in T2>(T1 arg1, T2);
delegate void Action<in T1, in T2, in T3>(T1 arg1, T2 arg2, T3 arg3);
```

Im folgenden Beispiel wir die Anwendung illustriert. Dabei werden 3 Delegates
genutzt um die Funktionen `PrintHello` und `Square()` zu referenzieren.

| Delegate-Variante | Bedeutung                                                                  |
| ----------------- | -------------------------------------------------------------------------- |
| myOutput          | C#1.0 Version mit konkreter Methode und individuellem Delegaten (Zeile 8) |
| myActionOutput    | Generischer Delegatentyp ohne Rückgabewert                                 |
| myFuncOutput                  |    Generischer Delegatentyp mit Rückgabewert                                                                        |


```csharp           ActionUndFunc
using System;
using System.Reflection;
using System.Collections.Generic;

namespace Rextester
{

    public delegate void Output(string text);

    public class Program{

      static void PrintHello(string text){
          Console.WriteLine(text);
      }

      static int Square(int x){
          Console.WriteLine("This is method Square(int x)");
          return x*x;
      }

      static float Square(float x){
          Console.WriteLine("This is method Square(float x)");
          return x*x;
      }

      public static void Main(string[] args){
         Output myOutput = PrintHello;
         myOutput("Das ist eine Textausgabe");

         Action<string> myActionOutput = PrintHello;
         myActionOutput("Das ist eine Action-Testausgabe!");

         Func<float, float> myFuncOutput = Square;
         Console.WriteLine(myFuncOutput(5));
      }
    }
}
```
@Rextester.eval(@CSharp)



## Anhang

**Referenzen**


**Autoren**

Sebastian Zug, André Dietrich
