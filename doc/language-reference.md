TSML - Language Reference
=========================

Introduction
------------

While [TypeScript](http://www.typescriptlang.org) give to developer a type safe superset of javascript suited for 
large scale web application development, it does not resolve the problem HTML was not designed for dynamic applications 
but static documents. 

With the lack of extensibility, and the poor possibility offered by the html syntax, developer often ends up by being forced 
to create their entire application in script.  
Framework like [Sencha Ext JS](http://www.sencha.com/products/extjs) or [SmartClient](http://smartclient.com/), 
completly abstract the Dom, and provide a 'JSON-Like' reprensation of a component hierarchy. However a language Like JavaScript is not really suited to user interface creation. By the nature of JavaScript verbose syntax it produce hardly readable code, harden the interaction between designer and developer, and decrease the overall productivity of the overall developer team.

//TODO speak about http://angularjs.org/ and backbone

Language like [MXML](http://en.wikipedia.org/wiki/MXML) or [XAML](http://en.wikipedia.org/wiki/XAML) in other technologies offer a more "declarative", more consise, and by fact a more productive way of defining thoses, while giving access to a fully type-safe environement.

TSML (TypeScript Markup Language) is and XML-based language designed to integrate with the TypeScript compiler by 
producing TypeScript abstract syntax tree.
By doing this every TSML document will produce class module or function that will be consumable by other TypeScript unit, 
and will benefit of the type checking of the TypeScript compiler.




TSML Document
-------------

###The \<tsml> root tag

Each TSML document start with the <code>\<tsml></code> tag, and empty document look like this :

	<tsml>
    </tsml>
	

Defining variables
--------------------
To define a variable simply create a tag inside your tsml document, the variable name will be valorized to the id of the this tag :

	<tsml>
		<string id='myVar'>Hello World</string>
	</tsml>
	
is equivalent to the following typescript code :

	var myVar = 'Hello World';
	
if no id attribute is provided, a random unique id will be assigned to the variable (todo should we not make the id attribute mandatory ? )

Basic type
----------

TSML bind a certain number of predefined tag as predefined type here is a list of those tags :

###string : 
	
	<string>string content</string>

is equivalent to :

	'string content';
	
###number :

	<number>2.5</string>
	
is equivalent to :

	2.5;
	
###boolean :

	<boolean>true</boolean>
	
is equivalent to :

	true;

###array :
	<array>
		<string>hello</string>
		<string>world</string>
	</array>
	
	
is equivalent to :

	['hello', 'world'];
	
###object :

	<object prop1='world' prop2='hello'>
		<boolean id='prop3'>false</boolean>
	</object>

is equivalent to :

	{
		prop1 : 'world',
		prop2 : 'hello',
		prop3 : false
	};
	
###class
	
	<class id="Greeter" />
	
is equivalent to :

	class Greeter {
		
	}

class definition will be further developed below

###module
	
	<module id="a.b" />
	
is equivalent to :

	module a.b {
		
	}

module definition will be further developed below

###function
	
	<function id="sayHello" />
	
is equivalent to :

	function sayHello() {
		
	}

function definition will be further developed below

	
	
Complex type
------------

Non TSML basic tag are considered like instance of the correspondly named class, for example if we have a class "Foo" in the context of our document :

	<Foo id="foo" />
	
is equivalent to :

	var foo = new Foo();

(todo should we introduce a special attribute constructorArg ?)

###children

todo

Import an namespace
-------------------

In tsml namespace has 3 purpose (todo don't know how to formulate this)
	
	<tsml xmlns:view='import:view/*' 
		  xmlns:ui='import:ui'
		  xmlns:foo='foo.*'>
		  
		  <ui:Button id='button' />
		  <ui:Checkbox id='checkbox' />
		  
		  <view:View1 id='view1' />
		  <view:View2 id='view2' />
		  
		  <foo:Bar id='bar' />
		  
	</tsml>
	
is equivalent to :

	import ui = require('ui');
	import View1 = require('view/View1');
	import View2 = require('view/View2');
	
	var button = new ui.Button();
	var checkbox = new ui.Checkbox();
	var view1 = new View1();
	var view2 = new View2();
	var bar = new foo.Bar();
	

Exporting
---------

to export a variable, class etc use the special attribute export :

	<string id='myVar' export='true' >Hello world!</string>
	
is equivalent to :

	export var myVar = 'Hello World';
	
	
	<string id='myVar' export='this' >Hello world!</string>
	
is equivalent to :

	var myVar = 'Hello World';
	export = myVar;


Setting property
----------------

###Attribute property

To set a property on an object you can use an attribute of the xml tag representing this object :

	<Bar id='bar' />
	<Foo id='foo' message='Hello World!' count='5' bar='bar' />
	
	
is equivalent to :

	var bar = new Bar();
	var foo = new Foo();
	foo.message = 'Hello World !';
	foo.count = 5;
	foo.bar = bar;
	
note that the tsml compiler infer the property value using the type of the property, string typed property and any typed property will receive the value attribute as string literal, boolean property will accept <code>true</code> or <code>false</code> as value, complex type will accept a reference to a variable etc …


###child tag property
	
You can also using a child to set a property value, for example :

	<Foo id='foo'>
		<bar>
			<Bar />
		</bar>
	</Foo>

is equivalent to :

	var foo = new Foo();
	foo.bar = new Bar();

(note property child tag must have the same namespace than their parent tag)


###interpolation

To set a property to a reference of an existing variable, you can use the curly brace characters ({ }) :

  
	<string id='myMessage'>Hello World!</string>
	<Foo id='foo' message='{myMessage}' />

is equivalent to :

	var myMessage = 'Hello World!';
	var foo = new Foo();
	foo.message = myMessage;

in case of string property interpolation can be used as a part of the property value :
	
	<string id='myMessage'>Hello</string>
	<Foo id='foo' message='{myMessage} World!' />

is equivalent to :

	var myMessage = 'Hello';
	var foo = new Foo();
	foo.message = myMessage + ' World!';
	
TODO speak of binding and dual binding here

The script tag
--------------

the special <code>\<script></code> tag allow to include typescript inside of a tsml document, dependings of the contex of the tag the authorized syntax inside the script tag, and the context execution of the script varies.

### Under the root tag

Placed as a children of the root <code>\<tsml></code> tag, the script tag  :

	<tsml>
		<script>
			var test = 'Hello World';
		</script>
	</tsml>

is equivalent to :

	var test ='Hello World';
	

### Inside a class declaration

Inside a class declaration, the script tag allows you to define property, methods and constructor of this class, the parser will consider that you are inside the class body :

	<tsml>
		<class id="Greeter">
			<script>
				sayHello() {
					console.log('Hello World');
				}
			</script>
		</class>
	</tsml>
	
is equivalent to :

	class Greeter {
		sayHello() {
			console.log('Hello World');
		}
	}		
	
### Inside a module declaration

Inside a module declaration, the script tag allow you to define class function etc.. inside your module :

	<tsml>
		<module id="greet">
			<script>
				class Greeter {
					sayHello() {
						console.log('Hello World');
					}
				}	
			</script>
		</module>
	</tsml>
	
is equivalent to :

	module greet {
		class Greeter {
			sayHello() {
				console.log('Hello World');
			}
		}	
	}	
	
Class declaration
-----------------

In a TSML document you declare a class by using the <code>\<class></code> tag. this tag offers special attributes to define the base class, interface implemented etc etc.

###A simple class

 	<tsml>
		<class id="Greeter" />
	</tsml>
	
is equivalent to :

	class Greeter {
		
	}

###Specifying the base class

To Specify the base class of a defined class, use the <code>extends</code> attribute

	<tsml>
		<class id="BaseGreeter" />
		<class id="Greeter"  extends="BaseGreeter"/>
	</tsml>
	
is equivalent to :

	class BaseGreeter {
		
	}
	
	class Greeter extends BaseGreeter {
		
	}
	
###Implementing interfaces

To Specify a list of implemented interfaces, use the <code>implements</code> attribute 

	<tsml>
		<script>
			interface IGreeter {
			
			}
			interface IHelloWorld {
			
			}
		</script>
		<class id="Greeter" implements="IGreeter IHelloWorld"/>
	</tsml>
	
is equivalent to :

	interface IGreeter {
			
	}
			
	interface IHelloWorld {
			
	}

	class Greeter implements IGreeter,IHelloWorld {
		
	}
	
###Generic

todo

###Default value for class property

To define default value fo your class you can use attribute :

 	<tsml>
		<class id="Greeter" extends="BaseGreeter" message="Hello World!"/>
	</tsml>
	
is equivalent to :

	
	class Greeter extends BaseGreeter {
		constructor() {
			super();
			this.message = 'Hello World!';
		}	
	}

note : in this example we assume that the BaseGreeter class define a property named <code>message</code> of type <code>string</code>.

you can also use a child tag 

	<tsml>
		<class id="Foo" extends="BaseFoo">
			<bar>
				<Bar />
			</bar>
		</class>
	</tsml>
	
is equivalent to :
	
	class Foo extends BaseFoo {
		constructor() {
			super();
			this.bar = new Bar();
		}	
	}

note : in this example we assume that the BaseFoo class define a property named <code>bar</code> of type <code>Bar</code>.

###Defining property of a class

To define properties of a class TSML provide a special tag <code>\<declaration></code> every object of this tag will be considered a property of the defined class :

	<tsml>
		<class id="Foo">
			<declaration>
				<object id="myObject" message="Hello World!"  />
			</declaration>
		</class>
	</tsml>
	
is equivalent to :
	
	class Foo  {
		myObject = { message : 'Hello World' };
	}
	
### the script tag

In the context of a class defintion the script tag is considered like a part of the class body.

	<tsml>
		<class id="Greeter">
			<script>
				constructor(private message : string) { }
				greet() {
					console.log(this.message);
				}
			</script>
		</class>
	</tsml>
	
is equivalent to :
	
	class Greeter  {
		constructor(private message : string) { }
		greet() {
			console.log(this.message);
		}
	}
	
###children

todo

Module Declaration
-------------------

Function Declaration
-------------------




