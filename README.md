# 1. Wstęp
Niniejszy styleguide zawiera bogaty wstęp teoretyczny potrzebny do zrozumienia działania startera i przykładowej aplikacji napisanych w modułowym podejściu za pomocą AngularJS 1.5.7 , webpack i TypeScript. 

Rozdział 2 opisuje breaking changes, które zaprezentował AngularJS od wersji 1.5. Nastepny rozdział to krótkie wprowadzenie do webpacka. Zaznaczam tutaj, że prawdopodobnie będziemy posiadać jedną konfigurację dzieloną pomiędzy projektami i zazwyczaj sprowadzi się do kopiowania, dlatego pamiętajcie, że nie musicie znać całego API webpacka aby czuć się w nim swobodnie. Konfiguracja jest na tyle elastyczna, że na pewno każdy da radę skonfigurować ją pod swój projekt, a na 99% przyjdziecie na gotowe ;). 

Dalej w 4 i 5 rozdziale znajduje się spis wszystkich potrzebnych wam informacji dotyczących ES6 i TypeScripta. Następnie rozdział 6 to przejście do samego styleguida czyli spisu best-practices używanych podczas pisania aplikacji. Na koniec prezentacja działającej aplikacji, którą polecam odpalić w WebStormie - otrzymujemy możliwość m.in skakania po referencjach i podpowiedzi Intellisense.

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
Pisząc w ES6 powinniśmy całkowicie zrezygnować z `var`. Co więcej powinniśmy w 90% przypadków używać `const` czyli oznaczać zmienne jako niemodyfikowalne. `const` zapewnia że referencja do danej zmiennej nie zostanie zmieniona, a każda próba takiej zmiany spowoduje błąd. Jeśli nasza zmienna jest mutowalna powinniśmy użyć `let`. 

```javascript
let counter = 0;
counter++;
```

```javascript
const url = `${CONFIG.API}/path/${id}`;

$http.get(url)
```

Różnica pomiędzy `const`, `let` oraz `var` jest taka, że `const` i `let` są `block-scoped`, a `var` jest `function-scoped` oznacza to, że są widoczne tylko w obrębie najbliższego bloku and nie funkcji. Co więcej w przypadku `let` i `const` nie działą hoisting czyli nie możemy użyć zmiennej przed jej deklaracją.

### 4.2 Method shorthand
Korzystając z ES6 możemy pominąć słowo kluczowege `function` w deklaracjach obiektów czy też samych klasach:
```javascript
class Person {
   addValues(num1, num2) {
     //pomijamy function
   },
   subtractValues: function(num1, num2) {
     //old-fashion way
   }
}

const config = {
   api: '/api/v2/',
   buildUrl(url) {
      return `${this.api}/${url}`;
   },
   dontDoThisPlease: function(){
     console.log('Use ES6 dude');
   }
}
```

### 4.3 Property value shorthand
Jest to jeden z moich ulubionych 'ficzerów' ES6, pozwala nam pominąć powtarzający się kod. Skoczmy odrazu do praktycznego przykładu:
```javascript
  .sevice('AuthService', function($http) {
      const service = {
         login: login,
         logout: logout,
         getMe: getMe
      };
      
      function login() {}
      function logout() {}
      function getMe() {}
      
      return service;
  });
```
Z ES6 możemy usunąć zbędny kod w obiekcie service:
```javascript
const service = {
  login,
  logout,
  getMe
};
```

### 4.4 Arrow function

Arrow function jest to skrócona wersja zapisu anonimowej funkcji, często upraszcza nasz kod w takim stopniu, że możemy zapisać jej logikę w jednej linjce kodu. Przy korzystaniu z arrow function należy pamiętać że `this` będzie wskazywał na kontekst w którym funkcja została wywołana. `call` , `apply` i `bind` nie mogą zmienić `this` w arrow function, ponieważ arrow function nie tworzy własnego this. Jeśli potrzebujemy zmienić `this` musimy skorzystać ze zwykłej funkcji.

