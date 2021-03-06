# 1. Wstęp
Niniejszy styleguide zawiera bogaty wstęp teoretyczny potrzebny do zrozumienia działania startera i przykładowej aplikacji napisanych w modułowym podejściu za pomocą AngularJS 1.5.7 , webpack i TypeScript. 

Rozdział 2 opisuje breaking changes, które zaprezentował AngularJS od wersji 1.5. Nastepny rozdział to krótkie wprowadzenie do webpacka. Zaznaczam tutaj, że prawdopodobnie będziemy posiadać jedną konfigurację dzieloną pomiędzy projektami i zazwyczaj sprowadzi się do kopiowania, dlatego pamiętajcie, że nie musicie znać całego API webpacka aby czuć się w nim swobodnie. Konfiguracja jest na tyle elastyczna, że na pewno każdy da radę skonfigurować ją pod swój projekt, a na 99% przyjdziecie na gotowe ;). 

Dalej w 4 i 5 rozdziale znajduje się spis wszystkich potrzebnych wam informacji dotyczących ES6 i TypeScripta. Następnie rozdział 6 to przejście do samego styleguida czyli spisu best-practices używanych podczas pisania aplikacji. Na koniec prezentacja działającej aplikacji, którą polecam odpalić w WebStormie - otrzymujemy możliwość m.in skakania po referencjach i podpowiedzi Intellisense.

# 2. Zmiany w AngularJS 1.5+
### 2.1 Components
**Wszystko jest komponentem.** To zdanie musicie zapamiętać przez najbliższe kilka lat pisząc aplikacje frontendowe. `React`, `Polymer`, `WebComponents`, częściowo `AngularJS 1` (dyrektywy) i `AngularJS 2` - wszystkie te biblioteki/standardy/frameworki wskazują jasny rozwój i kierunek w jakim będziemy (lub już zaczeliśmy pisać) nasze aplikacje.

**Czym jest komponent?**
Komponent to nie tylko funkcja ze swoją logiką i templatem HTML. Komponent to coś niepodzielnego, unitowego, zawierającego swoje zależności, często udostępniającego publiczne API (przez atrybuty w HTML), posiadającego wbudowane style, template HTML a co najważniejsze enkapsulującego wszystkie te rzeczy w jedną całość.
Dyrektywy w `AngularJS` były pierwszym krokiem w stronę tworzenia komponentów. Jeżeli chcemy zobaczyć jak powinny wyglądać te prawdziwe zgodne ze standardem W3C powinniśmy spróbować stworzyć jakiś w Polymerze. Budując aplikacje z wykorzystaniem jakiegoś frameworka/biblioteki idealnymi kandydatami są `React` i `AngularJS 2`.

Developerzy Googla udostępnili nam komponenty w `AngularJS 1` od wersji `1.5.0`. Musimy jednak pamiętać, że to tak na prawdę 'syntactic sugar' na dyrektywie i daleko im do tych prawdziwych.

Standardowe podejście z AngularJS 1 - dyrektywa:
```javascript
.directive('directiveName', function() {
   return {
     //konfiguracja:
     //link, controller, preLink, postLink, compile
     //inne X opcji 
   }
});
```

Nowe podejście od AngularJS 1.5.0 - component:
```javascript
.component('myComponent', {
   bindings: {}, // zastępuje nam scope: {} 
   controller: function() {
     //dostęp do bindings za pomocą this.propName
   },
   template: `{{$ctrl.propName}}`,
   templateUrl: 'path/to/template', //1:1 jak w .directive
   controllerAs: '$ctrl', //domyślnie dla każdego  komponentu (możemy pominąć)
   require: {}, //do komunikacji pomiędzy dyrektywami
   transclude: false //tak jak w .directive ale dodatkowo multi-slot transclusion (podrozdział 2.5)
});
```

Podsumowując powyższy snippet `.component` udostępnia nam **tylko 7 opcji!** Koniec z miliardem propertisów w konfiguracji dyrektywy.

Używając `component` pozbywamy się: 
* compile
* preLink
* postLink
* bindToController
* controllerAs (automatycznie zostaje przypisany alias `$ctrl`,  jeśli chcemy możemy go nadpisać)
* priority
* restrict (automatycznie `restrict: 'E'`)
* scope (automatycznie zawsze izolowana)
* terminal
* templateNamespace
* multiElement

Template w dyrektywie dostaje automatyczny dostęp do wszystkich właściwości `controllera` za pomocą zmiennej `$ctrl`.
Weźmy za przykład poniższy komponent:
```html
<my-component config="config"></my-component>
```
```javascript
.component('myComponent', {
   bindings: {
     config: '='
   },
   controller: function() {
     console.log(this.config);
   },
   template: `{{$ctrl.config}}`
});
```

Możemy powiedzieć, że w `template` `{{$ctrl.config}}` to to samo co `this.config` w `controller` i to samo co `config` w `bindings: {}` oraz `config` na tagu HTML.


