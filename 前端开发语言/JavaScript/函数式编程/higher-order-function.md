## higher-order-function

### Functional vs. object-oriented programming
Object-oriented applications, which are mostly imperative, rely heavily on object- based encapsulation to protect the integrity of their mutable state, both direct and inherited, in order to expose or manipulate that state via instance methods. Alternatively, functional programming removes the need to hide data from the callers and typically works with a smaller set of very simple data types.
```javascript
get fullname() {
  return [this._firstname, this._lastname].join(' ');
}
```
can be split out as follows:
```javascript
var fullname = 
  person => [person._firstname, person._lastname].join(' ');
```
- Person
```javascript
class Person {
  constuctor(firstname, lastname, ssn) {
    this._firstname = firstname;
    this._lastname = lastname;
    this._ssn = ssn;
    this._address = null;
    this._birthYear = null;
  }
  get ssn() {
    return this._ssn;
  }

  get firstname() {
    return this._firstname;
  }

  get lastname() {
    return this._lastname;
  }

  get birthYear() {
    return this._birthYear;
  }

  set birthYear(year) {
    this._birthYear = year;
  }

  set address(address) {
    this._address = address;
  }

  toString() {
    return `Person(${this._firstname} ${this._lastname})`;
  }
}

class Student extends Person {
  constructor(firstname, lastname, ssn, school) {
    super(firstname, lastname, ssn);
    this._school = school;
  }

  get school() {
    return this._school;
  }
}
```
The object-oriented solution tightly couples operations, via this and super, to the object and parent object, respectively:
```javascript
// Person class
peopleInSameCountry(friends) {
  var result = [];
  for(let idx in friends) {
    var friend = friends[idx];
    if(this.address.country === friend.address.country) {
      result.push(friend);
    }
    return result;
  }
};
// Student class
studentInSameCountryAndSchool(friends) {
  var closeFriends = super.peopleInSameCountry(friends);
  var result = [];
  for(let idx in closeFriends) {
    var friend = closeFriends[idx];
    if(this.school === friend.school) {
      result.push(friend);
    }
    return result;
  }
};
```
```javascript
var curry = new Student('Haskell', 'Curry', '111-11-1111', 'Penn State');
curry.address = new Address('US');
var turing = new Student('Alan', 'Turing', '222-22-2222', 'Princeton');
turing.address = new Address('England');
var church = new Student('Alonzo', 'Church', '333-33-3333', 'Princeton');
church.address = new Address('US');
var kleene = new Student('Stephen', 'Kleene', '444-44-4444', 'Princeton');
kleene.address = new Address('US');
```
```javascript
church.studentsInSameCountryAndSchool([curry, turing, kleene]);
//-> [kleene]
```
The functional solution, on the other hand, breaks the problem into smaller functions:
```javascript
// Creates a selector function that knows how to compare students’ country and school
function selector(country, school) {
  return function(student) {
    // Navigates the object graphs.
    return student.address.country() === country && student.school() === school;
  }; 
}

// Uses the filter operation on arrays and injects the special behavior via a selector function
var findStudentsBy = function(friends, selector) {
  return friends.filter(selector);
};

findStudentsBy([curry, turing, church, kleene], selector('US', 'Princeton'));
//-> [church, kleene]
```
| | Functional| OOP|
|-|-|-|-|
|Unit of composition| Functions| Objects|
|Programming style| Declarative| Imperative|
|Data and behaviro| Loosely coupied into pure, standalone functions| Tightly coupled in classes with methods|
|State management| Treats objects as immutable| Mutates objects|
|control flow| Functions and recursion| Loops and conditionals|
|Thread safety| Enables concurrent programming| Difficult to achieve|
|Encapsulation| Not needed because everything is immutatble| Needed to protect data integrity|

JavaScript is as object-oriented as it is functional, using it functionally will require some special attention in terms of controlling state changes.

#### managing the state of JavaScript Objects
The state of a program can be defined as a snapshot of the data stored in all of its objects at any moment in time. 

