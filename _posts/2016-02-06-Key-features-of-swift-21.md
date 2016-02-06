# Key features of Swift 2.1 

## Introduction
Swift, the programming language Apple designed to replace Objective-C, has split the opinions of many people during it's existence.
For hardcore Objective-C developers it's the incarnation of Lucifer himself, unleashed by demons in Apple to destroy their beloved language. 
But for me, and others like me, it's a gift from the good developers in Apple to us common-folk. It simplifies the development, it's almost infinitely extensible, it's modern and developing in it is fast. And by fast - I mean very,very fast. Since Swift approaches programming from a pragmatic rather than idealistic standpoint, once you get a hang of it - you can develop loads of functionality in only a few lines of code. Being an iOS and Android developer, I can't help myself but compare it to using Java in Android. And in that regard, it leaves Java behind in it's tracks. It has all the features Java has (only the language, not the library) and then some. It's verbose enough when it needs to be (with named external parameters for example) and very concise when you perform common tasks (filtering lists, unwrapping nils, initialising structures etc...). So now, in no particular order, here is a list of my favourite features of the Swift Programming Language

## Functions as first-class citizens
Functions in Swift are objects. Just like Ints, just like Strings and all the rest. Their type is defined by their signature. So for example `func refactorCode(codeBase : String, refactorer : Refactorer) -> String` would have a type `(String,Refactorer)->String` . Since functions are objects, you can do a lot of interesting things with them:

- Store them in variables and constants
- Pass them as arguments to a function
- Return them from a function
- Store them in Arrays and Dictionaries

This is one of those things that don't seem to be such a big deal when you read about it in the language spec, but it's extremely useful in day to day development. For example you can create a service that loads code Asynchronously and request the function as a parameter

```swift
func performAsyncCall(params : RequestParams, onCompleteHandler : (String)->Void)
```

And then in the calling code you can make a function that handles the request

```swift
func handleRequest(result : String){ ... }
```

And pass it as the function parameter

```swift
...
performAsyncCall(params: requestParams, handleRequest)
...
```
It works asynchronously and it looks concise. I prefer a different way of achieving this behaviour which I will show later in this post

## Tuple data type
Before going further, I must mention the Tuple data type. It's embedded in the language and it enables easy creation of objects without an accompanying class. The usefulness of tuples can't be overstated as they are, in my opinion, one of the strongest features of the language.

A tuple is an ordered pair of values of different types. In Swift we declare tuples by listing them within parentheses and separating them by commas. 
```swift
let someTuple = ("Hello", 2, 3.15, new DBContext("db_conn_string"))
```
By default you can access the values of the tuple by their position in the tuple

```swift
someTuple.0 //returns "Hello"
someTuple.1 //returns 2
someTuple.3 //returns an instance of DBContext
```
I use this method rarely since it obscures too much information for my taste. But fortunately, in Swift we are able to name the tuples

```swift
let person = (name: "Tyler Durden", age: 33, mentalState: MentalStateEnum.Schizophrenic)
```
And then access them by their name

```swift
print("Patient name: \(person.name), patient age: \(person.age), patient mental state: \(person.mentalState)")
```

This makes writing some code very short and concise and it enables us to have a function return multiple values - extremely useful for HTTP responses
```swift
func makeHttpRequest(request : HttpRequest) -> (httpResponse: Response, error : NSError?){
... 
return (response, error)
}
```
We declared a very useful data classification by splitting the data to response and error since now we can check if there is an error and act accordingly. This significantly increases the readability of our code.

You can also use tuples as a quickhand way of initialising multiple variables or constants:

```swift
var (x,y,z) = (2, 7, 3.554)
```
x , y  and z are set to the respective values of corresponding indices 

One more cool use of tuples if for exposing the index value of an element in an array while iterating over it - courtesy of the `.enumerate()`function 
```swift
for (index, value) in someArray.enumerate(){
...
}
```
## Optionals
Checking for nils (Swift equivalent of null) is a tedious task which should be, if at all possible, avoided. Swift has a syntax shaped in a way that, by default, you **must** pay attention to what each variable or constant can be. If you declare a variable like this
```swift
var someValue : String
```
You cannot set it to nil in any part of your code. Any attempt of setting it to nil will result in a compile time error. Even not initialising it in the constructor of the class will be a compile time error. This variable must, at every point during it's lifecycle, have a value. 
If you want to declare a variable that **can** be nil, then you must **explicitly** specify it like this
```swift
var someValue : String?
```
Since swift compiler has the ability to infer types, it will always assume that the type of your variable is not optional. So this code
```swift
var someValue = "This is a value"
someValue = nil
```
**will not work**. The compiler defaults to a non-nil type by default. A behaviour that is complementary to that of Java for example - which defaults to a nullable type.

In order to access values of optionals, we must unwrap them first. You can't just say `let someConst = someOptional` and hope for the best at runtime. You should't be doing this in any other languages as well, but you still do. Because you don't care.
Swift makes doing this very **explicit** and very **ugly**. In order to force unwrap a value, you must but an (!) exclamation mark after it like so
```swift
let someConst = someOptional!
```
this looks bad. And making bad practices explicit rather than default is always a good decision when designing a programming language.
Now, the *right* way of unwrapping in Swift is 
```swift
if let unwrappedValue = someOptional{
//perform success code
} else{
//value someOptional was nil -> handle accordingly
}
```
or
```swift
guard let unwrappedValue = someOptional else{
//unwrapping failed, handle accordingly
}
unwrappedValue.performAction() //use unwrappedValue safely, knowing it was unwrapped
```
This way makes it abundantly clear what you're trying to do. And you can always use the good-old nil coalescing operator `let unwrapped = optionalValue ?? defaultValue`which sets the unwrapped to `optionalValue` if it's not nil or sets it to `defaultValue` if `optionalValue` is nil