Przykłady:
```javascript
$http.get('url/to/api')
   .then((response) => response.data); //zwraca response.data
```

```javascript
const arr = [1,2,3,4,5];
arr.map(number => number *2);
```

```javascript
  Builder.build(() => {
     console.log('Gdy nie mamy parametrów, musimy użyć {}');
  });
```

```javascript
  const sum = arr.reduce((sum, number) => sum + number, 0)
```

### 4.5 Rest/Spread operator
`Rest` i `spread` operators wyglądają identycznie, ale używane są w innych przypadkach. Operator ten zapisujemy jako `...`
`rest` operator używany jest w celu zastepienia `arguments` czyli tablicopodobnej struktury, która przechowuje wszystkie parametry z jakimi została wywołana funkcja.

```javascript
function joinArr(){
    if(!arguments) {
		return; 
    }
	
    return Array.prototype.slice.call(arguments).join(' ');
}

joinArr('a', 'b', 'c', 'd', 'e', 'f'); // a b c d e f
```
ES6 rest operator:
```javascript
function joinArr(...words) {
   return words.join(' ');
}

joinArr('a', 'b', 'c', 'd', 'e', 'f'); // a b c d e f
```

`Rest` operator może również przechwycić tylko niektóre argumenty funkcji:
```javascript
function restMe(param1, param2, ...params){
  //param1 = 'A'
  //param2 = 'B' 
  //params = ['C', 'D', 'E', 'F'];
}

restMe('A','B', 'C', 'D', 'E', 'F');
```

`Spread` operator używamy na wszystkim co jest `iterable`. Najłatwiej zrozumieć go jako 'rozpakowanie' tablicy na elementy - podobnie działają streamy w RxJS. 
```javascript
console.log([1,2,3]); //[1,2,3]
console.log(...[1,2,3]) //"rozpakowanie" => 1 2 3
```

`Spread` operator znajduje również zastosowaie w kopiowaniu tablic:
```javascript
const items = [1,2,3,3,4,5,6,7];
// klasyczne podejście: pętla for z .push() lub .slice()
const copyItems = [...items];
```

### 4.6 Default parameters
Default parameters czyli domyślne wartości, które zmienna otrzyma jeśli nie przekażemy do funkcji żadnego innego argumentu:
```javascript
//ES5
function doSomething(param1, param2) {
   param1 = param1 || 1;
   param2 = param2 || 'default value';
}

```

```javascript
//ES6
function doSomething(param1 = 1, param2 = 'default value') {

}
```

### 4.7 String interpolation
String interpolation to świetne udogodnienie pozwalające nam pisać m.in multiline stringi lub zrezygnować z concatenacji stringów aby dołączyć do nich jakieś zmienne.

```javascript
//ES5
var url = CONFIG.API + '/users/' + user.id;

var template = '<h1>Hello World<h1>' +
                '<span>Placeholder</span>' +
                '<h2>Next line</h2>';
```

```javascript
//ES6
const url = `${CONFIG.API}/users/${user.id}`;

const template = `
   <h1>Hello World</h1>
   <span>Placeholder</span>
   <h2>Next line</h2>
`;
```
Aby użyć interpolacji należy użyć backtick (klawisz koło "1" na klawiaturze) i opcjonalnie ${} aby użyć zmiennej.

### 4.8 Destructuring
Destructuring to kolejny feature pozwalający usunąć powtarzający się kod. Przykład prezentuje skrót, dzięki któremu możemy stwrorzyć zmienne o tych samych nazwach jak te, które posiada obiekt:

```javascript
//ES5
function parseModel(model){
   var height = model.height;
   var width = model.width;
   
  //...
}
```

```javascript
//ES6
function parseModel(model){
   const {height, width} = model;
}
```