#### Treating objects as values
Strings and numbers are probably the easiest data types to work with in any program- ming language. Part of the reason is that, traditionally, these primitive types are inherently immutable, which gives us a certain peace of mind that other user-defined types don’t. In functional programming, we call types that behave this way *values*. 

Despite all the syntactic sugar added around classes in ES6, JavaScript objects are nothing more than bags of attributes that can be added, removed, and changed at any time.

ES6 uses the const keyword to create constant references. This moves the needle in the right direction because constants can’t be reassigned or re-declared. In practi- cal functional programming, you can use const as a means to bring simple configura- tion data (URL strings, database names, and so on) into your functional program if need be. Although reading from an external variable is a side effect, the platform pro- vides special semantics to constants so they won’t change unexpectedly between func- tion calls.
```javascript
const gravity_ms = 9.806;
gravity_ms = 20;
```
But this doesn’t solve the problems of mutability to the level that FP requires. You can prevent a variable from being reassigned, but how can you prevent an object’s internal state from changing? 
```javascript
const student = new Student('Alonzo', 'Church', '666-66-6666', 'Princeton');
student.lastname = 'Mouring';
```
For simple object structures, a good alternative is to adopt the Value Object pattern. A value object is one whose equality doesn’t depend on identity or reference, just on its value; once declared, its state may not change. In addition to numbers and strings, some examples of value objects are types like tuple, pair, point, zipCode, coordinate, money, date, and others.
```javascript
function zipCode(code, location) {
  let _code = code;
  let _location = location;
  return {
    code: function() {
      return _code;
    },
    location: function() {
      return _location;
    },
    fromString: function(str) {
      let parts = str.split('-');      
      return zipCode(parts[0], parts[1]);
    },
    toString: function() {
      return _code + '-' + _location;
    }
  };
}
const princetonZip = zipCode('08544', '3345');
printtonZip.toString(); //-> '08544-3345'
```
you can use functions and guard access to a ZIP code’s internal state by returning an object literal interface that exposes a small set of methods to the caller and treats _code and _location as pseudo-private variables. These variables are only acces- sible in the object literal via closures. The returned object effectively behaves like a primitive that has no mutating methods. Hence, the toString method, although not a pure function, behaves like one and is a pure string representation of this object. Value objects are lightweight and easy to work with in both functional and OOP. In conjunction with const, you can cre- ate objects with semantics similar to those of a string or number.
```javascript
function coordinate(lat, long) {
  let _lat = lat;
  let _long = long;
  return {
    latitude: function() {
      return _lat;
    },
    longitude: function() {
      return _long;
    },
    translate: function(dx, dy) {
      return coordinate(_lat + dx, _long + dy);
    },
    toString: function() {
      return '(' + _lat + ',' + _long + ')';
    }
  };
}
const greenwich = coordinate(51.4778, 0.0015);
greenwish.toString(); //-> '(51.4778,0.0015)'
```
Using methods to return new copies (as in translate) is another way to implement immutability. Applying a translation operation on this object yields a new coordinate object:
```javascript
greenwich.translate(10, 10).toString(); //-> '(61.4778, 10.0015)'
```
Value Object is an object-oriented design pattern that was inspired by functional pro- gramming. This is another example of how the paradigms elegantly complement each other. This pattern is ideal, but it’s not enough for modeling entire real-world prob- lem domains. In practice, it’s likely your code will need to handle hierarchical data (as you saw with Person and Student earlier) as well as interact with legacy objects. Luck- ily, JavaScript has a mechanism to emulate this using with Object.freeze.

