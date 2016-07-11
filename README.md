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

Ostatnim zagadnieniem jest rozszerzanie interfejsu lub klasy o inne interfejsy:
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
### 5.4 Generics
### 5.5 Enums
### 5.6 Decorators
### 5.7 Typings

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
