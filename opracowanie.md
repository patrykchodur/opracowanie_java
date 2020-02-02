# Programowanie obiektowe II - egzamin #

## I ##

Dany jest program w języku Java zawarty w pliku o następującej treści

```
//******//

public class Main {
	public static void main(String[] args) {
		E e = new E();
		System.out.println(e.run());
	}
}
```

Czy po wstawieniu poniższego fragmentu kodu w miejsce oznaczone //******//
program się skompiluje? Jeśli tak, to **co zostanie wypisane** po jego
wykonaniu (jakiś napis, informacja o wyjątku, nic, ...?). Jeśli *nie*,
to **dlaczego**?

### 1 ###

```
enum C { TAK, NIE }

class E {
	String run() { return "->" + C.TAK + C.NIE; }
}
```

Wpisze się `->TAKNIE`, ponieważ enum ma domyślnie metodę `toString()`,
która zwraca nazwę wartości (można ją nadpisać, np)

```
enum C { 
	TAK,
	NIE;

	@Override
	public String toString() {
		return super.toString().toLowerCase();
	}
}
```

### 2 ###

```
class D {
	String s = "ABC";
	public D(String s) {
		this.s = s;
	}
}

class E extends D {
	String run() {
		return new D("DEF").s;
	}
}
```

Nie skompiluje się, ponieważ E posiada domyślny konstruktor
(bo nie został żaden napisany), który będzie próbował wywołać
bezargumentowy konstruktor D, który nie istnieje.

### 3 ###

```
class D {
	String f() {
		return "ABC";
	}
	double f (double x) {
		return x/2;
	}
}

class E extends D {
	D d = new D();
	int f(int x) {
		return x + 2;
	}
	String run() {
		return d.f() + f(1) + f(1.0);
	}
}
```

Wypiszę się `ABC3.05` ponieważ są dostępne 3 przeładowania `f()`,
a 1 jest `int` i 1.0 to `double`, więc kompilator poprawie sobie znajdzie
odpowiednie funkcje.

### 4 ###

```
interface A {
	String f();
}
interface B {
	B g();
}

class E implements A, B {
	public String f() {
		return "A";
	}
	public B g() {
		return this;
	}
	public String run() {
		return g().f();
	}
}
```

Błąd kompilacji, ponieważ w `run()` próbujemy wołać funkcję `f()`
przez typ `B`, który nie posiada takiej metody.

### 5 ###

```
interface A {
	default String f() {
		return "Abc";
	}
}

class E implements A {
	String f() {
		return "Def";
	}
	String run() {
		return f();
	}
}
```

Błąd kompilacji, ponieważ wszystkie metody w interfejsie są domyślnie
`public`, a w `E` metoda `f()` jest prywatna (bo w klasie domyślnie
jest `private`)

### 6 ###

```
interface A {
	default String f() {
		return "Abc";
	}
}

interface B extends A {
	default String f() {
		return "Def";
	}
}

class E implements B {
	String run() {
		return f();
	}
}
```

Wypisze się `Def`, nie ma żadnego powodu, aby `B` nie mogło nadpisać
metody z `A`

### 7 ###

```
interface A {
	int f(int i);
}

class E {
	static final int i = 2;
	int x(int i, A a) {
		return a.f(i);
	}
	String run() {
		return "x=" + x(i, i->i*i);
	}
}
```

