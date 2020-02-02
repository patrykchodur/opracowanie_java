# Programowanie obiektowe II - egzamin #

## I ##

Dany jest program w języku Java zawarty w pliku o następującej treści

```Java
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

```Java
enum C { TAK, NIE }

class E {
	String run() { return "->" + C.TAK + C.NIE; }
}
```

Wpisze się `->TAKNIE`, ponieważ enum ma domyślnie metodę `toString()`,
która zwraca nazwę wartości (można ją nadpisać, np)

```Java
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

```Java
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

```Java
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

Wypisze się `ABC30.5` ponieważ są dostępne 3 przeładowania `f()`,
a 1 jest `int` i 1.0 to `double`, więc kompilator poprawie sobie znajdzie
odpowiednie funkcje.

### 4 ###

```Java
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

```Java
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
Edit: domyślnie jest `package-private`

### 6 ###

```Java
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

```Java
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

```Java
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

```Java
	int n = 4;
	String s1 = "X";
	String s2 = n + 1;
	System.out.println(s1 + " " + s2 + " " + (n+1));
```

Błąd kompilacji, poniewać nie można przypisać `int` do `String`
(trzecia linia)

### 2 ###

```Java
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

```Java
	String s = "abcdef";
	System.out.println(s.length() + s.charAt(3)
			+ s.charAt(6));
```

Skompiluje się, ale wyrzuci wyjątek. `s` ma 6 elementów,
a `charAt(6)` to siódmy element. Stanie się to dopiero w czasie
działania, bo tego typu sprawdzenia nie są wykonywane w czasie
kompilacji.

### 4 ###

```Java
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

```Java
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

```Java
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

```Java
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

```Java
	A a = new A();
	System.out.println(a.a.k);
```

Wyświetli się `2`. Pomimo tego, że `a.a` ma wartość `null`, `k` jest
zmienną statyczną, która nie jest w żaden sposób przechowywana w obiekcie
`a`, więc nawet nie jest on ruszany. Standardowo do zmiennych statycznych
odwołuje się przez nazwę klasy, a nie obiekt, ale Java pozwala tak też
to robić.

### 9 ###

```Java
	A a = new A(3);
	System.out.println(A.k + " " + A.n);
```

Błąd kompilacji - klasa `A` nie ma statycznej zmiennej `n`.

### 10 ###

```Java
	A a = new A(3);
	System.out.println(a.k + " " + a.n);
```

Wyświetli się `3 0`. `k` jest ustawiane w konstruktorze `A`
(możemy się odwoływać do zmiennych statycznych przez obiekt)
, a `n` dostaje domyślną wartość, czyli 0.

### 11 ###

```Java
	System.out.println(new A().k + A.k);
```