### 2.2 One-way databinding
One-way databinding uzyskujemy poprzez skonfigurowanie jakiejś zmiennej w `bindings` za pomocą operatora `<`. Oznacza to, że wszystkie zmiany rodzica zostaną przeniesione na dziecko, ale jakakolwiek zmiana w dziecku (componencie) zostanie zignorowana przez rodzica. 

```javascript
.component('myComponent', {
   bindings: {
      propA: '=', //object
      propB: '@', //string
      propC: '&', //expression
      propDL '<' //nowość - one-way databinding
   },
   controller: function() {
      this.propDL = 0; //zostanie zignorowane przez rodzica
   }
});
```
### 2.3 Stateless components
Stateless component to komponent, którego jedynym zadaniem jest wyświetlanie szablonu na podstawie przekazanych do niego danych.
Przykładowa implementacja: 

```javascript
const StatelessComponent = {
  bindings: {
    people: '<'
  },
  template: `
    <ul>
       <li ng-repeat="person in $ctrl.people">
         <span>{{person.fullName}}</span> - <span>{{person.age}}</span>
       </li>
    </ul>
  `
};

angular.module('app.components.statelessComponent', [])
       .component('statelessComponent', StatelessComponent);
```

### 2.4 Lifecycle hooks
`Lifecycle hooks` to funkcje do których ma dostęp komponent i które dotyczą jego cyklu życia. Od `Angular 1.5.0` mamy dostęp do 4 funkcji, które zostaną wywołane automatycznie przez `AngularJS` w określonych sytuacjach:

**`$onInit`** - zostanie wywołany gdy wszystkie `bindings` zostaną zainicjalizowane, dzięki czemu będziemy mieli do nich dostęp. Funkcję tą można traktować jako `constructor` w klasie.

```javascript
.component('myComponent', {
  bindings: {
    prop: '='
  },
  controller: function() {
    this.$onInit = function() {
       console.log(this.prop);
    }
  }
});
```

Jeśli tworzymy komunikacje pomiędzy dwoma komponentami należy pamiętać, że `$onInit` zostanie wywołany **po** inicjalizacji `controllera` rodzica. Co więcej dostęp do niego otrzymujemy poprzez `this.parent`, a nie jak w `dyrektywie` przez przedostatni parametr funkcji link: `link: function(scope, element, attrs, ctrl, transcludeFn)`

```javascript
.component('parentComponent', {...})
.component('childComponent', {
  ...
  require: {
    parent: '^parentComponent'
  },
  controller: function() {
    this.$onInit = function() {
      //dostęp do API rodzica
      console.log(this.parent.someFunction());
    }
  }
});
```

**`$postLink`** - zostanie wywołany gdy wszystkie dzieci przeszły `compile phase` i `linking phase` oraz nasz template jest gotowy na manipulacje DOM.

```javascript
.component('myComponent', {
  bindings: {
    prop: '='
  },
  controller: function() {
    this.$postLink = function() {
      //jeżeli potrzebujemy mieć dostęp do element, attrs musimy skorzystać z .directive
    }
  }
});
```

**`$onDestroy`** - zostanie wywołany gdy nasz komponent otrzyma `event` `$destroy` - użyteczne podczas zwalniania pamięci i innego czyszczenia po dyrektywie.

```javascript
.component('myComponent', {
  controller: function() {
    this.$onDestroy = function() {
      //identycznie jak w .directive scope.$on('$destroy', function(){...})
    }
  }
});
```

**`$onChanges`** - zostanie wywołany podczas wykrycia zmiany w `bindings` (także podczas pierwszej inicjalizacji) tylko gdy wiązanie jest typu `<` lub `@`.

```javascript
.component('myComponent', {
  bindings: {
    propA: '<',
    propB: '@'
  },
  controller: function() {
    this.$onChanges = function(changes) {
      // zakładając że parent zmienił propA, otrzymamy:
      //changes: {
      //   propA: {
      //      currentValue: value,
      //      previousValue: value
      //   }
      //}
      
      // Dodatkowy mamy dostęp przez prototype do specjalnej metody .isFirstChange()
      // changes.propaA.isFirstChange();
    }
  }
});
```

### 2.5 Multi-slot transclusion
Transclusion w kontekście AngularJS to wstawianie HTML, zawartego pomiędzy znacznikami naszej dyrektywy, we wcześniej oznaczone w niej miejsce np.:

```HTML
<transclude-demo>
   <p>Transclude</p>
</transclude-demo>
```

```javascript
.directive('transcludeDemo', {
   transclude: true,
   scope: true,
   replace: true,
   template: '<div>Directive and <span ng-transclude></span><div>'
});
```

Wynik:
```HTML
  <div>Directive and <span>Transclude</span><div>
```
Aby stworzyć `transclude` wystarczy udekorować dany node HTML przez dyrektywę `ng-transclude`. Problem polega na tym, że nie jesteśmy w stanie wybrać co chcielibyśmy transcludować i z automatu jest to cały HTML zawarty pomiędzy znacznikami naszej dyrektywy. Od AngularJS 1.5.x otrzymujemy wsparcie dla tzw. multi slot transclusion. Oznacza to że możemy wstrzyknąć kod HTML w wiele miejsc naszego template:

```HTML
<multi-transclude>
	<p>Transclude 1</p>
	<h1>Transclude 2</h1>
</multi-transclude>
```

```javascript
.component('multiTransclude', {
   transclude: {
      'slotName1': 'p',
      'slotName2': 'h1',
      'slotName3': '?a'
   },
   template: `
     <div>
        Static text
        <div ng-transclude="slotName1"></div>
        Other divider
        <div ng-transclude="slotName2"></div>
     </div>
   `
});
```

Pierwszą zmianą jest to, że property `transclude` przyjmuje tym razem obiekt konfiguracyjny. Kluczem tego obiektu jest nazwa danego slotu, do którego będziemy się później odwoływać w template, natomiast wartość to node HTML, który chcemy transcludować (może przyjąć również inną dyrektywę). Jeżeli tagi nie będą unikalne tj. pomiędzy znacznikami naszej dyrektywy znajdą się np. 2 elementy `<p>` to zostanie wybrany ten ostatni.

Aby oznaczyć opcjonalność slotu wystarczy oznaczyć go przez `?tagName`. Na końcu wystarczy wskazać miejsce w template gdzie chcemy wstrzyknąć transclude przez dyrektywę `ng-transclude="nazwa_slotu"`.


# 3. Webpack jako module bundler
### 3.1 Czym jest webpack?
Webpack to tzw. module bundler, czyli narzędzie pozwalające na analizę zależności modułów aplikacji i generowanie specjalnie przygotowanych fragmentów statycznego kodu (ang. chunks). Webpack pozwala za pomocą specjalnej konfiguracji zarządzać działaniem całej aplikacji. Gdy korzystamy z webpacka, każdy plik staje się modułem, oznacza to, że jeśli jakiś plik nie został dołączony w innym pliku przez `require` lub `import`, nie zostanie on dołączony do wynikowego `bundle`. "Pod spodem" tworzone jest tak na prawdę drzewo zależności, które możemy wygenerować dla każdego projektu:

![drzewo-zależności](https://camo.githubusercontent.com/875cc4cba72d9d261651bb0fefa8ff76f756087c/68747470733a2f2f7777772e657665726e6f74652e636f6d2f73686172642f7332312f73682f66353935366638332d626136382d343037352d383133342d6261633234663137333832362f33323630356338386232383238373836656463346435393239643136643539362f646565702f302f6d6f64756c65732e706e67)

Niesie to ze sobą wiele udogodnień:
- możemy obliczyć `diff` na grafie tj. wyciągnąć część wspólną i załadować ją do osobnego pliku - odświeżanie przeglądarki po zmianie w plikach, które nie są wspólne, odbywa się zdecydowanie szybciej
- możemy zaimplementować lazy-loading na wybranych fragmentach grafu
- od Webpacka w wersji 2 możemy skorzystać z `tree-shaking`. Oznacza to, że jeśli na danym module użyjemy `import` lub `require` ale nie użyjemy żadnej weksportowanej funkcji to taki moduł zostanie wyrzucony z drzewa zależności

Webpack to świetne narzędzie, które w całości zastępuje task runnery. Nasza aplikacja staje się 'inteligentna' , świadoma kontentu i zależności między nimi. Jedyny minus webpacka to to, że posiada duży próg wejścia. Jednak z własnego doświadczenia mogę powiedzieć, że po pewnym czasie wszystko staje się oczywiste i nie wyobrażam sobie powrotu do pisania tasków w `Gulp` lub `Grunt`.

### 3.2 Opis funkcjonalności
Co możemy zrobić z webpackiem? Niemal wszystko! 
Jeżeli brakuje nam jakiejś funkcjonalności wystarczy, że zainstalujemy jakiś loader lub plugin z ogólnie dostępnego katalogu npm.
W ostateczności nic nie stoi na przeszkodzie aby napisać swój własny ;)

