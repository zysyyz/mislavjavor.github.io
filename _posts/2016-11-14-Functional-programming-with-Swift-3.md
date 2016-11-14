---
layout: post
title: Functional programming with Swift 3
---


# Functional programming with Swift 3.x

## Disclaimer

This article assumes that you are familiar with the basics of Swift 3.x

## Intro to functional programming

Functional programming is a software development paradigm
that puts empahsis on immutable state, pure functions and declarative 
programming. If these terms mean nothing to you, don't despair. We'll cover
all of them in this article. Since Swift has excellent functional programming
support, it would really be a shame to ignore this part of the language 
and focus on pure object-oriented development. So in no particular order, here
are some of the main features of functional programming.


### 1. Functions as first-class objects

This rule is the basis of every programming language. It means that
functions are objects (citizens) just like any other value. 
It means we can assign functions to variables, pass them as function
parameters, return them from other functions and do all other forms
of data manipulation usually reserved for objects and primitives.

So for example - we can:

```swift
func bar() {
    print("foo")
}

let baz = bar

baz() // prints "foo"
```

See how we passed that function just like any other object? We can also 
do something like this:

```swift
func foo() {
    print("foo")
}

func bar() {
    print("bar")
}

func baz() {
    print("baz")
}

let functions = [
    foo,
    bar,
    baz
]

for function in functions {
    function() // runs all functions
}
```

Functions in an array?!? Yup! You can do lots of interesting things 
when functions are first-class citizens in your language.

We can also define a concept called

**Anynonmous functions a.k.a Closures**

They are just functions which do not have a name. They perform some action,
but are usually needed only in one place - so we don't bother naming them

The syntax for defining anonymous functions takes many forms - but in it's
most verbose form it's this:

```swift 
{(parameter1 : Type, parameter2: Type, ..., parameterN: Type) -> ReturnType in 

}
```

They can be defined like this:

```swift
let anonymous = { (item: Int) -> Bool in 
    return item > 2
}

anonymous(3)
```

But this function has a name you might say! It's clearly called `anonymous`. 
Did I not just claim that they had no name? 

Well, yes I did. And it's true. The anonymous function is only the code on the
right side of the `=` operator. We're just joining it to a variable - just
like we joined the `bar` function earlier. We could have just done this:

```swift
({ (item: Int) -> Bool in 
    return item > 2
}(4)) // returns true
```
Now we just moved the right part of the previous expression and called the 
function with the number 4 right when we defined it. So as you can see -
the function can be anonymous without any issues from the language. 
Anonymous function will be very useful later on  - in the higher order
functions chapter. Also it's important to note that each and every function
has it's own type. The function type describes the input parameters and 
the return value. It looks like this:

```swift
let anon : (Int, Bool) -> String = { input,condition in 
    if condition { return String(input) }
    return "Failed"
}

anon(12, true) // returns "12"
anon(12, false) // returns "Failed"
```
The part where we define the type is:

```swift
(Int, Bool) -> String
```

It means the function takes an `Int` and a `Bool` and returns a `String`.
Since functions have types - the compiler enforces compile time type safety

This means that you can do this:

```swift
var functionArray = [(Int, Bool)->String]()

let one : (Int, Bool) -> String = { a, b in 
    if b { return "\(a)"}
    else { return "Fail" }
}

functionArray.append(one)

let two = { (a: Int, b: Bool) -> String in 
    if b { return "\(a)" }
    else { return "FailTwo" }
}

functionArray.append(two)
```
And it will work just fine. But if you try this:

```swift
let three : (Int)->Bool = { item in 
    return item > 2
}

functionArray.append(three)
```

You'll get an error message saying:

```
ERROR at line 21, col 22: cannot convert value of type '(Int) -> Bool' to expected argument type '(Int, Bool) -> String'
functionArray.append(three)
```

The array type is `(Int, Bool) -> String` and function `three` has a 
type `(Int)->Bool` and they're incompatible - just you couldn't add a
`String` to an `Int` array.

The type syntax can sometimes be a little bit cumbersome, so Swift uses
a `typealias` operator to shorten complex type declarations. We can shorten
our previously used declaration like this:

```swift
typealias Checker = (Int, Bool) -> String

var funcArray = [Checker]()

let one : Checker = { a, b in 
    if b { return "\(a)" }
    return "Fail"
}

funcArray.append(one)
```

There is also an additional part of syntax sugar - Swift exposes the 
function parameters as `$0,$1,$2...$n` inside of the function.

This is very useful for writing one-liners. For example let's rewrite the 
`one` function as a one-liner 

```swift
let two: Checker = { return $1 ? "\($0)" :  "Fail" }
```

The `$0` is the first argument of type `Int` and `$1` is the second one
of type `Bool`

Also, if a closure is made of only one line - it returns that line by default
So there is an even shorter way to write the funciton:

```swift
let three: Checker = { $1 ? "\($0)" : "Fail" }
```

Now thats some concise programming!


### 2. Pure functions

Pure functions are merely functions which do not depend on anything besides
their function parameters. They do not change outside state, nor are affected
by changes to the outside state. That's it.

For example:

```swift 
func countUp(currentCount: Int) -> Int {
    return currentCount + 1
}
```

is a pure function. However, 

```swift 
var counter = 0
func countUp() -> Int{
    counter += 1
    return counter
}
```

is not a pure function. And that's all there is to say about pure functions.
They are a great way to make your code extremely testable and can save
you a lot of time when things go south since they make debugging much 
easier. They are also great for parallel programming since each new call
is depended only on the variables it created itself - thus avoiding all
of the parallel/concurrent programming pitfalls

### 3. Higher order functions

Simply put, higher order functions are functions which take other
functions as arguments and/or return other functions as a result.
This simple concept lies at the core of functional programming. It 
enables us to create very useful abstractions and deal with the 
specifics during the call to the function - greatly reducting code doubling.

What does this mean practically?

For example -  we might need to remove all elements from a sequence that
are smaller than a certain number 

An imperative approach would be:

```swift
func filter(sequence: [Int], elementsSmallerThan border: Int) -> [Int] {
    var newSequence = [Int]()
    for item in sequence {
        if item < border {
            newSequence.append(item)
        }
    }
    return newSequence
}

filter(sequence: [1,2,3,4], elementsSmallerThan: 3) // returns [1,2]
```

Then later on - we might need to remove all the elements that are 
larger than a certain number

```swift
func filter(sequence: [Int], elementsLargerThan border: Int) -> [Int] {
    var newSequence = [Int]()
    for item in sequence {
        if item > border {
            newSequence.append(item)
        }
    }
    return newSequence
}

filter(sequence: [1,2,3,4], elementsLargerThan: 2) // returns [3,4]
```

Now already, we have a lot of code doubling. Both times we need to
go through a couple of steps:
    
1. Create a new sequence
2. Iterate over items
3. Check the condition and append if it's fulfilled
4. Return the sequence

So how could higher-order functions help us?

They'll allow us to create a *single* function which can take a condition
as a parameter and filter out a sequence of integers any way we like

```swift
func filter(sequence: [Int], condition: (Int)->Bool) -> [Int] {
    var newSequence = [Int]()
    for item in sequence {
        if condition(item) {
            newSequence.append(item)
        }
    }
    return newSequence
}

let sequence = [1,2,3,4]

let smaller = filter(sequence: sequence, condition: { item in
    item < 3
})

let larger = filter(sequence: sequence, condition: { item in
    item > 2
})

print(smaller) // prints [1,2]
print(larger) // prints [3,4]
```

Now we can filter out our sequence based on any condition we want!
We removed the boilerplate code and our calls look more expressive than
ever.

We can even leverage Swifts inbuilt syntax sugar for calling functions
functions that are at the end of the argument list of another function

```swift
// We can ommit the parameter name and put the function as an 
// anonymous function outside of the round brackets
let equalTo = filter(sequence: sequence) { item in item == 2}

// We can even ommit calling the parameters by name - if we
// use the "exposed" $0 variable which refers to the first function
// parameter
let isOne = filter(sequence: sequence) { $0 == 1}
```
In addition to putting functions as parameters, we can also return functions.
Let's say we need to pick a function that modifies an `Int` array based on some
condition:

```swift
typealias Modifier = ([Int]) -> [Int]

func chooseModifier(isHead: Bool) -> Modifier {
    if isHead {
        return { array in 
            Array(array.dropLast(1))
        }
    } else {
        return { array in 
            [array[0]]
        }
    }
}

let head = chooseModifier(isHead: true)
let tail = chooseModifier(isHead: false)
```

Here we see how the `chooseModifier` function returns one of the two
possible functions depending on the condition. Another great use is when 
we need to get different variants of a same function. 

Let's define a function which checks weather a certain number is in some 
defined range:

```swift
typealias Range = (Int) -> Bool 

func range(start: Int, end: Int) -> Range {
    return { point in 
        return (point > start) && (point < end)
    }
}

let fourToFifteen = range(start: 4, end: 15)

//Check if a number belongs to a range
print(fourToFifteen(14)) //true
print(fourToFifteen(16)) //false
```

We've defined the `range` function as a function which returns another
function and that's enabled us to spew various variants of the `range` 
function without a large amount of effort. 

When we're talking about passing functions as parameters to other functions,
one very important use case apperats:

**Conditional parameter evaluation**

So let's say you have a parameter which may or may not be evaluated. You 
can't be sure how the user will generate the parameter, but it might be
computationally expensive to generate the value. You can make sure
the parameter is only generated *when it's being used* by taking advantage
of higher-order functions. For example:

```swift
func stringify1(condition: Bool, parameter: Int) -> String {
    if condition {
        return "\(parameter)"
    }
    return "Fail"
}

func stringify2(condition: Bool, parameter: ()->Int) -> String{
    if condition {
        return "\(parameter())"
    }
    return "Fail"
}

let a = stringify1(condition: false, parameter: expensiveOperation())
let b = stringify2(condition: false, parameter: { return expensiveOperation()})
```

The first example is the usual way we pass parameters. The parameter is 
called every time the function is called, regardless of weather it's being
used. 

The second version of the function optimizes for this behaviour by 
invoking a function that returns the value of the parameter *only* when
it's actually being used. 

Now this behaviour is great, but a little bit messy to use. That's why
Swift has an inbuilt mechanism of handling these types of behaviour.

**@autoclosure annotation**

If we write the function to look like this:

```swift 
func stringify3(condition: Bool, parameter: @autoclosure ()->Int) -> String {
    if condition {
        return "\(parameter())"
    }
    return "Fail"
}
```

where the `@autoclosure` annotation is put before the type of the second 
parameter - we can leverage the delayed (a.k.a. *lazy* ) loading functionallity,
while keeping the function call syntax of the normal function. 

If we want to call the `stringify3` function we do it like this:

```swift
stringify3(condition: true, parameter: 12)
```

Best of both worlds!

### 4. Function composition a.k.a. Currying

Function currying is a process in which we can take a function which 
takes n parameters and break it up into (at max) n functions which take one
parameter. At that point - each function has a fraction of the responsibility
of the entire "large" function. We can use that to break down complex functions
into a set of simple functions. The most basic example of function
currying is:

```swift
func add(_ a: Int) -> ((Int)-> Int) {
    return { b in 
        return a + b
    }
}

add(2)(3) // returns 5
```

Here we've defined a function `add` which takes an `Int` parameter and
returns a function which takes an `Int` parameter and returns an `Int`.
Then we called the function with the `add(2)(3)`. This is not any
special syntax - it's just a consequence of the fact that the return value
of the `add(2)` function is a function which takes an integer. 

Let's say we do the following:

```swift
typealias Modifier = (String)->String

func uppercase() -> Modifier {
    return { string in 
        return string.uppercased()
    }
}

func removeLast() -> Modifier {
    return { string in 
        return String(string.characters.dropLast())
    }
}

func addSuffix(suffix: String) -> Modifier {
    return { string in 
        return string + suffix
    }
}
```

These are some functions which take a String and modify it. 
Now if we wanted to call them separately we could do it like this:

```swift
let uppercased = uppercase()("Value")
let removed = removeLast()(uppercased)
let withSuffix = addSuffix()(removed)
```

