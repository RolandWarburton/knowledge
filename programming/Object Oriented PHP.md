# Object Oriented PHP

A collection of notes based loosely around the php 7 documentation.

for debugging its a good idea to prepend your php files with the following code to see any errors.

```php
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);
```

## Part 1 - The Basics & introduction

A class should be pascal case (upper camel case).

```php
class SimpleClass
{
	// property declaration
	public $var = 'a default value';

	// method declaration
	public function displayVar() {
		echo $this->var;
	}
}
```

A function without a public declaration is implied to be public.

```php
class SimpleClass
{
	// This is a public method even without "public".
	function printHello() {
		echo "Hello world";
	}
}
```

### Static methods

$this is a reference to the calling object. However if the object is called statically, from class B to class A then $this will be undefined, because $this refers to the object and "static" has no object that it can refer to.

```php
// Heres a working example
class Foo {
	public static function aStaticMethod() {
		echo "hello";
	}
}

// method 1
// call static method, it prints hello
Foo::aStaticMethod();

// method 2
// make a "class"
$classname = 'Foo';
// call a static method from the "class", it also prints hello
$classname::aStaticMethod();
```

Heres an example of what **WONT** work, if you wanted to use a static variable in a method, regardless of it was static as well it will not be accessible.

```php
// An example that WONT work
class Foo {
	public $a = "hello";
	public static $b = "world";
	public static function aStaticMethod() {
		echo $a . $b;
	}
}

// call the method, it wont print "hello world"
Foo::aStaticMethod();

// this also wont print "hello world" regardless that "world" is static
// it also wont with if $b was `static $b = "world"`
$classname = 'Foo';
$classname::aStaticMethod();
```

Here the `::` is used to access the method of the Foo class, normally you would use the `->` operator on a non static method.

### Creating a class

Create a class like you would expect with the `new` keyword.

```php
// you can also create a class using a variable
$className = 'SimpleClass';
$instance = new $className();
```

### Clone objects

If you copy an object, the copied object is a pointer to the original object.

```php
class SimpleClass
{
	// property declaration
	public $var = 'a default value';

	// method declaration
	public function setVar(string $val) {
		$this->var = $val;
		echo ("<br/>$this->var<br/>");
	}
}
// create an object
$instance = new SimpleClass();
// clone the object to $instance
// Read on after this code example about assign by ref (=&)
$assigned = $instance;

// change the value in the cloned object
$assigned->setVar("hello");

// These are now the same
var_dump($instance);
echo"<br/>";
var_dump($assigned);

// Now lets change $instance
$instance->setVar("sample");

// These are still the same (both are sample)
var_dump($instance);
echo"<br/>";
var_dump($assigned);
```

```output
hello
object(SimpleClass)#1 (1) { ["var"]=> string(5) "hello" }
object(SimpleClass)#1 (1) { ["var"]=> string(5) "hello" }
sample
object(SimpleClass)#1 (1) { ["var"]=> string(6) "sample" }
object(SimpleClass)#1 (1) { ["var"]=> string(6) "sample" }
```

This is essentially assign by reference instead of assign by value. Assign by val uses `=` and by ref uses `=&`. When two objects are assigned by reference then they are "tied" to each other. When the variable $assigned is to set **class** $instance it is implied that an assign by reference was used `$assigned =& $instance;`

### Assigning a method to a property

