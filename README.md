# 1. Wstęp

# 2. Zmiany w AngularJS 1.5+
### 2.1 Components
### 2.2 One-way databinding
### 2.3 Lifecycle hooks
### 2.4 Multi-slot transcusion

# 3. Webpack jako module bundler
### 3.1 Czym jest webpack?
### 3.2 Gulp vs Webpack
### 3.3 Opis funkcjonalności
### 3.4 Przykładowa konfiguracja

# 4. ES6
### 4.1 Const i let
### 4.2 Method shorthand
### 4.3 Property value shorthand
### 4.4 Arrow function
### 4.5 Spread operator
### 4.6 Default parameters
### 4.7 String interpolation
### 4.8 Destructuring
### 4.9 Import/Export/export default
### 4.10 Avoid wildcard imports
### 4.11 Class
### 4.12 Class inheritance

# 5. TypeScript
### 5.1 Silne typowanie
Używając TypeScript możemy uniknąć wielu testów sprawdzających i walidujących typy zmiennych oraz samych błędów. Jeżeli zadeklarujemy że zmienna jest typu X to kompilator TypeScript nie pozowoli nam zbudować projektu dopóki wszystkie referencje przypisania do tej zmiennej nie będą typu X.
```javascript
let counter: number;
let greeting: string;
let isValid: boolean;
```
Podczas deklaracji zmiennej możemy również przypisać jej wartość:
```javascript
let name: string = 'Greg';
let arr: number[] = [1,2,3];
let arr2: Array<string> = ['Hi', 'Hey', 'Hello'];
```
Podczas integracji z zewnętrznymi bibliotekami, często nie posiadamy silnego typowania co powoduje że jesteśmy zmuszeni do pisania własnych interfejsów (o tym za chwilę). Z pomocą przychodzi magiczny typ any, który pozwala przypisać dowolny typ do zadeklarowanej tak zmiennej. Pamiętajmy jednak aby go unikać i używać tylko w specjalnych przypadkach (nie po to podpinamy TypeScript aby używać any):
```javascript
let myArr: any[] = [81, false, 'go away'];
```
Silne typowanie to nie tylko deklaracja zmiennych. Weźmy za przykład funkcje:
```javascript
function addNumbers(num1: number, num2: number): number {
  // nie musimy sprawdzać czy num1 i num2 to liczby bo zapewnia to TypeScript!
  return num1 + num2;
}
```
Zwróćmy również uwagę, że podajemy jawnie typ zwracanej wartości (po nawiasach funkcji) - number. Pozwali to WebStormowi zapewnić świeny IntelliSense.

Jeśli funkcja nie zwraca żadnej wartości możemy skorzystać z kolejnego typu - void:
```javascript
function echo(msg: string): void {
  console.log(msg);
}
```

TypeScript pozwala nam korzystać również z enumów 'out of the box':
```javascript
enum Event {DONE, GONE, FAILED};
let status: Event = Event.FAILED;
```
Tak jak w językach typu C#, enumy są numerowane od 0, możemy to jawnie zmienić:
```javascript
enum Color {RED = 16, BLUE, GREEN}
```
Wyłuskanie wartości tekstowej z enuma:
```javascript
let myColor: string = Color[16]; // 'RED'
```

Jeśli wiemy jakiego typu jest nasza zmienna, możemy ją rzutować tak jak w klasycznych językach programowania:
```javascript
let message: any = 12;
let message2: any = 'Hello world!';

let strLength = (<string>message).length;
```

Posumowując za pomocą TypeScripta dostajemy mozliwość korzystania z silnego typowania w naszym kodzie JavaScript. Wyróżniamy nastepujące typy:
* Boolean
* Number
* String
* Array lub type[]
* Tuple
* Enum
* Any
* Void

### 5.2 Interfaces
Interfejsy to sposób na deklaracje złożonych typów - innymi słowy interfejs to przepis na naszą strukturę danych (obiekt). Weźmy za przykład interfejs dla modelu danych:
```javascript
interface IPerson {
  name: string;
  lastName: string;
  age?: number; //parametr opcjonalny
  getFullName: Function;
}

function playWithPerson(person: IPerson): void {
    //TypeScript zapewnia że person jest typu IPerson i posiada name, lastName oraz funkcję getFullName.
    //Pole age jest opcjonalne
    console.log(person.getFullName());
    
    if(person.age && person.age > 30) {
      console.log('You are old dude');
    }
}
```
Inny przykład to opisanie naszego Angularowego serwisu przez interface:
```javascript
interface IPersonServiceActions {
  getPerson(id: number): IPerson;
  deletePerson(id: number): IPerson;
  getPeople(): Array<IPerson>;
}

class PersonService implements IPersonServiceActions {
  public getPerson(id: number): IPerson {
     //$http.get ...
  }
  
  public deletePerson(id: number): IPerson {
    //$http.delete ...
  }
  
  public getPeople(): Array<IPerson> {
    //$http.get
  }
}
```
Jedyną trudnością jest zapisanie implements i podanie interfejsu :)