### 4.9 Import/Export/export default
Moduły z ES6 to duży przełom jeśli chodzi o sposób pisania kodu po stronie frontendu. Jeżeli ktoś z was pisał w node.js to wie, że każdy plik jest osobnym modułem dzięki czemu to my decydujemy co zostanie wyeksportowane (pokazane światu na zewnątrz) a co zaimportowane (użyte w danym module). Musimy pamiętać że pełna modułowość nie będzie nigdy dostępna w przeglądarce ze względu na sposób działania skryptów gdzie kazdy plik jest widoczny. Przed specyfikacją ES6 modułowość mogła zostać osiągnięta za pomocą zewnętrznych bibliotek np. require.js.

Każdy plik ma swój moduł. Moduł może importować rzeczy wyeksportowane z innych modułów:
```javascript
//magic.js
function magicFunction() {
  console.log('Magic function');
}

export {magicFunction} // 4.3 Property value shorthand
```

```javascript
//utils.js
import {magicFunction} from './magic';

function moreMagic() {
  magicFunction();
}

```
Należy pamiętać, że tak jak i w świecie node.js `'./'` oznacza bieżący katalog a `'../'` katalog wyżej.
Nie ma obowiązku aby dany moduł coś importował - staje się wtedy modułem niezależnym.
Jeżeli moduł nic nie eksportuje, a zostanie dołączony do builda, to zostanie uruchomiony ale żaden inny moduł nie będzie mógł
skorzystać z jego metod. Jest to bardzo przydatne np. jesli chcemy zarejestrować coś w angularze.

Ostatnią rzeczą wartą wspomnienia jest `export default`. Moduł może eksportować wiele rzeczy np.:
```javascript
function a () {}
function b() {}
function c() {}

export {a,b,c}
```
ale może eksportować tylko jedną 'domyślną' rzecz. Jeżeli coś zostało `export default` to podczas importu możemy pominąć `{}` oraz przypisać tej zmiennej swoją nazwę:
```javascript
//magic.js
function magicFunction() {
  console.log('Magic function');
}

export default magicFunction;
```
```javascript
//utils.js
import magic from './magic';

function moreMagic() {
  magic();
}

```
A teraz najważniejsze: pisząc w ES5 wszystko co nie było owrapowane w `IIFE` rejestrowało przypadkowo lub celowo różne rzeczy w `window`, dlatego w każdym pliku używaliśmy `IIFE` a w nim nasz kod angularowy.
Wraz z ES6 wszystko się zmienia, musimy zapomnieć o tamtym podejściu, ponieważ każdy plik jest modułem - uzyskujemy enkapsulacje jak z `IIFE` oraz `"use strict"` jest włączony automatycznie. Reasumując nie musimy już bać się o dołączanie rzeczy do `global object`, dlatego świadomie rezygnujemy z `IIFE` i `"use strict"`.

### 4.10 Class
Klasy w ES6 to tylko ukrycie prototypowego dziedziczenia. JavaScript nie posiada typowego dziedziczenia obiektowego i `class` wraz z ES6 nic w tym temacie nie zmienia. `Class` posiada specjalną metodę `constructor` która zostanie odpalona tylko raz podczas tworzenia nowego obiektu. Konstruktor to miejsce, w którym najczęściej inicjalizujemy properties danego obiektu lub przeprowadzamy `DI` w AngularJS1 i AngularJS2. 

```javascript
class MyFancyClass {
  constructor(url){
     this.url = url;
  }
}
```

Kod zawarty w klasie jest domyslnie w `"strict mode"`. Klasa podobnie jak `let` i `const` jest odporna na hoisting.
Metody w klasie pomijają słowo kluczowe `function`:

```javascript
class User {
  constructor(user){
    this.user = user;
  },

  getMe() {
     return this.user;
  }
}
```

Podobnie jak w językach obiektowych klasa może posiadać metody satyczne. Oznacza to, że dostęp do takiej metody odbywa się bez konieczności stworzenia instancji takiej klasy:

```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    },

    static distance(a, b) {
        const dx = a.x - b.x;
        const dy = a.y - b.y;

        return Math.sqrt(dx*dx + dy*dy);
    }
}

Point.distance(2,5); //OK!
const p = new Point(3,4);
p.distance(3,4); //niedostępne
```
Jeżeli stworzymy instancję klasy, która posiada statyczne metody automatycznie tracimy do nich dostęp. 

