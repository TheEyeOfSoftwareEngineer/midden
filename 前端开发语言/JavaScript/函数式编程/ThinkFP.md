## Thinking Functional Programming

### What is Functional Programming?
The goal of FP is to *abstract control flows and operations on data* with functions in order to *avoid side effects and reduce mutations of state* in the application.

- 第一段代码
```javascript
document.querySelector('#msg').innerHTML = 'Hello World!';
```
- 第二段代码
```javascript
function printMessage(elementId, format, message) {
  document.querySelector(`#${elementID}`).innerHTML = `<${format}> ${message} </${format}>`;
}
printMessage('msg', 'h1', 'Hello World!');
```
对比第一段代码，有一些提升，但是仍然不是完全可重用的代码。比如现在需要是写一个文件而不是一个网页。函数式编程的目标是评估和组合很多函数从而达到更好的行为。
- 第三段代码
```javascript
var printMessage = run(addToDom('msg'), 'h1', echo);
printMessage('Hello World!');
```
我们将使用一个魔术方法，run，去依次触发一系列的函数，比如addToDom，echo。
- 第四段代码
```javascript
var printMessage = run(consol.log, repeat(3), h2, echo);
printMessage('Get Functional!');
```
这个例子中，我们将使用一个函数，consol.log，将输出到控制台，然后将输出重复3次，然后将输出到h2，然后将输出到控制台。

### 函数式编程的基础概念
#### Declarative programming
函数式编程属于声明式编程（表明一些列的操作，不需要表示这些操作是怎么实现的和数据怎么处理的）。
- 命令式编程
```javascript
var array = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
for(let i = 0; i < array.length; i++) {
  array[i] = Math.pow(array[i], 2);
}
array; // -> [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```
命令式事无巨细的告诉电脑怎么去执行一个特殊的任务。Declarative programming, on the other hand, separates program description from evaluation. It focuses on the use of expressions to describe what the logic of a program is without necessarily specifying its control flow or state changes. 
- 声明式编程
声明式只需要去关心把正确的行为用在每个元素上，把循环控制权交给系统的其他部分。
```javascript
[1, 2, 3, 4, 5, 6, 7, 8, 9].map(function(num) {
  return Math.pow(num, 2);
})
// -> [1, 4, 9, 16, 25, 36, 49, 64, 81]
```
Compared with the previous example, you see that this code frees you from the respon- sibility of properly managing a loop counter and array index access; put simply, the more code you have, the more places there are for bugs to occur. Also, standard loops aren’t reusable artifacts unless they’re abstracted with functions.
```javascript
[1, 2, 3, 4, 5, 6, 7, 8, 9].map(num => Math.pow(num, 2));
// -> [1, 4, 9, 16, 25, 36, 49, 64, 81]
```
为什么移除了循环？循环是难以重用和难以插入到其他操作的命令式的控制结构。除此之外，它也表明了代码在持续的在新的iteration中持续的改变或者修改。函数式编程追求stateless和immutability。无状态代码不会去改变或者破坏全局状态。为了使用避免副作用和状态的改变，我们需要纯函数。

#### Pure functions
函数式编程基于一个前提：我们将创建一个由纯函数构建的不可变代码。纯函数有以下特性：
- 函数仅依赖于提供的输入，而不是任何隐藏的或者外部的可能在调用或者运算中会变化的状态
- 函数不改变非自身作用域的状态，比如修改全局状态或者引用传值的参数
```javascript
var counter = 0;
function increment() {
  return ++counter;
}
```
这个函数会改变外部的状态，所以不能被称为纯函数。另外，一个常见的副作用发生在通过this关键字访问实例数据的时候。javascript中的this和其他语言不同，因为it determines the runtime context of a function.
这经常会导致代码难以推测。副作用可以出现在以下场景：
- 全局修改和一个变量，属性和数据结构
- 改变函数参数的原始值
- 处理用户输入
- 抛出异常，除非在同一个函数中捕获
- 在屏幕上显示或者打印
- 查询HTML元素，浏览器cookies或者数据库
纯函数很能在充满了动态行为和修改的场景中，但是有效的函数式编程不会限制所有的状态改变。它只是提供了一个框架去帮助管理和减少这些改变，并且将纯函数和非纯函数分离开。不纯的代码会产生外部可见的副作用。
```javascript
function showStudent(ssn) {
  var student = db.get(ssn);
  if(student !== null) {
    document.querySelector(`#{elementId}`).innerHTML = 
      `${studeent.ssn},
       ${student.firstname},
       ${student.lastname}`;
  }
  else {
    throw new Error('Stundet not found');
  }
}
showStudent('123-45-6789');
```
- 与外部的变量db交互，且没有在函数签名中声明。在任意时间点，这个变量可能是null或者在下次调用的时候改变，产生完全不同的结果并连累程序的完整性
- 全局变量elementId可以在任何时间改变，且不在控制范围内
- HTML元素是被直接修改了，DOM本身是一个可变的，共享的，全局资源
- 函数可能会抛出异常，可能导致整个程序卡顿和突然停止
这些使得代码不灵活，难以推测，并且难以测试。纯函数有着明确的以函数参数的约束，清晰的描述所有的函数的正式参数（一系列的输入），使得它们更好理解和使用。
我们现在可以做下面两件事情：
- 把长函数分离成更短的函数，每个函数只是单一目标
- 通过显式的定义函数所需要的参数去减少副作用的数量
和外部db记dom的交互是无法避免的，但是我们可以让它更好管理而且把它们和主逻辑分开。我们使用currying柯里化。柯里化可以部分设置一些函数参数从而避免它们到一个函数中。
```javascript
var find = curry(function(db, id) {
  var obj = db.get(id);
  if(obj === null) {
    throw new Error('Object not found');
  }
  return obj;
})