And with function composition it looks like:

```swift 
let a = addSuffix(suffix: "suffix")(removeLast()(uppercase()("IntitialFunction")))
```

Very short, but very readable and, honestly, quite confusing. 
That's why we can define a `compose` function which takes two funcions and
composes them into a single one

```swift 
func compose(_ left: @escaping Modifier, _ right: @escaping Modifier) -> Modifier {
    return { string in 
        left(right(string))
    }
}
```

Now we can make a function like this:

```swift 
let a = compose(compose(uppercase(), removeLast()), addSuffix(suffix: "Abc"))("IntitialValue")
```

Now the function order and the bracket structure are much more clear, but it's 
still a lot of brackets. One way to solve this is to implement a currying operator,
but another way which I prefer is to wrap the `Modifier` in a structure. After
doing that we get this code: 

```swift
struct Modifier {
    
    private let modifier: (String) -> String
    
    init() {
        self.modifier = { return $0 }
    }
    
    private init(modifier: @escaping  (String)->String) {
        self.modifier = modifier
    }
    
    var uppercase: Modifier {
        return Modifier(modifier: { string in 
            self.modifier(string).uppercased()
        })
    }

    var removeLast : Modifier {
        return Modifier(modifier: { string in 
            return String(self.modifier(string).characters.dropLast())
        })
    }

    func add(suffix: String) -> Modifier {
        return Modifier(modifier: { string in 
            return self.modifier(string) + suffix
        })
    }
    
    func modify(_ input: String) -> String {
        return self.modifier(input)
    }
}

// The call is now clean and clearly states which actions happen in 
// which order
let modified = Modifier().uppercase.removeLast.add(suffix: "suffix").modify("InputValue")

print(modified)
```

So we've used functional programming to create a `Modifier` for strings
which can now use arbitrarily complex modifications such as:

```swift
let modified = Modifier().add(suffix: "Initial").uppercase.add(suffix: "Later").removeLast.removeLast.removeLast.add(suffix: "Other").modify("Value")
```

Although these examples lack a dose of real-world usefulness - they are great at 
showing how functional programming can be used to build highly modular code.

The last example which uses `struct` breaks the functional pattern a little
bit since it holds the private `modifier` variable. It sacrifices some of the
safety for a little bit of syntactic sugar. The approach which would avoid all of this
would be to defin an `infix` operator for currying the functions. 

Then the entire thing could look something like this:

```swift 
typealias Modifier = (String)->String

func uppercase() -> Modifier {
    return { string in 
        return string.uppercased()
    }
}

func removeLast() -> Modifier {
    return { string in 
        return String(string.characters.dropLast())
    }
}

func addSuffix(suffix: String) -> Modifier {
    return { string in 
        return string + suffix
    }
}

precedencegroup CurryPrecedence {
    associativity: left
}

infix operator |>  : CurryPrecedence

func |> ( left: @escaping Modifier, right: @escaping Modifier) -> Modifier {
    return { string in 
        right(left(string))
    }
}

let modified = uppercase() |> removeLast() |> addSuffix(suffix: "123")
print(modified("Abcd"))
```

This is the more *functional* approach since the functions are all pure
and we keep the entire thing stateless.

## Conclusion

Functional programming is a great way to improve the safety and testability of
your code. But remember - it's not a one-size-fits-all paradigm. In fact,
no paradigm is. So use it when you need it, and combine it with the
features you love in Swift to make better software.

<div id="disqus_thread"></div>
<script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
     */
    /*
    var disqus_config = function () {
        this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
        this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() {  // DON'T EDIT BELOW THIS LINE
        var d = document, s = d.createElement('script');
        
        s.src = '//mislavjavor.disqus.com/embed.js';
        
        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>

<script language="JavaScript">var fhsh = document.createElement('script');var fhs_id_h = "3200054";
fhsh.src = "//s1.freehostedscripts.net/ocount.php?site="+fhs_id_h+"&name=Visits&a=1";
document.head.appendChild(fhsh);document.write("<span id='h_"+fhs_id_h+"'></span>");
</script>