### 4.11 Class inheritance
Dziedziczenie w ES6 jest zdecydowanie łatwiejsze od klasycznego dziedziczenia prototypowego. Nie musimy myśleć o prototypach, konstruktorach itd. Aby stworzyć dziedziczenie dwóch klas należy użyć słowa kluczowego extends:
```javascript
class Animal {
  constructor(name){
    this.name = name;
  }
}

class Kitty extends Animal {
   constructor(name) {
      super(name);
   },
   
   moew() {
     alert(`Moew! ${this.name}`);
   }
}
```
Jeśli dana klasa dziedziczy po innej, musimy wywołać w konstruktorze konstruktor klasy bazowej, dzieje się to za pomocą specjalnej metody `super()`. Jeśli konstruktor klasy bazowej przyjmuje parametry, możemy je przekazać właśnie przez funkcję `super()`.

# 5. TypeScript
### 5.1 Silne typowanie
Używając TypeScript możemy uniknąć wielu testów sprawdzających i walidujących typy zmiennych oraz samych błędów. Jeżeli zadeklarujemy że zmienna jest typu `X` to kompilator TypeScript nie pozowoli nam zbudować projektu dopóki wszystkie referencje przypisania do tej zmiennej nie będą typu `X`.
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
Podczas integracji z zewnętrznymi bibliotekami, często nie posiadamy silnego typowania co powoduje że jesteśmy zmuszeni do pisania własnych interfejsów (o tym za chwilę). Z pomocą przychodzi magiczny typ `any`, który pozwala przypisać dowolny typ do zadeklarowanej tak zmiennej. Pamiętajmy jednak aby go unikać i używać tylko w specjalnych przypadkach (nie po to podpinamy TypeScript aby używać `any`):
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
Zwróćmy również uwagę, że podajemy jawnie typ zwracanej wartości (po nawiasach funkcji) - `number`. Pozwali to WebStormowi zapewnić świetny IntelliSense.

Jeśli funkcja nie zwraca żadnej wartości możemy skorzystać z kolejnego typu - `void`:
```javascript
function echo(msg: string): void {
  console.log(msg);
}
```

TypeScript pozwala nam korzystać również z enumów:
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
* `Boolean`
* `Number`
* `String`
* `Array` lub `type[]`
* `Tuple`
* `Enum`
* `Any`
* `Void`

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
Pamiętajmy że interface to nie dziedziczenie prototypowe, co więcej interface jest usuwany na etapie kompilacji do `*.js`. Interface to coś abstrakcyjnego, pozwalającemu nam tworzyć silne typowanie podczas procesu developmentu i usuwane podczas etapu kompilacji.
### 5.3 Classes
Klasy zostały stworzone aby ukryć nielubiane prototypowe dziedziczenie. Zostały wprowadzone do języka wraz z ES6, a dzięki TypeScript możemy jeszcze więcej!