Ostatnim zagadnieniem jest rozszerzanie interfejsu o inne interfejsy:
```javascript
interface IPet {
  name: string;
  age: number;
  eat: Function;
}

interface ICat extends IPet {
  // ICat rozszerza interfejst IPet
  moew: Function;
}

interface IBird extends IPet {
  fly: Function;
}

function playWithPets(kitty: ICat, birddy: IBird) {
	console.log(kitty.name); //zapewnia nam to IPet
	kitty.moew(); //ICat
	
	birddy.eat(); //IPet
	birddy.fly(); //IBird
}
```

### 5.3 Classes
Klasy zostały stworzone aby ukryć nielubiane prototypowe dziedziczenie. Zostały wprowadzone do języka wraz z ES6, a dzięki TypeScript możemy jeszcze więcej!

Deklaracja klasy:
```javascript
class Company {

}
```
Każda klasa posiada specjalną metodę - constructor, zostanie ona wywołana tylko raz podczas tworzenia nowego obiektu:
```javascript
class Company {
  public name: string;
  
  constructor(name: string) {
    this.name = name;
  }
}

var pgs = new Company('PGS');
pgs.name; //PGS
```
Powyższy kod możemy uprościć, dzięki specjalnym skrótom TS'a:
```javascript
class Company {
  constructor(public name: string) {
  }
}
```
Otrzymujemy też możliwość ukrywania zmiennych:
```javascript
class Car {
  constructor(public engine: string, private owner: IPerson) {
  }
}

//gdzieś w kodzie, zakładając że car to instancja klasy Car
car.owner; // błąd - zmienna jest prywatna

```
Gdy nasza lista zmiennych inicjalizowana w konstruktorze staje się coraz dłuższa możemy skorzystać z interfejsów:
```javascript
class Example {
  constructor(public name: string, public lastName: string, public age: number, public height: number, public width: number){
  }
}

//lub
interface IThing {
  name: string;
  lastName: string;
  age: number;
  height: number;
  width: number;
}

class Example2 {
  constructor(public thing: IThing){
  }
}
```
TypeScript wymusza na nas abyśmy nie używali słowa kluczowego function w obrębie klasy:
```javascript
class FunctionLess {
  doSomething(name: number): void {
  }
  
  doSomethingElse(tab: Array<string>): Array<string> {
    return tab.map(x => x.name = x.name.toUpperCase());
  }
}
```
TypeScript udostępnia nam również klasy abstrakcyjne, które używamy jako klasy bazowe np. wyobraźmy sobie że tworzymy klasę odpowiedzialna za paginację. Taka klasa może być dziedziczona w dowolnym komponencie, dzięki czemu komponent ten zyskuje całe API potrzebne do jej zaimplementowania. 
```javascript
abstract class Pagination {
  public currentPage: number;
  public totalElements: number;
  public limit: number;
  
  next(): void {
    this.currentPage++;
  }
  
  setPage(pageNo: number): void {
    this.currentPage = pageNo;
  }
}

class MyAppComponent extends Pagination {
  constructor() {
     // jeśli dana klasa dziedziczy po innej, musimy wywołać w konstruktorze
     // konstruktor klasy bazowej, dzieje się to za pomocą specjalnej metody super()
     super();
     this.setPage(10);
  }
}
```
Część z was zapyta czym w takim razie różni się interface od klasy abstract? W najprostszych słowach za pomocą abstract możemy przedstawić przykładową implementację danych metod. W przypadku interfejsów nie jest to możliwe, ponieważ interface to przepis na klasę. Po drugie abstract class stworzy nam dziedziczenie prototypowe, a interface zostanie usunięty podczas kompilacji TypeScripta (nie ma czegoś takiego jak Interface w JavaScripcie).

Możemy również dziedziczyć po 'zwykłych' klasach również to sprowadza się do prostego dziedziczenia prototypowego:
```javascript
class A {
  constructor(public name: string){
  }
}

class B extends A {
  constructor() {
    super('AName');
  }
}
```