#### Deep-freezing moving parts
JavaScript’s new class syntax doesn’t define keywords to mark fields as immutable, but it does support an internal mechanism for doing so by controlling some hidden object metaproperties like writable. By setting this property to false, JavaScript’s `Object.freeze()` function can prevent an object’s state from changing.
```javascript
var person = Object.freeze(new Person('Haskell', 'Curry', '111-11-11111'));
person.firstname = 'Barkley'; // Not allowed
```
Executing the preceding code makes the attributes of person effectively read-only.
Any attempt to change them (_firstname, in this case) will result in an error:
```javascript
TypeError: Cannot assign to read only property '_firstname' of #<Person>
```
Object.freeze() can also immobilize inherited attributes. So freezing an instance of Student works exactly the same way and follows the object’s prototype chain protecting every inherited Person attribute. But it can’t be used to freeze nested object attributes.
> Object.freeze不能冻结类里面的对象参数
```javascript
class Address {
  constructor(country, state, city, zip, street) {
    this._country = country;
    this._state = state;
    this._city = city;
    this._zip = zip;
    this._street = street;
  }

  get street() {
    return this._street;
  }

  get city() {
    return this._city;
  }

  get state() {
    return this._state;
  }

  get zip() {
    return this._zip;
  }

  get country() {
    return this._country;
  }
}
```
Unfortunately, no errors will occur in the following code:
```javascript
var person = new Person('Haskell', 'Curry', '444-44-4444');
person.address = new Address(
    'US', 'NJ', 'Princeton',
     zipCode('08544','1234'), 
     'Alexander St.');
person = Object.freeze(person);
person.address._country = 'France'; //-> allowed!
person.address.country; //-> 'France'
```
Object.freeze() is a shallow operation. To get around this, you need to manually freeze an object’s nested structure.
```javascript
var isObject = (val) => val && typeof val === 'object';

function deepFreeze(obj) {
  if(isObject(obj)
    && !Object.isFrozen(obj)) {
      Object.keys(obj)
        .forEach(name => deepFreeze(obj[name]));
      Object.freeze(obj);
  }
  return obj;
}
```
I’ve just shown some techniques you can use to enforce a level of immutability in your code, but it’s unrealistic to expect that you can create entire applications without ever modifying any state. Thus, strict policies when creating new objects from originals (as with coordinate.translate()) are extremely beneficial in your quest to reduce the complexities and intricacies of JavaScript applications.

#### Navigating and modifying object graphs with lenses
In OOP, you’re accustomed to calling methods that change the internal contents of a stateful object. This has the disadvantage of never being able to guarantee the outcome of retrieving the state and may break the functionality of part of the system that expects the object to stay intact. You could opt to implement your own copy-on-write strategy and return new objects from each method call—a tedious and error-prone process, to say the least.
```javascript
set lastname(lastname) {
  return new Person(this._firstname, lastname, this._ssn);
}
```
You need a solution for mutating stateful objects, in an immutable manner, that’s unobtrusive and doesn’t require hardcoding boilerplate code everywhere. Lenses, also known as functional references, are functional programming’s solution to accessing and immutably manipulating attributes of stateful data types. Internally, lenses work similarly to a copy-on-write strategy by using an internal storage component that knows how to properly manage and copy state. By default, Ramda exposes all of its functionality via the global object R. Using R.lensProp, you can create a lens that wraps over the lastname property of Person:
```javascript
var person = new Person('Haskell', 'Curry', '444-44-4444');
var lastnameLens = R.lensProp('lastName');
```
You can use R.view to read the contents of this property:
```javascript
R.view(lastnameLens, person); //-> 'Curry'
```
This is, for all practical purposes, similar to a get lastname() method. Nothing impressive so far. What about the setter? Here’s where the magic comes in. Now, calling R.set creates and returns a brand-new copy of the object containing the new value and preserves the original instance state (copy-on-write semantics for free!):
```javascript
var newPerson = R.set(lastnameLens, 'Mourning', person);
newPerson.lastname; //-> 'Mourning'
person.lastname; //-> 'Curry'
```
Lenses are valuable because they give you an unobtrusive mechanism for manipulat- ing objects, even if these are legacy objects or objects outside of your control. Lenses also support nested properties, like the address property of Person:
```javascript
person.address = new Address(
  'US', 'NJ', 'Princeton',
  zipCode('08544','1234'),
  'Alexander St.');
)
```
Let’s create a lens that navigates to the address.zip property:
```javascript
var zipPath = ['address', 'zip'];
var zipLens = R.lens(R.path(zipPath), R.assocPath(zipPath));
R.view(zipLens, person); //-> zipCode('08544','1234')
```
Because lenses implement immutable setters, you can change the nested object and still return a new Person object:
```javascript
var newPerson = R.set(zipLens, person, zipCode('90210', '5678'));
R.view(zipLens, newPerson); //-> zipCode('90210', '5678')
R.view(zipLens, person); //-> zipCode('08544','1234')
newPerson !== person; //-> true
```
This is great because now you have getter and setter semantics in a functional way. In addition to providing a protective immutable wrapper, lenses also fit extremely well with FP’s philosophy of isolating field-access logic away from the object, eliminating the reliance on this, and giving you powerful functions that know how to reach into and manipulate the contents of any object.