var csv = function(student) {
  return `${student.ssn},
          ${student.firstname},
          ${student.lastname}`;
};

var append = curry(function(elementId, info) {
  document.querySelector(`#{elementId}`).innerHTML = info;
})

var showStudent = run(append('#student-info'), csv, find(db));

showStudent('123-45-6789');
```
这样改写的好处：
- 更灵活，因为现在有三个可重用的组件
- 提高了效率
- 提高了代码的可阅读行
- 和HTML的交互移到了特定的函数中，实现了纯函数和非纯函数的隔离。

#### Referential transparency
Referential transparency是一种更加正式的定义纯函数的方式。纯函数的纯在这里表示的一个函数的参数和函数返回值的映射。一个函数同一个输入产生同样的输出，这就是
Referential transparency。
```javascript
var counter = 0;
function increment() {
  return ++counter;
}
```
有状态的increment函数不是
Referential transparency，因为它的返回值严重依赖外部的参数counter。为了让这个函数
Referential transparency，要移除状态依赖并且让这个参数变成一个正式的显式函数参数。
```javascript
var increment = counter => counter + 1;
```
We seek this quality in functions because it not only makes code easier to test, but also allows us to reason about entire programs much more easily. Referential transparency or equational correctness is inherited from math, but functions in programming lan- guages behave nothing like mathematical functions; so achieving referential transparency is strictly on us. 
`Program = [Input] + [func1, func2, func3, ...] => Output`

```javascript
// imperative
increment();
increment();
print(counter); // -> ?