Straight from the php [docs](https://www.php.net/manual/en/language.oop5.basic.php#language.oop5.basic.properties-methods) this one is very straightforward. This seems to be a bit theoretical or superfluous but regardless.

```php
class Foo
{
	public $bar;
	public function __construct() {
		// set bar to this function
		$this->bar = function() {
			return 42;
		};
	}
}

// create an object
$obj = new Foo();

// store the function from $bar in $func
$func = $obj->bar;
// now $func is the function from the class Foo
echo $func(), PHP_EOL;

// alternatively, as of PHP 7.0.0 you can use an anon function by enclosing it in brackets
echo ($obj->bar)(), PHP_EOL;
```

### Extend classes

Another usual suspect.

```php
class Foo {
	public $name;

	public function __construct() {
		$this->name = "cat";
	}

	public function callSign() {
		echo "meows like a cat";
	}

}

class Bar extends Foo {
	public $name;

	public function __construct() {
		$this->name = "dog";
	}

	// this function overwrites the og one
	// You can also overwrite a constructor (like i did above)
	public function callSign() {
		echo "barks like a dog";
	}
}


// create an object
$b = new Bar();

echo $b->callSign()."<br>";

echo $b->name."<br>";
echo $b->bar."<br>";
```

```output
barks like a dog
dog
```

### Get the name of a class

You can use `::class` to obtain the name of the class. PHP cites this as being useful in a namespace. When you create the NS namespace, no code can be defined outside of it, then we can use the static `::` operator on the name of the class (ClassName) to get its class name within its namespace. IE. `NS/ClassName`.

```php
namespace NS {
	class ClassName {
		$hello = "world";
	}

	echo ClassName::class;
}
```

## Properties

A property is either `public, protected, or private`, as seen above assignment of t property is done through the `$this->variable` operator in which $this refers to the object. Static properties are accessed via the `::` operator. Declaring class properties or methods as static makes them accessible without needing an instantiation of the class.

### Heredocs and Nowdocs

Heredocs allow for pretty formatting of long multi line strings in double quotes (""). A heredoc is a `<<<` followed by a delimiter, similar to how you would define a motd on a cisco device (thats obscure but how i remember it). The result will be on one line.

```php
$str = <<<EOD
Example of string
spanning multiple lines
using heredoc syntax.
EOD;
```

```output
Example of string spanning multiple lines using heredoc syntax.
```

A Nowdoc is the same as a Heredoc except its using single quotes instead ('). A single quote will not substitute $variables, double quotes (") will.

### Typed declarations

Typed declarations can be used for properties of a class, a method of a class, or a function.

```php
class Foo {
	// ?int means it can be int OR null as a primitive.
	public int $myNumber;
	public ?int $myNumberOrNull;
}
```

### Constants

A const can be created within a class scope. Class constants are allocated once per class, and not for each class instance. By default a const is **public**, if a const is made private you cannot access it outside the class using `myClass::CONSTANT`

```php
class MyClass
{
	// public can be dropped as its inferred
	public const CONSTANT = 'constant value';

	function showConstant() {
		echo  self::CONSTANT . "\n";
	}
}

echo MyClass::CONSTANT . "\n";
```

## Autoloading Classes

If you want to load a bunch of classes you can use the `spl_autoload_register()` function.

From the docs...

> This example attempts to load the classes MyClass1 and MyClass2 from the files MyClass1.php and MyClass2.php respectively.

```php
spl_autoload_register(function ($class_name) {
	include $class_name . '.php';
});

$obj  = new MyClass1();
$obj2 = new MyClass2();
```

## Constructors

Constructors can be interesting when they are extending other classes.

```php
class Pokemon {
	public $name;

	// Heres a constructor for any pokemon
	public function __construct() {
		$this->name = "unknown";
	}

	public function whoami() {
		echo $this->name;
	}

}

class pikachu extends Pokemon {

	// We wont overwrite the __construct so we are still "unknown"
	// Because we inherited both $name and __construct from class Pokemon
	public function whoami() {
		echo $this->name;
	}
}

// create an object
$pokemon = new pikachu();

echo $pokemon->whoami()."<br>";
// OUTPUT: "undefined"
```

Now with overwriting on the __construct.

```php
class Pokemon {
	public $name;

	public function __construct() {
		$this->name = "unknown";
	}

	public function whoami() {
		echo $this->name;
	}

}

class pikachu extends Pokemon {

	// Now we overwrite the __construct so every pikachu has the $name "pika"
	public function __construct() {
		$this->name = "pika";
	}

	public function whoami() {
		echo $this->name;
	}
}


// create an object
$pokemon = new pikachu();

echo $pokemon->whoami()."<br>";
// OUTPUT: "pika"
```

Similar to construct, destruct does the opposite (runs before the object is destroyed).

```php
class MyDestructableClass
{
	function __construct() {
		print "In constructor"."<br/>";
	}

	function __destruct() {
		print "Destroying " . __CLASS__ ."<br/>";
	}
}

$obj = new MyDestructableClass();
echo <<<EOD
now the program will close and
the class will be destroyed after this message <br/>
EOD;
```

## Visibility

referring to `public, protected, or private`.

* Public can be accessed everywhere.
* Protected can be accessed only within the class itself and by inheriting and parent classes
* Private may only be accessed by the class that defines the member.

This applies to properties, methods, and constants (and maybe other stuff but im not sure).

## Scope Resolution Operator

This has been covered/mentioned before too but i want to section it off for clarity.

The Scope Resolution Operator (or Paamayim Nekudotayim) is a token that allows access to **static**, **constant**, and **overridden** properties or methods of a class.

```php
class MyClass {
	const CONST_VALUE = 'A constant value';
}

// method 1
$classname = 'MyClass';
echo $classname::CONST_VALUE; // As of PHP 5.3.0

// method 2
echo MyClass::CONST_VALUE;
```

### Accessing the parent from a client (extended) class

```php
class Pokemon {
	public $name = "unknown";
	public const SOMEVAL = "secret";
}

class pikachu extends Pokemon {
	public function whoami() {
		// Use the parent::SOMEVAL to access the const from the parent
		// This will only work for constants
		echo parent::SOMEVAL."<br/>";
		echo "i am $this->name";
	}
}


// create an object
$pokemon = new pikachu();
$pokemon->name = "pika";

echo $pokemon->whoami()."<br>";
```

You can also call a static function from an extended class.

```php
class Pokemon {
	public $name = "unknown";
	public const SOMEVAL = "secret";

	function sayCheese() {
		echo "cheese!";
	}
}

class pikachu extends Pokemon {
	public function whoami() {
		echo parent::sayCheese()."<br/>";
	}
}


// create an object
$pokemon = new pikachu();
$pokemon->name = "pika";

echo $pokemon->whoami()."<br>";
```

## Static

Again as i already half covered this, i am recovering the static keyword for clarity.

Declaring class properties or methods as static makes them accessible without needing an instantiation of the class.

```php
// Heres a working example
class Foo {
	public static function aStaticMethod() {
		echo "hello";
	}
}

// call static method, it prints hello
Foo::aStaticMethod();
```

You can also do this with static properties.

```php
class Foo {
	public static $myVar = "hello world";
}

// call the static method will work
echo Foo::$myVar;
```

## Abstraction in PHP

PHP supports abstraction, to learn about abstraction you can read my notes under */notes/programming/principles_of_oop* or read the quote from the documentation which is really well explained.

> When inheriting from an abstract class, all methods marked abstract in the parent's class declaration must be defined by the child; additionally, these methods must be defined with the same (or a less restricted) visibility.
> Furthermore the signatures of the methods must match, i.e. the type hints and the number of required arguments must be the same. For example, if the child class defines an optional argument, where the abstract method's signature does not, there is no conflict in the signature.

```php

abstract class AbstractClass
{
	// Force Extending class to define this method
	abstract protected function getPokemon();

	// Common method that each extending class gets
	public function printOut() {
		print $this->getPokemon() . "<br/>";
	}
}

class ConcreteClass1 extends AbstractClass
{
	protected function getPokemon() {
		return "pikachu";
	}
}


$class1 = new ConcreteClass1;
$class1->printOut();
```

You are **NOT** allowed to extend the concrete class with your own custom parameters if its an abstracted function from AbstractClass.

```php
// NOT ALLOWED
class ConcreteClass1 extends AbstractClass
{
	// NOT ALLOWED BECAUSE WE ADDED A PARAMETER
	protected function getPokemon(string $name) {
		return $name;
	}
}
```

Lets fix by changing the *AbstractClass* to enforce the use of a $name parameter.

```php
abstract class AbstractClass {
	// Force Extending class to define this method
	abstract protected function getPokemon(string $name);

	// Common method that each extending class gets
	public function printOut() {
		print $this->getPokemon() . "<br/>";
	}
}
```

### The exception to the rule: Optional parameters

Extending the above rule that "a concrete class, when based on an abstraction CANNOT implement its own parameters for methods that are abstract". The exception to this is if its an optional parameter. Lets modify the concrete class to work with this.

```php
class ConcreteClass1 extends AbstractClass
{
	protected function getPokemon() {
		return $name;
	}
}
class ConcreteClass1 extends AbstractClass
{
	// This is valid because its optional
	protected function getPokemon(string $name="optional") {
		return $name;
	}
}
```

## Interfaces in PHP

Another great explanation from the docs.

> Object interfaces allow you to create code which specifies which methods a class must implement, without having to define how these methods are implemented.

An interface is created the same way except replace `abstract class AbstractClass {...}` with `interface interfaceClass`.

Interfaces also support constructors for the factory design pattern.

The difference between an abstract class and an interface is that an **interface** does not require you to describe how a function is implemented, its entirely up to the implementing function, IE no methods are allowed in the *"interface pokeTemplate"*, only empty function decelerations.

```php
// Declare the interface 'pokeTemplate'
interface pokeTemplate
{
	public function setName($name, $age);
	public function getName();
}

class Template implements pokeTemplate
{
	private $pokemon;
	private $pokeAge;

	public function setName($name, $age)
	{
		$this->pokemon = $name;
		$this->age = $age;
	}
  
	public function getName()
	{
		return (object) [
			"name" => $this->pokemon,
			"age" => $this->pokeAge
		];
	}
}

$poke = new Template();
$poke->setName("pikachu", 10);
echo $poke->getName()->name;
```

## Traits

> Traits are a mechanism for code reuse in single inheritance languages such as PHP. A Trait is intended to reduce some limitations of single inheritance by enabling a developer to reuse sets of methods freely in several independent classes living in different class hierarchies.

Furthermore from the docs...
> Traits are a mechanism for code reuse in single inheritance languages such as PHP. A Trait is intended to reduce some limitations of single inheritance by enabling a developer to reuse sets of methods freely in several independent classes living in different class hierarchies.

```php
trait Hello {
	public function sayHello() {
		echo 'Hello ';
	}
}

trait World {
	public function sayWorld() {
		echo 'World';
	}
}

class MyHelloWorld {
	use Hello, World;
	public function sayExclamationMark() {
		echo '!';
	}
}

$o = new MyHelloWorld();
$o->sayHello();
$o->sayWorld();
$o->sayExclamationMark();
```

```output
Hello World!
```

### Precedence of traits

Inherited methods are overwritten by traits. In this example *Base sayHello()* is overwritten by *trait sayHello()* when we call *sayHello()* from an object that is extending a base with a method thats the same method name that the traits method name.

```php
class Base {
	// this version of sayHello is overwritten by the trait in MyHelloWorld
	public function sayHello() {
		echo 'Hello ';
	}
}

trait SayWorld {
	public function sayHello() {
		parent::sayHello();
		echo 'World!';
	}
}

// the class MyHelloWorld extends base. base HAS sayHello()
// the class MyHelloWorld then uses the trait sayWorld()
class MyHelloWorld extends Base {
	// When we sayWorld, the sayWorld function asks its parent to parent::sayHello first
	use SayWorld;
}

$o = new MyHelloWorld();
$o->sayHello();
```

```output
Hello World!
```

You can resolve this conflict and overriding by using an alias. Note that my intellisense extension things that $t->smallTalk(); is undefined. However this still works. You can also use the "as" keyword to duplicate already used aliases.

```php
trait A {
	public function smallTalk() {
		echo 'a';
	}
	public function bigTalk() {
		echo 'A';
	}
}

class Talker {
	use A, B {
		B::smallTalk insteadof A;
		A::bigTalk insteadof B;
		B::bigTalk as talk;
	}
}

$t = new Talker();
$t->smallTalk();
```

#### Expanding on the as keyword

The "*as*" keyword can also be expanded to change the visibility of traits.

```php
trait HelloWorld {
	public function sayHello() {
		echo 'Hello World!';
	}
}

// Change visibility of sayHello
class MyClass1 {
	use HelloWorld { sayHello as protected; }
}
```

#### Traits as properties

Traits can also implement properties.

```php
trait PropertiesTrait {
	public $x = 1;
}

class PropertiesExample {
	use PropertiesTrait;
}

$example = new PropertiesExample;
$example->x;
```

## Anonymous classes

Link anon functions, just a class instead! In the example from the docs they use a make believe "util" class that takes the anon class like a one off object, in fact you can think of an anon class like a one off object as you will only make it once.

Anon classes can also extend other classes, implement interfaces, or use traits. All objects created by the same anonymous class declaration are instances of that very class. So if you were using the same.

```php
$util->setLogger(new class {
	public function log($msg)
	{
		echo $msg;
	}
});
```

Heres an example of a function that returns an anonymous class, each of these classes are considered **instances** of that same class, however they are not using the same class name (the class name is randomly generated internally and not important).

```php
function anonymous_class()
{
    return new class {};
}

$myClass = new anonymous_class();
```