### Function
In functional programming, functions are the basic units of work, which means every- thing centers around them. A function is any callable expression that can be evaluated by applying the () operator to it. Functions can return either a computed value or undefined (void function) back to the caller. Because FP works a lot like math, functions are meaningful only when they produce a usable result (not null or undefined); otherwise, the assumption is that they modify external data and cause side effects to occur.

JavaScript functions have two important characteristics that are the bread and but- ter of its functional style: they’re first-class and higher-order.

#### Functions as first-class citizens
In JavaScript, the term first-class comes from making functions actual objects in the language—also called first-class citizens. You’re probably used to seeing functions declared like this:
```javascript
function multiplier(a, b) {
  return a * b;
}
```
But JavaScript offers more options. Like objects, a function can be:
- Assigned to variables as an anonymous function or lambda expression
```javascript
var square = function(x) {
  return x * x;
};

var square = x => x * x;
```
- Assigned to object properties as methods:
```javascript
var obj = {
  square: function(x) {
    return x * x;
  }
};
```
Whereas a function call uses the () operator, as in square(2), the function object is printed as follows:
```javascript
square;
// function(x) {
//   return x * x;
// }
```
Although not common practice, functions can also be instantiated via constructors, which is proof of their first-class nature in JavaScript. The constructor takes the set of formal parameters, the function body, and the new keyword, like so:
```javascript
var multiplier = new Function('a', 'b', 'return a * b');
multiplier(3, 4); //-> 12
```
In JavaScript, every function is an instance of the Function type. A function’s length property can be used to retrieve the number of formal parameters, and methods such as apply() and call() can be used to call functions with contexts.

The right side of an anonymous function expression is a function object with an empty name property. You can use anonymous functions to extend or specialize a function’s behavior by passing them as arguments. Consider JavaScript’s native Array.sort(comparator) as an example; it takes a comparator function object. By default, *sort converts values being sorted into **strings** and uses their Unicode values* as natural sorting criteria. 
```javascript
var fruit = ['Banana', 'Orange', 'Apple', 'Mango'];
fruit.sort(); // ['Apple', 'Banana', 'Mango', 'Orange']
// Numbers are converted into strings and compared with their Unicode points.
var ages = [1, 10, 21, 2];
ages.sort(); // [1, 10, 2, 21];
```
As a result, sort() is a function whose behavior is frequently driven by the criteria implemented in the comparator function, which by itself is almost useless. You can force proper numerical comparisons and sort a list of people by age using a custom function argument:
```javascript
people.sort((p1, p2) => p1.getAge() - p2.getAge());
```
The comparator function takes two parameters, p1 and p2, with the following contract:
- If comparator return less than 0, p1 comes before p2.
- If comparator returns 0, leave p1 and p2 unchanged.
- If comparator returns greater than 0, p1 comes after p2.
In addition to being assignable, JavaScript functions like sort() accept other functions as arguments and belong to a category called higher-order functions.

#### Higher-order functions
Because functions behave like regular objects, you can intuitively expect that they can be passed in as function arguments and returned from other functions. These are called higher-order functions. 

The following snippet shows that functions can be passed in to other functions. The applyOperation function takes two arguments and applies any operator func- tion to both of them:
```javascript
function applyOperation(a, b, opt) {
  return opt(a, b);
}

var multiplier = (a, b) => a * b;

applyOperation(3, 4, multiplier); //-> 12
```
In the next example, the add function takes an argument and returns a function that, in turn, receives a second argument and adds them together:
```javascript
function add(a) {
  return function(b) {
    return a + b;
  }
}
add(3)(3); //-> 6
```
Because functions are first-class and higher-order, JavaScript functions can behave as values, which implies that *a function is nothing more than a yet-to-be-executed value defined immutably based on the input provided to the function*. This principle is embedded in everything that you do in functional programming, especially when you get into function chains.