// functional
var plus2 = run(increment, increment);
print(run(0)); // -> 2
```
如果函数是纯函数，我们可以容易地重写函数将函数替换成函数产生的值。
```javascript
var input = [80, 90, 100];
var average = (arr) => divide(sum(arr), size(arr));
average(input); // -> 90 
```
这个函数的输入是一个数组，它的输出是一个数字，它的输入和输出都是纯函数。所以我们可以改写为
```javascript
var average = divide(290, 3); // -> 90
```
Referential transparency makes it possible to reason about programs in this systematic, almost mathematical, way. The entire program can be implemented as follows:
```javascript
var sum = (total,current) => total + current;
var total = arr => arr.reduce(sum);
var size = arr => arr.length;
var divide = (a, b) => a / b;
var average = divide(total(input), size(input));
avaverage(input); // -> 90
```

#### Immutability
不可变的数据是在创建后就不可改变的。primitive data types are immutable.但是对于对象，可以改变。虽然他们可以作为输入传入到函数中，但是可以通过改变原始内容造成副作用。
```javascript
var sortDesc = function(arr) {
  return arr.sort(function(a, b) {
    return b - a;
  });
}
```
It does what you’d expect it to do—you provide an array, and it returns the same array sorted in descend- ing order:
```javascript
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
sortDesc(arr); // -> [9, 8, 7, 6, 5, 4, 3, 2, 1]
```
但是Array.sort是有状态的并且原地排序数组会造成副作用，原始的引用改变了。
functional programming refers to the declarative evaluation of pure functions to create immutable programs by avoiding externally observable side effects。

Most of the issues JavaScript developers face nowadays are due to the heavy use of large functions that rely greatly on externally shared variables, do lots of branch- ing, and have no clear structure. Unfortunately, this is the situation for many Java- Script applications today—even successful ones made up of many files that execute together, forming a shared mesh of mutable, global data that can be hard to track and debug.

Being forced to think in terms of pure operations and looking at functions as sealed units of work that never mutate data can definitely reduce the potential for bugs.


### 函数式编程的好处

#### encouraging the decmposition of complex tasks
At a high level, functional programming is effectively the interplay between decompo- sition (breaking programs into small pieces) and composition (joining the pieces back together). Modularization in FP is closely related to the singularity principle, which states that functions should have a single purpose. Purity and referential transparency encourage you to think this way because in order to glue simple functions together, they must agree on the types of inputs and outputs. From referential transparency, you learn that a function’s complexity is sometimes directly related to the number of arguments it receives (this is merely a practical observation and not a formal concept indicating that the lower the number of function parameters, the simpler the function tends to be).

In reality, run is an alias for one the most impor- tant techniques: composition. The composition of two functions is another function that results from taking the output of one and plugging it into the next. Assume that you have two functions f and g. Formally, this can be expressed as follows: `f·g = f(g(x))`. The formula reads "f composed of g" which creates a loose, type-safe relationship *between g’s return value and f’s argument*.
```javascript
var showStudent = compose(append('#student-info'), csv, find(db));
showStudent('123-45-6789');
```
Understanding compose is crucial for learning how to implementing modularity and reusability in functional applications. Functional composition leads to code in which the meaning of the entire expression can be understood from the meaning of its individual pieces—a quality that becomes hard to achieve in other paradigms.

函数式编程可以提高抽象的程度，不需要暴露细节。因为Compose接收其他函数作为参数，所以被认为是高阶函数。But composition isn’t the only way to create fluent, modular code; in this book, you’ll also learn how to build sequences of operations by connect- ing operations in a chain-like manner.

#### processing data using fluent chains
A chain is a sequential invocation of functions that share a common object return value (such as the $ or jQuery object). Like composition, this idiom allows you to write terse and concise code, and it’s typically used a lot in functional as well as reac- tive programming JavaScript libraries.
```javascript
let enrollment = [
  {enrolled: 2, grade: 100},
  {enrolled: 2, grade: 80},
  {enrolled: 1, grade: 89},
]
```
命令式编程过程大概是：
```javascript
var totalGrades = 0;
var totalStudentsFound = 0;
for(let i = 0; i < enrollment.length; ++i) {
  let student = enrollment[i];
  if(student !== null) {
    if(student.entolled > 1) {
      totalGrades += student.grade;
      totalStudentsFound += 1;
    }
  }
}
var average = totalGrades / totalStudentsFound; // -> 90
```
Just as before, decomposing this problem with a functional mindset, you can identify three major steps:
- Selecting the proper set of students (whose enrollment is greater than one)
- Extracting their grades
- Calculating their average grade
A function chain is a lazy evaluated program, which means it defers its execution until needed. This benefits performance because you can avoid executing entire sequences of code that won’t be used anywhere else, saving precious CPU cycles. This effectively simulates the call-by-need behavior built into other func- tional languages.
```javascript
_.chain(enrollment)
 .filter(student => student.enrolled > 1)
 .pluck('grade')
 .average()
 .value(); // -> 90
```

#### Reacting to the complexity of asynchronous applications
reactive prradigm的好处是it raises the level of abstraction of your code, allowing you to focus on specific business logic while forget- ting about the arduous boilerplate code associated with setting up asynchronous and event-based programs. Also, this emerging paradigm takes full advantage of FP’s abil- ity to chain or compose functions together.

```javascript
var valid = false;
var elem = document.querySelector('#student-ssn');
elem.onkeyup = function(event) {
  var val = elem.value;
  if(val !== null && val.length !== 0) {
    val = val.replace(/^\s*|\s*$|\-s/g, '');
    if(val.length === 9) {
      console.log(`Valid SSN: ${val}`);
      valid = true;
    }
  } else {
    consol.log('Invalid SSN: ${val}!');
  }
}
```
Because reactive programming is based on functional programming, it benefits from the use of pure functions to process data with the same familiar operations like map and reduce and the terseness of lambda expressions. So learning functional is half the battle when learning reactive! This paradigm is enabled through a very important artifact called an observable. Observables let you subscribe to a stream of data that you can process by composing and chaining operations together elegantly. 
> 这个范式通过一个特殊的物品——Observable，可以让你订阅一个数据流，然后通过编写和链接操作来完成你的业务逻辑。

```javascript
Rx.Observable.fromEvent(document.querySelector('#student-ssn'), 'keyup')
 .map(input => input.srcElement.value)
 .filter(ssn => ssn.length !== 0 && ssn !== null)
 .map(ssn => ssn.replace(/^\s*|\s*$|\-s/g, ''))
 .skipWhile(ssn => ssn.length !== 9)
 .subscribe(validSsn => console.log(`Valid SSN ${validSsn}`));
```
One of the most important takeaways is that all the operations performed are completely immutable, and all the business logic is segregated into individual functions.