Webpack pozwala nam m.in:
- Pre-procesować dowolny plik za pomocą tzw. loaderów. Przykład działania loadera: dla plików z rozszerzeniem *.js z katalogu app użyj babela tak aby outputem był ES5. Inny przykład: dla zdjęć poniżej 100kB zamień je na base64 i wstaw je bezpośrednio do pliku css.
- Używać pluginów na całym bundlu np. minifikacja JavaScriptu, generowanie automatycznie index.html, wyciągnięcie części wspólnej i zapisanie do osobnego bundla, otwarcie przeglądarki po zbudowaniu projektu czy nawet natywne notyfikacje systemowe powiadamiające nas o błędach/sukcesie builda (i wiele wiele więcej [npm webpack-plugins](https://www.npmjs.com/search?q=webpack-plugin))
- możliwość stworzenia `lazy-loading` z `ui-router` dla AngularJS 1 (postaram się napisać coś o tym za jakiś czas)
- użycie `webpack-dev-server` czyli serwera pozwalającego w całości zastąpić nam `BrowserSync` 
- korzystanie z `AMD` lub `CommonJS` - wedle naszego uznania
- ręczne ustawianie `process.env`
- wyszukanie duplikacji kodu i usunięcie tego zbędnego!
- generowanie `source-map` 
- `HMR` (ang. hot module replacement) - pozwala na odświeżanie naszej aplikacji podczas zmian w taki sposób aby trzymała swój stan - przydatne dla `Reacta` , `AngularJS 2`.
- natywne wsparcie dla modułów `ES6`

Oraz wiele innych, zapraszam do wgłębienia się w Webpacka we własnym zakresie! 
[Oficjalna dokumentacja](https://webpack.github.io/docs/what-is-webpack.html)

### 3.3 Przykładowa konfiguracja

Poniższy snippet zawiera większość koncepcji webpacka, z którymi spotkacie się w projektach. Należy pamiętać, że jedna konfiguracja to zazwyczaj za mało. Warto stworzyć osobną dla builda, testów i procesu developmentu lub ifować fragmenty kodu w zależności od `process.env`. Konfigurację wg. konwencji umieszczamy w pliku `webpack.config.js`:

```javascript
//webpack.config.js 
//Przykładowa konfiguracja projekt z ES6 i SASS
const path = require('path');
const webpack = require('webpack');

// Webpack wspiera system 3rd party pluginów, wystarczy znaleźć interesujący w npm i go pobrać

//automatyczne otwieranie nowego okna a'la BrowserSync
const OpenBrowserPlugin = require('open-browser-webpack-plugin');
//kopiuje pliki/katalogi podczas builda
const CopyWebpackPlugin = require('copy-webpack-plugin');

//helpery, aby nie hardcodować stringów, warto wypchać je do osobnego pliku config.js
const app = path.join(__dirname, 'app');
const public = path.join(__dirname, 'public');
const port = 3000;

module.exports = {
    //entry czyli "wejście" do naszej aplikacji z którego webpack zacznie tworzyć drzewo zależności
    // entry może przyjmować wiele "wejść" - przydatne przy lazy loadingu
    entry: {
        'app': './app.js'
    },
    //output czylu "wyjście" naszej aplikacji
    //poniższa konfiguracja oznacza: weź każde entry i stwórz plik np. app.bundle.js (name = klucz z entry),
    //umieść go w katalogu public: "/public/app.bundle.js"
    output: {
        path: public,
        filename: '[name].bundle.js'
    },
    //konfiguracja webpack-dev-server, który zastępuje w całości browserSynca
    devServer: {
        //serwuj dane z katalogu public (powiązane z output)
        contentBase: public,
        outputPath: public,
        // wsparcie dla HTML history api
        historyApiFallback: true,
        //gzipuj bundle
        compress: true,
        //auto-refresh po wykryciu zmiany
        hot: true,
        inline: true,
        //nasłuchuj na porcie
        port: port
    },
    module: {
        //Dla webpacka wszystko jest JavaScriptem, dlatego jeśli używamy innych rozszerzeń np. scss
        //musimy przekazać mu informację w jaki sposób powinien wczytać dany plik i przekonwertować go na js
        loaders: [
            //test - regex na dane rozszerzene pliku
            //loader - nazwa loadera instalowanego przez npm
            //query - opcje konfiguracyjne danego loadera
            //include/exclude - regex na dane directory
            
            //Poniższa konfiguracja oznacza: dla wszystkich plików w katalogu app z rozszerzeniem *.js
            //użyj babela (konwersja ES6 -> ES5)
            {test: /\.js$/, loader: 'babel', query: {presets: ['es2015']}, include: /app/},
            
            //loadery są aplikowane od prawej do lewej tj dla plików *.scss otrzymamy:
            //sass-loader -> css-laoder -> style-loader
            {test: /\.scss$/, loader: 'style!css!sass', include: /app/}
        ]
    },
    resolve: {
        //gdzie webpack ma szukać modułów
        //np. node_module bower_components itp
        modulesDirectories: [
            'node_modules'
        ],
        //tablica rozszerzeń, która zostanie użyta do resolve modułów
        //np. jeśli moduł jest zapisany w CoffeeScript powinniśmy umieścić w tablicy .coffe
        //extensions są brane od prawej do lewej
        extensions: ['', '.js'] 
    },
    //rejestracja, konfiguracja pluginów
    plugins: [
        new OpenBrowserPlugin({
            url: `http://localhost:${port}`
        }),
        new CopyWebpackPlugin([
            {from: 'mocks'}
        ])
    ]
};
```


# 4. ES6
### 4.1 Const i let
Pisząc w ES6 powinniśmy całkowicie zrezygnować z `var`. Co więcej powinniśmy w 90% przypadków używać `const` czyli oznaczać zmienne jako niemodyfikowalne. `const` zapewnia, że referencja do danej zmiennej nie zostanie zmieniona, a każda próba takiej zmiany spowoduje błąd. Jeśli nasza zmienna jest mutowalna powinniśmy użyć `let`. 

```javascript
let counter = 0;
counter++;
```

```javascript
const url = `${CONFIG.API}/path/${id}`;