You can combine higher-order functions to create meaningful expressions from smaller pieces and simplify many programs that would otherwise be tedious to write.
```javascript
function printPeopleInTheUs(people) {
  for(let i = 0; i < people.length; ++i) {
    var thisPerson = people[i];
    if(thisPerson.address.country === 'US') {
      console.log(thisPerson.name);
    }
  }
}
printPeopleInTheUs([p1, p2, p3]);
```
With higher-order functions, you can nicely abstract out the action performed on each person: in this case, printing to the console. You can freely supply any action function you want to a higher-order printPeople function:
```javascript
function printPeople(people, action) {
  for(let i = 0; i < people.length; ++i) {    
    action(people[i]);
  }
}

var action = function(person) {
  if(person.address.country === 'US') {
    console.log(person.name);
  }
}

printPeople(people, action);
```
A noticeable pattern that occurs in languages like JavaScript is that function names can be passive nouns like multiplier, comparator, and action. Because they’re first-class, functions can be assigned to variables and executed at a later time. Let’s refactor printPeople to take full advantage of higher-order functions:
```javascript
function printPeople(people, selector, printer) {
  people.forEach(function(person) {
    if(selector(person)) {
      printer(person);
    }
  });
}

var inUs = Person => person.address.country === 'US';
printPeople(people, inUs, console.log);
```
This is the mindset you must develop to fully embrace functional programming. This exercise shows that the code is a lot more flexible than what you started with, because you can quickly swap (or configure) the criteria for selection as well as change where you want to print.

```javascript
var countryPath = ['address', 'country'];
var countryL = R.lens(R.path(countryPath), R.assocPath(countryPath));
var inCountry = R.curry((country, person) =>
        R.equals(R.view(countryL, person), country));
people.filter(inCountry('US')).map(console.log);
```

#### Types of function invocation
JavaScript’s function-invocation mechanism is an interesting part of the language and different from other programming languages. JavaScript gives you complete freedom to dictate the runtime context in which a function is invoked: the value of this in the function body. JavaScript functions can be invoked in many different ways:
- As a global function —The reference to this is set either to the global object or to undefined (in strict mode):
```javascript
function doWork() {
  this.myVar = 'Some value';
}
doWork();
```
- As a method —The reference to this is set to the owner of the method. This is an important part of JavaScript’s object-oriented nature:
```javascript
var obj = {
  prop: 'Some property',
  getProp: function() {
    return this.prop;
  }
}
obj.getProp();
```
- As a constructor by prepending the call with new—This implicitly returns the refer- ence to the newly created object:
```javascript
function MyType(arg) {
  this.prop = arg;
}
var someVal = new MyType('Some value');
```
Unlike in other programming languages, the this reference is set based on how the function is used (globally, as an object method, as a constructor, and so son) and not by its lexical context (its location in the code). This can lead to code that’s hard to understand, because you need to pay close attention to the context in which a function is executing.

#### Function methods
JavaScript supports calling functions via the function methods (like meta-functions) call and apply, which belong to the function’s prototype. Both methods are used extensively when scaffolding code is built so that API users can create new functions from existing ones.
```javascript
function negate(func) {
  return function() {
    return !func.apply(null, arguments);
  }
}

function isNull(val) {
  return val === null;
}

var isNotNull = negate(isNull);
isNotNull(null); //-> false
isNotNull({}); //-> true
```
The negate function creates a new function that invokes its argument and then logi- cally negates it. This example uses apply, but you could use call the same way; the dif- ference is that the latter accepts an argument list, whereas the former takes an array of arguments. The first argument, thisArg, can be used to manipulate the function context as needed. Here are both signatures:
```javascript
Function.prototype.apply(thisArg, [argsArray]);
Function.prototype.call(thisArg, arg1, arg2, ...);
```
If thisArg refers to an object, it’s set to the object the method is called on. If thisArg is null, the function context is set to the global object, and the function behaves like a simple global function. But if the method is a function in strict mode, the actual value of null is passed in.