Wyświetli się `4`, bo najpierw zostanie stworzony obiekt klasy `A`
i w konstruktorze ustawione będzie `k = 2` (ten bloczek luźny się odpala
w trakcie działania konstruktora, po *stworzeniu* `this`/`super`
a przed innymi instrukcjami w konstruktorze.

### 12 ###

```Java
	if (A.k)
		System.out.println("TAK");
	else
		System.out.println("NIE");
```

Błąd kompilacji, w Javie nie ma niejawnej konwersji z `int` do `boolean`
(w ogóle mało jest niejawnych konwersji, trzeba by dać
`if (A.k != 0)` żeby się zachowywało tak jak domyślnie w ***C++***

### 13 ###

```Java
	System.out.println( new A(3).f(new A(4)));
```

Wyświetli się `0`, `f()` zwraca n, które nie jest nigdzie ustawiane,
więc ma domyślną wartość 0

### 14 ###

```Java
	A a = new A();
	A b = new A(0);

	if (a != b)
		throw new E1();
	if (!a.equals(b))
		throw new E2();
	System.out.println("KONIEC");
```

Błąd kompilacji, `main()` nie jest zadeklarowany jako rzucający wyjątek,
np w ten sposób:
```Java
	public static void main(String[] args) throws Exception {
```
ani nie ma bloku `catch` naokoło miejsc, które rzucają wyjątki.

### 15 ###

```Java
	A a = new A();
	A b = a;
	try {
		if (a == b)
			throw new E1();
		System.out.print("A");
	}
	catch (Exception e) {
		System.out.print("B");
	}
	finally {
		System.out.print("C");
	}
	System.out.println("D");
```

Output to `BCD`. `a` i `b` wskazują na tę samą instancję, więc rzucony
zostanie wyjątek `E1`, który dziedziczy po `Exception`, więc wejdziemy
do bloku `catch`. Blok `finally` wywoływany jest zawsze po `try` `catch`
niezależnie, czy wyjątek miał miejsce, czy nie, czy został złapany, czy
puszczony wyżej. Jako, że złapaliśmy wyjątek kontynuowane jest wykonywanie
programu, więc wyświetla się `D`.

### 16 ###

```Java
	A a = new A();
	E2 x = new E2();
	if (a.k == 1)
		throw x;
	if (a.k == 2)
		throw a;
	System.out.println("KONIEC");
```

Błąd kompilacji - `A` nie dziedziczy po klasie `Throwable`, więc język
nie pozwala, aby zostało wyrzucone jako wyjątek.

### 17 ###

```Java
	var t = new ArrayList<Object>();
	t.add (new A());
	t.add(new E1());
	System.out.println(
			t.iterator().next().getClass().getName());
```

Wyświetli się `A`. `iterator()` zwraca iterator wskazujący na pierwszy
element listy, a `next()` zwraca obecny element i inkrementuje iterator.
Coś w stylu `*iter++` z ***C++***

### 18 ###

```Java
	A a = new A(5);
	var x = a.f(new A(6) {
		public int f(A a) { return 7; }
	} );
	System.out.println(x);
```

Wyświetli się `0`, bo obiekt klasy `A` przesłany do fukncji `f()`
wciąż nie zmieniał w żaden sposób `n`, więc jest domyślne 0.

### 19 ###

```Java
	Map<int, String> m = new TreeMap<>();
	m.put(1, "Nowak");
	m.put(2, "Kowalski");
	System.out.println(m.get(1));
```

Błąd kompilacji - wszystkie kontenery w Javie wymagają obiektów. `int`,
`byte`, `char` nie są obiektami. Trzeba by było dać `Integer`

### 20 ###

```Java
	Map<String, String> m = new TreeMap<>();
	m.put("Nowak", 1 + "");
	m.put("Kowalski", "2");
	System.out.println(m.get(1));
```

Błąd w czasie wykonywania programu. Metoda `get()` przyjmuje referencję
do typu `Object`, a nie typ podany jako klucz, więc 1 jest konwertowane
do `Integer`, które jest przekazywane do funkcji `get()`, która dopiero
w środku sprawdza poprawność typów. Trochę więcej na ten temat jest
w następnym zadaniu.

### 21 ###

```Java
	class B<T extends E3> {
		T x = new T();
		void f() {
			throw x;
		}
	}

	new B<E3>().f();
```

Błąd kompilacji - wyrażenie `new T()` jest niedozwolone w Javie.
Szablony w Javie działają inaczej niż w ***C++***, nie jest tworzona
wersja funkcji/klasy z określonymi parametram, tylko tak naprawdę
używany jest tam typ `Object`, bądź jeśli podamy więcej informacji inny
(tutaj `E3`). Kompilator nie wie więc jakiego konstruktora ma użyć.

### 22 ###

```Java
	A a = new A();
	a = a.a;
	System.out.println(a);
```

Wynik: `null`. Najpierw tworzony jest obiekt `A` i referencja na niego
zapisana jest do `a`. Następnie do `a` przypisujemy element `a` klasy `A`
(`a.a`), które (zawsze) wynosi `null`. `a` wskazuje więc na `null`,
co zostaje wyświetlone w `println()` (normalnie byłby adres, ale jeśli
przekażemy `null` to wyświetli `String` `"null"`.

## III ##

Uzupełnić kod tak, aby plik z kodem się kompilował

### 1 ###

```Java

// ...........

class D {
	A a = new C();
	B b = new C();
}
```

Odpowiedź:

```Java
interface A { }
interface B { }
class C implements A, B { }



class D {
	A a = new C();
	B b = new C();
}
```

Można też dać klasy dziedziczące jedna po drugiej.

### 2 ###

```Java

// ..........


public class E {
	void f() {
		try {
			F x = new G();
			throw x;
		}
		catch (RuntimeException e) { }
		catch (/* ... */ e) { }
		catch (F e) { }
		catch (/* ... */ e) { }
	}
}
```

Odpowiedź:

```Java
class F extends Exception { }

class G extends F { }


public class E {
	void f() {
		try {
			F x = new G();
			throw x;
		}
		catch (RuntimeException e) { }
		catch (G e) { }
		catch (F e) { }
		catch (Exception e) { }
	}
}
```

Jeśli jeden wyjątek dziedziczy po drugim, to dziedziczący musi się znaleść
wcześniej na liście łapanych wyjątków, ponieważ np `G` może być złapane
przez `catch (F e)` lub `catch (Exception e)`, po którym dziedziczy przez
`F`

### 3 ###

```Java

// ........

		String a;
		String b;

		// .......

		System.out.println(a == b);
	}
}
```

Ma dawać `true`.

Odpowiedź:

```Java
public class E {
	
	public static void main(String[] args) {
		String a;
		String b;

		a = b = new String();

		System.out.println(a == b);
	}
}
```

Porówanie `a == b` sprawdza, czy `String` `a` wskazuje na ten sam obiekt
co `b`, więc trzeba do nich przypisać to samo.


### 4 ###

```Java

// ........

		String a;
		String b;

		// .......

		System.out.println(a == b);
	}
}
```

Ma dawać `false`.

Odpowiedź:

```Java
public class E {
	
	public static void main(String[] args) {
		String a;
		String b;

		a = new String();
		b = new String();

		System.out.println(a == b);
	}
}
```

`a` i `b` wskazują na inne obiekty. Co ciekawe, dla `a = ""; b = "";`
wychodzi `true`, co jest raczej kwestią optymalizacji (literały znajdują
się w innym miejscu pamięci niż zmienne, kompilator jest w stanie się
zorientować, że dany literał był używany wielokrotnie i przypisze stringom
ten sam adres. Jeśli chcemy stringa zmodyfikować i tak tworzony jest nowy
obiekt).

### 5 ###

```Java
public /* ... */
	E x;
	V y;
	public A(E a, V b) {

		// .....

	}
	public static void main(String[] args) {
		A o = new A("a", "b");
		System.out.println(o.x + "" + o.y);
	}
}
```

Ma dawać `ab`.

Odpowiedź:

```Java
public class A<E extends String, V extends String> {
	E x;
	V y;
	public A(E a, V b) {

		x = a;
		y = b;

	}
	public static void main(String[] args) {
		A o = new A("a", "b");
		System.out.println(o.x + "" + o.y);
	}
}
```

W konstruktorze podane są dwa literały, więc potrzeba czegoś, co da się
niejawnie przekonwertować do `String`a.
`E` i `V` przedłużają klasę `String`, żeby były traktowane przez
kompilator jak `String`, lub coś więcej niż `String`.

### 6 ###

```Java
public class B {

	/* ... */ = new /* ... */ (0);

	public static void main(String[] args) {
		n = /* ... */
		System.out.println("n-" + n.toString());
	}
}
```

Ma dawać `n=1`.

Odpowiedź:

```Java
public class B {

	static Integer n = new Integer(0);

	public static void main(String[] args) {
		n = 1;
		System.out.println("n-" + n.toString());
	}
}
```

Trzeba pamiętać o `static`, bo n jest zmieniane w funkcji `main()`.

### 7 ###

```Java

// .....

class /* ... */
	public E (int a, int b) {
		super(a, b);
	}
}
```

Odpowiedź:

```Java
class B {
	public B(int a, int b) { }
}

class E extends B {
	public E (int a, int b) {
		super(a, b);
	}
}
```

`super()` wywołuje konstruktor klasy nadrzędnej, w tym przypadku `B`.

### 8 ###

```Java
```