Deklaracja klasy:
```javascript
class Company {

}
```
Każda klasa posiada specjalną metodę - `constructor`, zostanie ona wywołana tylko raz podczas tworzenia nowego obiektu:
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
TypeScript wymusza na nas abyśmy nie używali słowa kluczowego `function` w obrębie klasy:
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
     super();
     this.setPage(10);
  }
}
```
Część z was zapyta czym w takim razie różni się `interface` od klasy `abstract`? W najprostszych słowach za pomocą `abstract` możemy przedstawić przykładową implementację danych metod. W przypadku interfejsów nie jest to możliwe, ponieważ `interface` to przepis na klasę. Po drugie `abstract class` stworzy nam dziedziczenie prototypowe, a `interface` zostanie usunięty podczas kompilacji TypeScripta (nie ma czegoś takiego jak `interface` w JavaScripcie).

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
`Generics` to kolejna rzecz zapożyczona z języków obiektowych. Służy do stworzenia jak najbardziej re-używalnego kodu. Przykład:
```javascript
function echo(value: any): any {
  return value;
}
```
W tym wypadku tworzymy funkcję, która przyjmuje dowolny argument i go zwraca. Jest to trochę oszukany `Generic`, ale pozwala zrozumieć całą ideę. Do funkcji `echo` możemy przekazać dowolną wartość, bez względu na jej typ zostanie ona zwrócona. A teraz zobaczmy jak możemy implementować `Generics` w TypeScript:
```javascript
function echo<T>(value: T): T {
  return value;
}
```
Zasada działania powyższych 2 funkcji jest identyczna. `T` - jest to placeholder, może być to dowolnie inna litera, w innych językach programowania przyjęło się aby używać litery `T`.
```javascript
let num: number = <number>echo(12);      //12
let str: string = <string>echo('Hello'); //'Hello'
```
Nasza metoda stała się generyczna - oznacza to że przyjmuje dowolne typy zmiennych.

### 5.5 Decorators
Dekoratory to eksperymentalny feature TypeScripta. Pozwala nam w łatwy sposób "dekorować" lub innymi słowy dołączać metandane do naszych klas, funkcji czy też properties. AngularJS 2 wykorzystuje dekoratory do tworzenia `Component` oraz `Service`.
```javascript
// AngularJS 2
@Component({
  selector: 'app',
  template: '<h3>Hello world!</h3>'
})
class MyAppComponent {
}

//AngularJS 2 Service
@Injectable()
export class UserService {
  getUsers() {
    //fetch users
  }
}
```
Oprócz samych klas możemy dokorować również metody:
```javascript
class SubjectClass {

   @meta()	
   doSomething(){
   }
}
```

Ale po co nam tak na prawdę dekoratory? Jak wygląda ich przykładowa implementacja?

Wyobraźmy że budujemy kolejną nudną aplikacje w `express.js`. Za każdym razem tworzymy funkcję, która jest Routerem, do funkcji tej dołączamy różne routes gdzie obsługujemy nasze requesty. Możemy przełamać tę konwencję i wykorzystać dekoratory:
```javascript

@Controller('/job')
class JobController {

  @get('/:id')
  getJobById(id: number): IJob {
     // call db
  }
  
  @put('/:id')
  updateJoById(id: number): IJob {
     // call db
  }
}
```

Czy praca z takimi klasami nie byłaby o wiele łatwiejsza? Przeglądając taką klasę, wiemy dokładnie co robi każda metoda, możemy również dekorować każdą ścieżkę np. middlewarami.

Drugi przykład to coś co może zmienić wasze podejście do pisania aplikacji w AngularJS 1. Co by się stało gdybyśmy mogli tworzyć komponenty identycznie jak w AngualrJS 2?

```javascript
//Przykładowy komponent AngularJS 1
angular.module('app.components.myApp', [])
   .component('myAppComponent', {
       bindings: {
          magicParam: '='
       },
       template: `<h1>Hello!</h1>`,
       controller: function() {
       }
   });