## Closures

Closures are anonymously defined bits of code that perform an action. They are, like functions, first-class citizens and can be dealt with as we please. Basic closure syntax is
```swift
{
(param_1, param_2,...,param_n-1, param_n)->ReturnType in
//perform some action
}
```
This type of invoking closures is most verbose, but I don't use it very often since I like my code to be concise. An example, referring to the function from the first chapter would be
```swift
performAsyncCall(params, {
(result->String)->Void in
//Do some work
})
```
Another way of writing the same code would be
```swift
performAsyncCall(params, {
(result) in
//Do some work
})
```
This takes advantage of swifts inbuilt type inference capabilities.
Yet another way of writing this would be
```swift
performAsyncCall(params, {
$0 //Do some work with $0 
})
```
If you don't name the parameter, and Swift is able to infer their type - the compiler automatically exposes them as variables with a dollar($) sign and their position index in the function definition

Now yet *another* way of writing the same code would be 
```swift
performAsyncCall(params){
$0 //Do some work with $0
}
```
This leverages the fact that the function onComplete is the **last** parameter of the `performAsyncCall`function. If a function has a function as it's last parameter, you can call it via a **trailing closure** - a block of code encapsulated by curly braces. 

You can use all three types of closure calls in trailing closures

## Super powerful enums
In most languages enums tend to be boring. Not in Swift. 
You declare enums pretty conventionally
```swift
enum Compass{
case North
case West
case East
case South
}
```
but very quickly things get very interesting. You can make certain types of enums receive parameters. Define an enum...
```swift
enum Barcode{
case QR(String)
case Regular2D([Int])
}
```
...and then simply initialise it and expose it's properties to the underlying code
```swift
var currentBarcode = Barcode.QR("someQrString")
...
...
switch currentBarcode{
case .QR(let qrCodeString):
print(qrCodeString) //Prints someQrString
}
```
And just like that you've categorised your data in a simple and intuitive way. 
And not just this - Swift enums can have properties. The properties must be computed in some way - most often based on the type of the enum

```swift
enum Compass{
case West,East,North,South

var coordinateBase : [Int : Int] {
switch self{
case .North :
return [0 : 1]
case .South :
return [0 : -1]
case .East :
return [1 : 0]
case .West : 
return [-1 : 0]
}
}
}
```
Then just use this enum in the following way
```swift
let position = Compass.East
print(position.coordinateBase) // returns [1 : 0]
```
This is particularly useful in creating enum routers - a concept widely used in Swift. Enum routers enable type safe and easy generation of request strings for API calls.

Note that the `switch` in Swift must be exhaustive

## Property observers

In Swift you can declare an *observer* for every class propery. There are four kinds of observers:

- `didSet`
- `willSet`
- `didGet`
- `willGet`

the syntax is like so 
```swift
var someProperty : Type? {
didSet{

}
willSet{

}
didGet{

}
willGet{

}
}
```
Not only is this extremely useful in asynchronous programming (throw some exceptions maybe?), it's a very powerful synchronisation tool. I've used it in the past for exposing variables that get written in the database.
```swift
class DBLayer{
public static let dbExpose = DBContext("conn_string")

var someVariable : String {
didSet{
dbExpose.update(forKey: "SOME_VARIABLE", someVariable)
dbExpose.save()
}
didGet{
//Increment counter maybe??
}
}
}
```
This snippet, as useless as it is (don't write it like this!) - demonstrates a very intuitive way to synchronise the database every time a variable gets set.

## Extensions

Swift has a simple way of extending the functionality of types. It should be used with caution because if you're not careful you could have the names of your functions collide with the names of other functions - leading to a compiler error

One thing I did using it is extend the inbuilt `String` and extend it with a generic function for parsing the `String` to JSON
```swift
extension String{
func mapToObject<TType : Mappable>()->TType?{
return Mapper<TType>().map(self)
}
}
```
Which can than be called like so
```swift
var someObjectInstance : ObjectType = stringJson.mapToObject()
```
## Do whatever you want with operators

In languages like `Java`, operators are set in stone. If you wanted to implement the addition of two instances of some type, you could only implement something like `.add(Type first, Type second)` and return an instance of the `Type` . In Swift you can

- Overload existing operators
- Implemet the operators as
- `prefix`
- `infix`
- `postfix` 
- Adjust the operator precedence
- Implement custom operators

I'll demonstrate this on the example of the `ComplexNumber` class

```swift
struct ComplexNumber{
public real : Double
public complex : Double
}

func + (left : ComplexNumber, right: ComplexNumber) -> ComplexNumber{
return ComplexNumber(left.real + right.real, left.complex + right.complex)
}

//Invert the number
prefix func - (number : ComplexNumber)->ComplexNumber{
return ComplexNumber(-number.real, -number.real)
}

postfix func ++ (number: ComplexNumber)->ComplexNumber{
return ComplexNumber(number.real + 1, number.complex + 1)
}

//Modulus of the complex number
prefix func /\ (number: ComplexNumber)->ComplexNumber{
//sqrt(x*x + y*y) implement
return ComplexNumber(...,...)//TODO
}
```

This is just some of the most interesting functionality. If you liked the article, follow me on [Twitter](http://twitter.com/mislavjavor) for updates