### Closures and scopes
Prior to JavaScript, closures only existed in FP languages used in certain specific appli- cations. JavaScript is the first to adopt it into mainstream development and signifi- cantly change the way in which we write code. 
```javascript
function zipCode(code, location) {
  let _code = code;
  let _location = location;
  return {
    code: function() {
      return _code;
    },
    location: function() {
      return _location;
    }
  };
}
```
the zipCode function returns an object literal that seems to have full access to variables declared outside of its scope. In other words, after zipCode has finished executing, the resulting object can still see information declared in this enclosing function:
```javascript
const princetonZip = zipCode('90210', '3345');
princetonZip.code(); //-> '90210'
```
This is a bit mind-bending, and it’s all thanks to the closure that forms around object and function declarations in JavaScript. Being able to access data this way has many practical uses;

A closure is a data structure that binds a function to its environment at the moment it’s declared. It’s based on the textual location of the function declaration; therefore, a closure is also called a static or lexical scope surrounding the function definition. Because it gives functions access to its surrounding state, it makes code clear and read- able. As you’ll see shortly, closures are instrumental not only in functional programs when you’re working with higher-order functions, but also for event-handling and callbacks, emulating private variables, and mitigating some of JavaScript’s pitfalls.

The rules that govern the behavior of a function’s closure are closely related to JavaScript’s scoping rules. A scope groups a set of variable bindings and defines a section of code in which a variable is defined. In essence, a closure is a function’s inheri- tance of scopes akin to how an object’s method has access to its inherited instance variables—both have references to their parents. Closures are readily seen in the case of nested functions.
```javascript
function makeAddFunction(amount) {
  function add(number) {
    return number + amount;
  }
  return add;
}

function makeExponentialFunction(base) {
  function raise(exponent) {
    return Math.pow(base, exponent);
  }
  return raise;
}
var addTenTo = makeAddFunction(10);
addTenTo(10); //-> 20

var raiseThreeTo = makeExponentialFunction(3);
raiseThreeTo(4); //-> 81
```
It’s important to notice in this example that even though the amount and base variables in both functions are no longer in the active scope, they’re still accessible from the returned function when invoked. Essentially, you can imagine the nested functions add and raise as functions that package not only their computation but also a snapshot of all variables surrounding them. a function’s clo- sure includes the following:
- All function parameters (params and params2, in this case)
- All variables in the outer scope (including all global variables, of course), as well as those declared after the function additionalVars
```javascript
var outerVar = 'Outer';
function makeInner(params) {
  var innerVar = 'Inner';
  function inner() {
    console.log(`I can see: ${outerVar}, ${innerVar}, and ${params}`);
  }
  return inner;
}
var inner = makeInner('Params');
inner(); // -> I can see: Outer, Inner, and Params
```

#### Problems with the global scope
Polluting the global namespace can be problematic because you run the chance of overriding variables and functions declared in different files. Global data has the detrimental effect of making programs hard to reason about because you’re obligated to keep a mental note of the state of all variables at any point in time. This is one of the main reasons program complexity increases as your code becomes larger. It’s also conducive to having side effects in your functions, because you inevitably create external dependencies when reading from or writing to it. It should be obvious at this point that when writing in an FP style, you’ll avoid using global variables at all cost.

#### JavaScript’s function scope
This is JavaScript’s preferred scoping mechanism. Any variables declared in a function are local to that function and not visible anywhere else. Also, when a function returns, any local variables declared in are deleted with it. So in the function
```javascript
function doWork() {
  let student = new Student(...);
  let address = new Address(...);
  // do more work
};
```
the variables student and address are bound in doWork() and are inaccessible by the outside world. resolving a variable by name is similar to the prototype name-resolution chain described earlier. It begins by checking the innermost scope and works its way outward. JavaScript’s scoping mechanism works as follows:
- It checks the variable’s function scope.
- If not in the local scope, it moves outward into the surrounding lexical scope, searching for the variable reference until it reaches the global scope.
- If the variable can’t be referenced, JavaScript returns undefined.
```javascript
var x = 'Some value';
function parentFunction() {
  function innerFunction() {
    console.log(x);
  }
  return innnerFunction;
}
var inner = parentFunction();
inner(); //-> 'Some value'
```