```
A teraz przykład z dekoratorem:
```javascript
@Component({
  bindings: {
     magicParam: '='
  },
  template: `<h1>Hello!</h1>`
})
class MyAppComponent() {
}
```

I sama deklaracja dekoratora:
```javascript
export const Component = function(options) : Function {
   return (controller: Function) => {
      return options? angular.extend(options, {controller}) : controller;
   }
}
```

Jak widać dekorator to nic strasznego, jest to po prostu funckja, która zwraca funkcję. Funckja może modyfikować prototypy, rozszerzać metody, klasy czyli wszystko to co możemy zrobić w JavaScripcie. W powyższym przykładzie `MyAppComponent` to nasz `controller`, natomiast dekorator do dodatkowe properties, które kopiujemy do naszej konfiguracji. Po kompilacji kod będzie identyczny jak ten w klasycznym podejściu z Angular1.

### 5.6 Typings
Typings to idealnym przykład wkładu społeczności w rozwój języka. Wyobraźmy sobie że podpinamy do naszego projektu `ui-router`, a następnie konfigurujemy go w naszej aplikacji dodając różne stany itp. Skąd mamy wiedzieć jakie metody posiada service `$state` albo provider `$stateProvider`? Nie wiemy, musimy przejść do dokumentacji, tam zobaczyć jakie funkcje są nam udostępniane, a następnie wrócić do kodu i jeszcze sprawdzając 5x dokumentację przepisac daną metodę/własność. Inną opcją jest tworzenie własnych interface'ów tak aby ułatwić nam życie np. w jednym miejscu stworzyć katalog `typings` a tam zadeklarować `interface` `IState` lub `IStateProvider` przepisując definicje funkcji 1:1 zgodnie z dokumentacją. 

A co jeśli powiem wam, że ktoś zrobił to już za was? :)
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
Na tym właśnie polega cała idea TypeScripta - typowanie, klasy, dziedziczenie i darmowe typingsy. Budując model aplikacji w osobnych klasach, typując odpowiedzi z serwera, korzystając z generics i typingsów cały flow wszystkich programistów jest o wiele szybszy i odporny na literówki i błędne typowania. Zapraszam do analizy staretera i przykładowej aplikacji, gdzie powyższa teoria została zaprezentowana w praktyce.

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
- **Avoid wildcard imports**

# 7. Starter
### 7.1 Struktura plików
### 7.2 Opis
### 7.3 Lintery

# 8. Przykładowa aplikacja

# 9. Co dalej?
Celem tego guideline'a jest zaznajomienie jak największej ilości developerów z nowym stackiem technologicznym oraz przygotowanie ich aby w jak najłatwiejszy sposób przestawili się na nowy ekosystem AngularJS2. Prawdopodobnie dopiero w Q4 2016 community będzie na tyle rozwinięte, że będziemy w stanie przejść ze stackiem na AngularJS2. Do tego czasu warto zmienić swój sposób myślenia i zapomnieć o wszystkich starych nawykach.

Planuję rozwijać i updatować ten guideline do Q4 2016 kiedy oficjalnie przejdziemy na AngularJS 2. AngularJS w wersji pierwszej jeszcze długo nie umrze i już za niedługo możemy spodziewać się nowej wersji 1.6. Dlatego na pytanie czy warto uczyć się tego, jeżeli zaraz dostaniemy nową wersję Angulara odpowiem, że warto bo po pierwsze uczycie się nowego stacka, który w dużej ilości pokrywa się ze stackiem AngularJS 2, nabywacie wiedzy o good pratcies, zmieniacie również swoje myślenie na myślenie modułowe i componentowe. Siadając i ucząc się nowego Angulara z powyższą wiedzą wszystko będzie dla was o wiele łatwiejsze!

# 10. Reference i uwagi
- [John Papa's ng1 styleguide - bardziej jako antypatterny](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md)
- [John Papa's ng2 styleguide](https://github.com/johnpapa/angular-styleguide/blob/master/a2/README.md)
- [Todd Motto ng1 styleguide](https://github.com/toddmotto/angular-styleguide)
- [Airbnb ES6 styleguide](https://github.com/airbnb/javascript)
- [Lessons learned from PayPal](https://medium.com/@bluepnume/sane-scalable-angular-apps-are-tricky-but-not-impossible-lessons-learned-from-paypal-checkout-c5320558d4ef#.xbx6x2ltv) 
- [Todd Motto - blog](https://toddmotto.com/)
- [Todd Motto - ngMigrate](http://ngmigrate.telerik.com/)
- [Angular Performance - Todd Motto](https://www.youtube.com/watch?v=LoIuokh6NUI&list=WL&index=6)
- [TypeScript - official handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html)
- [Instagram team - Webpack howto](https://github.com/petehunt/webpack-howto)

W razie znalezienia jakichkolwiek niejasności, literówek i (mam nadzieję że nie) błędów ortograficznych proszę o otwarcie PR. Jeżeli ktoś chciałby napisać jakiś dodatkowy rozdział/ zmienić istniejący/ napisać lepsze przykłady to bardzo do tego zachęcam i jestem otwarty na wszelkie propozycje.