### 5.4 Generics
Generics to kolejna rzecz zapożyczona z języków obiektowych. Służy do stworzenia jak najbardziej re-używalnego kodu. Przykład:
```javascript
function echo(value: any): any {
  return value;
}
```
W tym wypadku tworzymy funkcję, która przyjmuje dowolny argument i go zwraca. Jest to trochę oszukany Generic, ale pozwala zrozumieć całą ideę. Do funkcji echo możemy przekazać dowolną wartość, bez względu na jej typ zostanie ona zwrócona. A teraz zobaczmy jak możemy implementować Generics w TypeScript:
```javascript
function echo<T>(value: T): T {
  return value;
}
```
Zasada działania powyższych 2 funkcji jest identyczna. T - jest to placeholder, może być to dowolnie inna litera, w innych językach programowania przyjęło się aby używać litery T.
```javascript
let num: number = <number>echo(12);      //12
let str: string = <string>echo('Hello'); //'Hello'
```
Nasza metoda stała się generyczna - oznacza to że przyjmuje dowolne typy zmiennych.

### 5.5 Decorators
### 5.6 Typings
Typings to idealnym przykład wkładu społeczności w rozwój języka. Wyobraźmy sobie że podpinamy do naszego projektu ui-router, a następnie konfigurujemy go w naszej aplikacji dodając różne stany itp. Skąd mamy wiedzieć jakie metody posiada service $state albo provider $stateProvider? Nie wiemy, musimy przejść do dokumentacji, tam zobaczyć jakie funkcje są nam udostępniane, a następnie wrócić do kodu i jeszcze sprawdzając 5x dokumentację przepisac daną metodę/ własność. Inną opcją jest tworzenie własnych interface'ów tak aby ułatwić nam życie np. w jednym miejscu stworzyć katalog typings a tam zadeklarować interface IState lub IStateProvider przepisując definicje funkcji 1:1 zgodnie z dokumentacją. A co jeśli powiem wam, że ktoś zrobił to już za was? :)

Lista dostępnych typingsów: https://github.com/DefinitelyTyped/DefinitelyTyped

Sposób instalacji
```
npm install typings -g
typings install angular-ui-router --save --ambient
```

Następnie w naszym kodzie:
```javascript
angular.module('app.routing', [])
	.config(($stateProvider: ng.ui.IStateProvider) => {
	   //posiadamy Intellisense dla $stateProvidera
	});
```
Na tym właśnie polega cała idea TypeScripta - typowanie, klasy, dziedziczenie i darmowe typingsy. Zapraszam do analizy staretera i przykładowej aplikacji.

# 6. Przykłady
- **Unikaj jQuery**
- **Wszystko jest komponentem**
- **Przestań używać controller, controllerAs, ng-controller**
- **Zrezygnuj z factory i provider na rzecz service**
- **ui-router powinien operować na komponentach**
- **Spójrz inaczej na dyrektywy**
- **Zrezygnuj z $rootScope**
- **Sposoby komunikacji rodzic-dziecko, dziecko-rodzic**
- **Zapomij o $scope, chyba że potrzbujesz $watch**
- **Używaj ES6/TS class tam gdzie jest to możliwe**
- **Unikaj 2-way-databinding**
- **Korzystaj z lifecycle hooks**
- **Używaj 'ngInject'**
- **Wszystko jest modułem**
- **Buduj apliakcje jako drzewo komponentów**
- **index.js**
- **Konwencje nazewnicta plików**
- **Struktura plików**
- **Rozważ użycie folderu shared**
- **Unikaj resolve**
- **Zrezygnuj a anonimowych funkcji**
- **Unikaj vm**
- **Używaj vendor.js dla bibliotek i modułów zewnętrznych**
- **index.html powinien być tak mały jak to miżliwe**
- **Pojedyńczy plik index.js powinien rejestrować całą aplikację**
- **Tworzenie dyrektyw**
- **Rozważ użycie dekoratora @Component**
- **Unikaj restrict:'E' dla dyrektyw***
- **Pamiętaj o wyrejestrowaniu funkcji $on i $timeout**
- **Zarządzaj interfejsami**
- **Statefull/Stateless components**
- **Performance tricks**
- **Unikaj zahardcodowanych nazw modułów**
- 
# 7. Starter
### 7.1 Struktura plików
### 7.2 Opis
### 7.3 Lintery

# 8. Przykładowa aplikacja

# 9. Co dalej?
### 9.1 AngularJS 2
### 9.2 Yeoman generator

# 10. Reference i uwagi
### 10.1 Przydatne linki
### 10.2 Kontakt