#### A pseudo-block scope
```javascript
function doWork() {
  if(!myVar) {
    var myVar = 10;
  }
  consol.log(myVar); // -> 10
}
doWork();
```
The variable myVar is declared in the if statement, but it’s visible from outside the block. Strangely enough, running this code prints out the value 10. This can be baf- fling, especially for developers used to the more common block-level scope. An inter- nal JavaScript mechanism hoists variable and function declarations to the top of the current scope—the function scope, in this case.
```javascript
var arr - [1, 2, 3, 4];
function processArr() {
  function multipleBy10(val) {
    i = 10;
    return val * i;
  }
  for(var i = 0; i < arr.length; ++i) {
    arr[i] = multipleBy10(arr[i]);
  }
  return arr;
}
processArr(); // -> [10, 2, 3, 4];
```
S6 JavaScript provides the let keyword to help resolve this loop-counter ambiguity by properly binding the loop counter to its enclosing block:
```javasript
for(let i = 0; i < arr.length; ++i) {
  // ...
}
i; // i === undefined
```

#### Practical applications of closures
Closures have many practical applications that are important to apply when imple- menting large JavaScript programs. These aren’t specific to functional programming, but they do take advantage of JavaScript’s function mechanism:
- Emulating private variables
Using closures, however, it’s possible to emulate this behavior. One example is returning an object, much like zipCode and coordinate in the earlier example. These functions return object literals with methods that have access to any of the outer func- tion’s local variables, but don’t expose these variables, therefore effectively making them private.

Closures can also provide a way to manage your global namespace to avoid globally shared data. Library and module authors take closures to the next level by hiding an entire module’s private methods and data. This is referred to as the Module pattern because it uses a single immediately invoked function expression (IIFE) to encapsulate internal variables while allowing you to export the necessary set of functionality to the outside world and severely reduce the number of global references.
```javascript
var MyModule = (function MyMjodule(export) {
  let _myPrivateVar = ...;
  export.method1 = function() {
    // do work
  };
  export.method3 = function() {
    // do work
  };
  }
}(MyModule || {}));
```
The object MyModule is created globally and passed into a function expression, created with the function keyword, and immediately executed when the script is loaded. Due to JavaScript’s function scope, _myPrivateVar and any other private variables are local to the wrapping function. The closure surrounding the two exported methods is what allows the object to safely access all of the module’s internal properties.
- Making asynchronous server-side calls
JavaScript’s first-class, higher-order functions can be passed into other functions as callbacks. Callbacks are useful as hooks to handle events in an unobtrusive manner.
Suppose you need to make a request to the server and want to be notified once the data has been received. The traditional idiom is to provide a callback function that will handle the response:
```javascript
getJSON('/student', (student) => {
  getJSON('/student/grades',
    grades => processGrades(grades),
    error => console.log(error.message));
  },
  (error) => console.log(error.message))
})
```
getJSON is a higher-order function that takes two callbacks as arguments: a success function and an error function. A common pattern that occurs with asynchronous code as well as event handling is that you can easily corner yourself into deeply nested function calls; this forms the unpleasant “callback pyramid of doom” when you need to make several subsequent remote calls to the server. As you’ve probably experi- enced, when code is deeply nested, it becomes hard to follow. 
- Creating artificial block-scoped variables
the underlying issue is JavaScript’s lack of block-scope semantics, so the objective is to artificially create this block scope. What can you do about this? Using let mitigates many of the issues with the traditional looping mechanism, but a functional approach would be to take advantage of closures and JavaScript’s function scope and consider using forEach. 
```javascript
arr.forEach(function(elem, i) {
  ...
});
```