Wynik: `x=4`. `i->i*i` to lambda, która dla `i` zwraca `i*i`.
Wyrażenie labmda w Javie może implementować interfejs pod warunkiem,
że interfejs ma jedną metodę bez definicji. Kompilator jest sobie
w stanie powiącać, że `i -> i*i` musi dziedziczyć po `A`, które spełnia
te wymagania.
[Źródło](http://tutorials.jenkov.com/java-functional-programming/functional-interfaces.html#functional-interfaces-can-be-implemented-by-a-lambda-expression)

## II ##

Dany jest następujący program

```
import java.util.*;
class A {
	public static int k = 1;
	public int n;
	public final A a = null;

	{ k = 2; }

	public A() { }
	public A(int k) { this.k = k; }
	public int f(A a) { return a.n; }
}

class E1 extends Exception { }
class E2 extends RuntimeException { }
class E3 extends ArithmeticException { }

public class Main {
	public static void main(String[] args) {
		/******/
	}
}
```
	
Czy po wstawieniu poniższego fragmentu kodu w miejsce oznaczone //******//
program się skompiluje? Jeśli tak, to **co zostanie wypisane** po jego
wykonaniu (jakiś napis, informacja o wyjątku, nic, ...?). Jeśli *nie*,
to **dlaczego**?

### 1 ###

```
	int n = 4;
	String s1 = "X";
	String s2 = n + 1;
	System.out.println(s1 + " " + s2 + " " + (n+1));
```

Błąd kompilacji, poniewać nie można przypisać `int` do `String`
(trzecia linia)

### 2 ###

```
	String s1 = "ABC";
	StringBuilder s2 = new StringBuilder("ABC");
	if (s1 == s2)
		System.out.print("1");
	if (s1.equals(s2))
		System.out.print("2");
```

Błąd kompilacji, w Javie nie ma przeładowywania operatorów,
więc `s1 == s2` porównuje, czy są to te same instancje (a są to
dwa kompletnie różne typy, więc nawet nie ma jak sprawdzić).
`s1.equals(s2)` też zwaca `false` jbc.

### 3 ###

```
	String s = "abcdef";
	System.out.println(s.length() + s.charAt(3)
			+ s.charAt(6));
```

Skompiluje się, ale wyrzuci wyjątek. `s` ma 6 elementów,
a `charAt(6)` to siódmy element. Stanie się to dopiero w czasie
działania, bo tego typu sprawdzenia nie są wykonywane w czasie
kompilacji.

### 4 ###

```
	String s = "";
	s += 1;
	s += 'X';
	s += true;
	if (s == "1Xtrue")
		System.out.print("A");
	if (s == "1X1")
		System.out.print("B");
	if (s.equals("1Xtrue"))
		System.out.print("C");
```

Wyświetli się `C`. `s += 1` to `s = s + 1`, więc nie ma problemu
z tym, że po prawej jest `int`. `==` sprawdza, czy to ta sama instancja,
a tak nie jest, bo `"1Xtrue"` nie ma nawet szansy być przypisany do
czegokolwiek.

### 5 ###

```
	int[][] ti = new int[5][];
	Object[][][] to = new Object[3][0][5];
	to[0][0][0] = ti[0][0];
```

Skompiluje się, ale wyrzuci wyjątek, bo:

1. `to` ma w drugim wymiarze 0 (przy tworzeniu), więc indeks 0
jest poza zakresem,
2. drugi wymiar `ti` nie jest podany przy tworzeniu,
więc `ti[0]` to `null` (a my robimy tu `(ti[0])[0]`

### 6 ###

```
	ArrayList<Integer> v = new ArrayList<>();
	v.add(4);
	v.add(6);
	v.set(1, 6);
	v.remove(0);
	for (var i : v)
		System.out.print(i);
```

Wyświetli się `6`. Najpierw dodajemy 4 i 6, potem ustawiamy na drugim
miejscu 6 (co nic nie zmienia), a potem usuwamy pierwszy element
(przekazaliśmy indeks typu `int`, więc zostaje nam 6

### 7 ###

```
	ArrayList<Integer> v = new ArrayList<>();
	v.add(4);
	v.add(5);
	v.set(1, 6);
	v.remove(Integer.valueOf(0));
	for (int i : v)
		System.out.print(i);
```

Wyświetli się `46`. Metoda `remove()` przyjmuje albo indeks, albo
obiekt (referencję na niego). Tutaj przekazujemy `Integer`, który jest
obiektem, więc `remove()` szuka, czy ten konkretny obiekt (stworzony
wewnątrz nawiasu `remove()`) znajduje się w liście, co jest niemożliwe,
bo nie miał nawet szansy być nigdzie zachowany (nie mówiąc o tym,
że w liście nie ma obieku `Integer` 0). 4, 5 i 6 są konwertowane z
`int` do `Integer` przy dodawaniu.

### 8 ###

```
	A a = new A();
	System.out.println(a.a.k);
```

Wyświetli się `2`. Pomimo tego, że `a.a` ma wartość `null`, `k` jest
zmienną statyczną, która nie jest w żaden sposób przechowywana w obiekcie
`a`, więc nawet nie jest on ruszany. Standardowo do zmiennych statycznych
odwołuje się przez nazwę klasy, a nie obiekt, ale Java pozwala tak też
to robić.

### 9 ###

```
	A a = new A(3);
	System.out.println(A.k + " " + A.n);
```

Błąd kompilacji - klasa `A` nie ma statycznej zmiennej `n`.

### 10 ###

```
	A a = new A(3);
	System.out.println(a.k + " " + a.n);
```

Wyświetli się `3 0`. `k` jest ustawiane w konstruktorze `A`
(możemy się odwoływać do zmiennych statycznych przez obiekt)
, a `n` dostaje domyślną wartość, czyli 0.