$http.get(url)
```

Różnica pomiędzy `const`, `let` oraz `var` jest taka, że `const` i `let` są `block-scoped`, a `var` jest `function-scoped` oznacza to, że są widoczne tylko w obrębie najbliższego bloku a nie funkcji. Co więcej w przypadku `let` i `const` nie działa hoisting, czyli nie możemy użyć zmiennej przed jej deklaracją.

### 4.2 Method shorthand
Korzystając z ES6 możemy pominąć słowo kluczowe `function` w deklaracjach obiektów czy też samych klasach:
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
Jest to jeden z moich ulubionych 'ficzerów' ES6, pozwala nam pominąć powtarzający się kod. Skoczmy od razu do praktycznego przykładu:
```javascript
  .service('AuthService', function($http) {
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

Arrow function jest to skrócona wersja zapisu anonimowej funkcji, często upraszcza nasz kod w takim stopniu, że możemy zapisać jej logikę w jednej linijce kodu. Przy korzystaniu z arrow function należy pamiętać że `this` będzie wskazywał na kontekst w którym funkcja została wywołana. `call` , `apply` i `bind` nie mogą zmienić `this` w arrow function, ponieważ arrow function nie tworzy własnego this. Jeśli potrzebujemy zmienić `this` musimy skorzystać ze zwykłej funkcji.

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

`Spread` operator znajduje również zastosowanie w kopiowaniu tablic:
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
String interpolation to świetne udogodnienie pozwalające nam pisać m.in multiline stringi lub zrezygnować z konkatenacji stringów aby dołączyć do nich jakieś zmienne.

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
Destructuring to kolejny feature pozwalający usunąć powtarzający się kod. Przykład prezentuje skrót, dzięki któremu możemy stworzyć zmienne o tych samych nazwach jak nazwy atrybutów, które posiada obiekt:

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
Używając TypeScript możemy uniknąć wielu testów sprawdzających i walidujących typy zmiennych oraz samych błędów. Jeżeli zadeklarujemy że zmienna jest typu `X` to kompilator TypeScript nie pozwoli nam zbudować projektu dopóki wszystkie referencje przypisania do tej zmiennej nie będą typu `X`.
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

Posumowując za pomocą TypeScripta dostajemy możliwość korzystania z silnego typowania w naszym kodzie JavaScript. Wyróżniamy nastepujące typy:
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
- **Unikaj jQuery i lodash**

Zgodnie z nowym podejściem pisania aplikacji z wykorzystaniem `components` tracimy dostęp do `DOM` dyrektywy. Wciąż możemy używać tradycyjnej składni jeśli musimy manipulować `HTML`. Na pytanie czy potrzebujemy `jQuery` każdy musi odpowiedź indywidualnie w kontekscie projektu. Warto jednak zrezygnować z dodatkowej zależności na rzecz `jqLite`.

To samo tyczy się `lodash/underscore` - pisząc w ES6 większość metod została wbudowana w język. Na podstawie ostatniego projektu komerycjnego mogę potwierdzić, że brak `lodasha` jest bolesny tylko na początku a z czasem zaczynamy doceniać sam język.

- **Wszystko jest komponentem**

I to dosłownie. Nawet strona z katalogu `pages` to nie zlepek `ng-controller` i `HTML` tylko pełnoprawny `componenent`, z własnym `scope` , `template` i logiką. Nowa wersja `ui-router` wspiera routing po komponentach. W tradycyjnym podejściu konfiguracja może być nastepująca:

```javascript
$stateProvider.state('app.pages.dashboard', {
    url: '/dashboard',
    views: {
        'main@': {
            template: '<dashboard-page></dashboard-page>',
        },
    },
    data: {
        permissions: {
            only: [ROLE.ADMIN],
            redirectTo: 'app.pages.login',
        },
    },
});
```

Rezygnujemy również z `controller` w konfiguracji state'a. Pamiętajmy że `resolve` powinien posiadać logikę związaną z autentykacją/redirectem a nie fetchowaniem danych dla danego komponentu. Dane dociągamy w samym komponencie wyświetlając przy tym piękny loader. Dbajmy o user experience i nie rzucajmy białą/pustą stroną tylko po to zeby z niewiadomych przyczy pobierać dane w `ui-router`.

- **Przestań używać controller, controllerAs, ng-controller**

Jest to związane z wcześniejszym punktem. Skoro wszystko jest komponentem (każda strona) nie potrzebujemy angularowych controllerów.
Jedyny `controller` jaki ma prawo istnięć to ten wbudowany w `component`. Jeśli używasz `controller` gdziekolwiek w aplikacji to utrudniasz migrację projektu do `AngularJS 2`.

- **Zrezygnuj z factory i provider na rzecz service**

Zapomnij o `provider` i `factory`. Tak jak w `AngularJS 2` mamy tylko `service` tak i my powinniśmy dążyć do ujednolicenia typów naszych serwisów. Przykładowy `service`:

```javascript
import User from '../models/user.model';

export default class UserService {

    public user: User;

    constructor(private $http: IHttpService) {
        'ngInject';
    }

    public setUser(user: User): void {
        this.user = user;
    }
    
    public getUser(): angular.IPromise<User> {
        return $http.get(...);
    }
}
```

Serwis to po prostu klasa, która możemy wstrzyknąć przez `constructor` do naszego komponentu. Więcej o `DI` w `TS` w kolejnych punktach.

- **Spójrz inaczej na dyrektywy**
Jeśli myślisz w tej chwili o dyrektywie to tak na prawdę myslisz o komponencie. Dyrektywa to wciąż coś co jest dostępne w `AngularJS 2` ale ich zastosowanie jest inne. 

> Dyrektywa to komponent bez widoku. Jej zadaniem jest modyfikacja zachowania danego węzła HTML. 
> **~Paulo Coelho**

Podsumowując powyższe zdanie, dyrektywa to np. `ng-if` , `ng-switch`, ponieważ **modyfikują** dany węzeł/węzły HTML ale **nie posiadają** własnego widoku.
Przykład customowych dyrektyw: podświetlanie tekstu, transformacje CSS, animacje itp.

- **Zrezygnuj z $rootScope**

Jest to raczej zasada oczywista, ale nie dla wszsystkich. `$rootScope` używamy tylko jeśli musimy zmienić coś poza `ui-view` gdzieś gdzie nie mamy dostępu. Użycie `$rootScope` jest też usprawiedliowne podczas `$broadcast` eventów w dół np. gdy side menu zostało zamknięte.
- **Sposoby komunikacji rodzic-dziecko, dziecko-rodzic**

Uwaga! Unikajmy `$broadcast` i `$emit` jako sposobu synchronizacji naszych komponentów, traktujmy to jako ostateczność!

Sposoby komunikacji `rodzic-dziecko`:

1. przekazywanie parametrów do dyrektywy w dół przez atrybuty w `HTML`
2. serwis (synchronizacja modelu)
3. `$broadcast` - rodzic wymusza wywołanie metody na dziecku

Sposoby komunikacji `dziecko-rodzic`:

1. wywoływanie funkcji przez `bindings` i `&`
2. `$watch` na zmiennych z `bindings` typu `=`, `<`, `@`
3. serwis (synchronizacja modelu)
4. dostęp do `controller` rodzica przez `require` i `this.parent`

- **Zapomij o $scope, chyba że potrzebujesz $watch**

Jedynym sensownym wytłumaczenia korzystania ze `$scope` jest użycie `$watch`. Od AngularJS w wersji 1.5.x nasz template zostaje automatycznie zbindowany do zmiennej `$ctrl` dlatego nie musimy korzystać również z `var vm = this`. 

- **Używaj ES6/TS class tam gdzie jest to możliwe**

Korzystanie z ES6 powinno być oczywiste. Jest to standard od 2015 roku, który jest wspierany prawie w całości przez większość przeglądarek. W sekcji 4 opisałem najważniejsze zmiany w języku,a szcegółowy opis wszystkich zmian można znaleźć [tutaj](http://es6-features.org/#Constants).

TypeScript to kwestia indywidualna. Część z was może nie zgodzić się z koniecznością używania `TypeScript` w projektach. Jednak z własnego doświadczenia mogę powiedzieć, że aplikacja napisana w całości w `TypeScript` staje się łatwa w utrzymaniu, odporna na literówki, dziwne błędy odkrywane poczas runtime, nie musimy pisac testów sprawdzających typ zmiennych i co najważniejsze otrzymujemy `Intellisense` dla naszych modeli i funkcji.

- **Unikaj two-way databinding**

Chociaż jest to bardzo wygodne powinniśmy unikać `two-way databinding`. Obserwując obecny rozwój `React` i `AngularJS 2` widzimy że popularne stało się myślenie `one-way`. Pozwala to w dużej mierze pisać bardziej wydajny kod, unikać mutowalności danych co przekłada się na odporność na błędy. Myśląc również w kontekscie migracji do `AngularJS 2` powinniśmy korzystać w naszych komponentach tylko z operatora `<` i wybrać jeden z wielu sposobów komunikacji z pkt. "Sposoby komunikacji rodzic-dziecko, dziecko-rodzic".

- **Używaj 'ngInject'**

Pisząc aplikację z wykorzystaniem `TypeScripta` zmienia się sposób `DI` (wstrzykiwania zależności) do naszych serwisów/dyrektyw i komponentów. Aby wstrzyknąć coś do przykładowego komponentu musimy podać typ danej zmiennej i udekorować `constructor` przez `ngInject`. Komentarz ten pozwoli webpackowi (przez `ng-inject-loader`) stworzyć inline tablicę zależności znaną z klasycznego `AngularJS` np. (`["$scope", "$http", function($scope, $http) {}]`).

```javascript
class ModalComponent{

    constructor(
        private $scope: ng.IScope,
        private $document: ng.IDocumentService
    ) {
        'ngInject';
    }

    public $onInit() {
        this.$document.on('keydown', ($event) => this.onKeyEntered($event));
    }

    public $onDestroy() {
        this.$document.off('keydown');
    }
    
    public onKeyEntered($event) {
       console.log($event);
    }
}
```

- **Wszystko jest modułem**

Korzystając z webpacka i ES6 wszystko staje się modułem. Dlatego nie ma potrzeby aby wrapować funkcję w `IIFE` czy też wpisywać ręcznie `"use strict"` (domyślny mode w ES6). Dodatkowo należy pamiętac że jeśli coś nie zostanie jawnie dołączone do bundla przez `require` czy też `import` to się w nim nie znajdzie.

```javascript
// importowanie zależności do serwisu
import {UserModel} from '../models/user-model';
import Config from '../../common/config';

//eksportowanie serwisu na zewnątrz
export class UserService {

    constructor (private $http: angular.IHttpService) {
        'ngInject';
    }

    public findUsers(query: string): ng.IPromise<Array<User>> {
        return this.$http.get(`${Config.API_URL}/users?query=${query}`)
            .then((response) => response.data);
    }
}
```

- **Buduj aplikację jako drzewo komponentów**

Aplikacja powinna mieć strukturę identyczną z założeniem `React` czy też `AngularJS 2`

![component-tree](http://www.matrixtsl.com/wiki/images/e/e3/Eg_Tree_Structure.jpg)

Główny rodzic `app-component` powinien zawierać w swoim widoku `ui-view`. Następnie `ui-router` na podstawie dopasowanej ścieżki załaduje do niego jakiś `component`. Komponent może być liściem - nie zawierać dzieci (innych komponentów) lub być rodzicem dla innych komponentów. We wszystkich przypadkach tworzymy strukturę drzewiastą.

- **index.js**

Z punktu widzenia `node.js` i `webpack` `index.js` jest domyślnym plikiem każdego katalogu. Oznacza to, że jeśli nasz import będzie wyglądał następująco:

```javascript
import {AboutPage} from './pages/about';
```
 To Webpack spodziwa się pliku `index.js` w katalogu about. Jeśli chcemy przełamać tą konwencję, wystarczy że sprecyzujemy nasz `import`:
 
 ```javascript
 import {AboutPage} from './pages/about/file_name';
 ```

Czym jest `index.js` i co powinien w sobie zawierać?

![catalogue-structure](https://github.com/jpudysz/ng1-styleguide/blob/master/catalogue.PNG)

Plik `index.js` w roocie katalogu `components` powinien `importować` wszystkie komponenty i rejestrować je w angularowym module. Powinien również eksportować go wyżej, tak aby zaimportował go główny moduł aplikacji.

Plik `index.js` w dowolnym komponencie powinien rejestrować ten komponent w danym angularowym module i eksportować go wyżej.

Biorąc za przykład `person-list` i pamiętając, że każdy plik musi zostać zaimportowany aby został dołączony do builda musimy wypracować taką strukturę imprtów/eksportów aby:

1. komponent został zarejestrowany w specjalnie stworzonym dla niego module
2. template HTML został dołączony do tego komponentu
3. style SCSS zostały dołączone do strony
4. ścieżka została zarejestrowana w ui-router
5. komponent został zarejestrowany w module wyżej odpowiedzialnym za zebranie wszystkich komponentów i rejestracje w głównym komponencie aplikacji

Testy nie muszą zostać dołączane do builda, ponieważ są wyszukiwane po regexie.

Przykład implementacji plików `index.js` znajduje się w przykładowej aplikacji dołączonej do styleguida.

- **Pojedyńczy plik index.js powinien rejestrować całą aplikację**

Uzupełniając powyższy punkt, gdzieś w głównym katalogu aplikacji powinien być plik `index.js`, który zaimportuje wszystkie zależności z `pages` i `components` i zarejestruje jest w angularowym module głównym `app`. W pliku tym powinniśmy umieśćić również logikę związaną z providerami i interceptorami .

- **Konwencje nazewnictwa plików**

Pliki powinny być nazywane zgodnie z konwencją na powyższym obrazku tj.:

1. stosujemy `kebab-case` zamiast `camelCase`
2. każdy folder powinien zawierać `index.js`
3. prefix każdego pliku to nazwa komponentu
4. w wypadku powtarzających się rozszerzeń pliku np. `*.js` powinniśmy doprecyzować co dany plik zawiera przez dodanie dodatkowego członu po kropce np. `component` , `e2e` , `route` , `spec`

- **Rozważ użycie folderu shared**

Folder `shared` to folder, który możemy umieścić w dowolnie innym folderze, a który gromadzi re-używalne pliki. Weźmy za przykad  stronę `admin` a w niej katalog `shared`. Folder ten może zawierać m.in dzielone serwisy, modele, interfejsy `TypeScript`, mocki itd.

- **Zrezygnuj a anonimowych funkcji i zahardcodowanych nazw modułów**

Korzystając z `webpacka` i `Webstorma` otrzymujemy świetną okazję na budowę apliakcji w stylu Java czy też .NET. Kilkając na daną referencję komponentu, modelu, serwisu czy też modułu możemy przejść bezpośrednio do jego definicji. Każdy plik staje się ze sobą powiązany dzięki czemu cały projekt składa się w logiczną całośc. Dlatego unikajmy zahardcodowanych nazw angularowych modułów czy też anonimowych funkcji.

Rejestracja modułu:
```javascript
//example.component.js
class ExampleController {
}

const ExampleComponent = {
    controller: ExampleController,
    template: require('./example.html')
}

export default ExampleComponent;
```

```javascript
//index.js w folderze example
import ExampleComponent from './example.component';

const exampleModule = angular
    .module('app.components.example', [])
    .component('exampleComponent', ExampleComponent) //nazwana funkcja (const)
    .name; //wyeksportuj nazwę modułu = 'app.components.example'
    
export default exampleModule;
```

```javascript
//index.js w roocie katalogu components
import exampleModule from './example'; //importuje index.js

const componentsModule = angular
	.module('app.components', [
	    exampleModule //rejestruje 'app.components.example'!
	])
	.name;

export default componentsModule;
```

```javascript
//główny index.js
import componentsModule from './app/components';

angular.module('app', [
   componentsModule
])
.config(() => {
   //konfiguracja aplikacji
})
.run(() => {
  //konfiguracja aplikacji
});
```

- **Używaj vendor.js dla bibliotek i modułów zewnętrznych**

Każda biblioteka/angularowy moduł powinien zostać dołączony do aplikacji przez jeden plik `vendor.js`. Następnie plik ten nalezy umieścić  w konfiguracji `webpack` jako osobne `entry` dla aplikacji:

```javascript
//vendor.js
require('jquery');

import 'angular'
import 'angular-translate';
import 'angular-ui-router';

require('selectize/dist/js/standalone/selectize.js)'
```

```javascript
//webpack.config.js
...
entry: {
   app: './app', //główny angularowy moduł aplikacji
   vendor: './vendor.js' // wszystkie biblioteki
}
...
```

- **Tworzenie dyrektyw**

Dyrektywy w `TypeScript` tworzymy przez `factory` (nie mylić z `factory` w `AngularJS`):
```javascript
export class ExampleDirective {
   public restrict: 'A';
   public scope: {
      prop1: '=',
      prop2: '<'
   };
   
   public static Factory(
   	exampleDirectiveDependency: ExampleDependency
   ): ng.IDirective {
       'ngInject';
       
       return new ExampleDirective(exampleDirectiveDependency);
   }
   
   constructor(
       private exampleDirectiveDependency: ExampleDependency
   ) {
      this.link = this.link.bind(this);
   }
   
   public link() {
      //link function
   }
```

```javascript
//rejestracja
.directive('exampleDirective', ExampleDirective.Factory);
```

- **Unikaj restrict:'E' dla dyrektyw**

Zgodnie z punktem `Spójrz inaczej na dyrektywy` musimy rozróżniać dyrektywy i komponenty. Dlatego zabrionym jest aby tworzyć dyrektywy które posiadają `restrict: 'E'`. Coś co jest elementem i ma swój template HTML jest komponentem.

- **Rozważ użycie dekoratora @Component**

Gdy korzystamy z `TypeScript` warto rozważyć podejście przedstawione w sekcji `5.5 Decorators`. Standardową definicję komponentu, możemy zamienić na klasę i udekorować ją przez `@Component` jak w `AngularJS 2`:

```javascript
//standardowe podejście
class ExampleController {
}

const ExampleComponent = {
    controller: ExampleController,
    template: require('./example.html')
}

export default ExampleComponent;
```

```javascript
//TypeScript + dekorator @Component
@Component({
  template: require('./example.html')
})
class ExampleComponent {
  //controller
}
```

- **Avoid wildcard imports**

Unikaj importowania wszystkiego z modułu, dołączaj tylko to, co jest Ci potrzebne:
```javascript
//źle
import * as Helpers from '../utils/helpers';
```

```javascript
//ok
import {ParseCurrency, ComputeTax} from '../utils/helpers';
```

Unikaj również powtarzających się importów:
```javascript
//źle
import {A, B} from './c';
import {D} from './e';
import {F} from './c'
```

```javascript
//ok
import {A, B, F} from './c';
import {D} from './e';
```

- **Performance tricks**

Zachęcam do zapoznania się z trickami dotyczącymi performance AngularJS 1 przedstawionymi przez
[Todd Motto](https://www.youtube.com/watch?v=LoIuokh6NUI&list=WL&index=6).

# 7. Starter
Wkrótce (w trakcie tworzenia)

# 8. Przykładowa aplikacja
Wkrótce (w trakcie tworzenia na podstawie startera)

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
- [Kurs ES6 na Egghead.io](https://egghead.io/courses/learn-es6-ecmascript-2015?utm_source=drip&utm_medium=email&utm_campaign=learn-es6)
 
W razie znalezienia jakichkolwiek niejasności, literówek i (mam nadzieję że nie) błędów ortograficznych proszę o otwarcie PR. Jeżeli ktoś chciałby napisać jakiś dodatkowy rozdział/ zmienić istniejący/ napisać lepsze przykłady to bardzo do tego zachęcam i jestem otwarty na wszelkie propozycje.
