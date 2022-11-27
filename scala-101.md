## Getting Up To Speed with Scala ##
This set of notes contains all we need to get started and comfortable with Scala programming language.

### Scala Intro ###
1. A functional programming language.

1. Use REPL like feature called `ScalaWorksheet` (in Intellij IDE) to start experimenting with the language features.

1. Syntax (alot like Golang): [val | var] *name*: [data type] = [some value]. And dont need semicolons too (like Golang).  
For example:  
```
val numberOne: Int = 1

var helloWho: String = hello 
helloWho = hello + " there!"
```
4. Intro to `val`, `var`, data types, formatting, regex, `Boolean`
```
// VALUES are immutable constants; use VALUES over VARIABLES in Scala
// Also because Scala is functional, we will be passing functions around.
// A variable would not be thread-safe and can intro race conditions too.

val hello: String = "Ola!"

// VARIABLES are your regulars; they are mutable
var helloWho: String = hello // you can assign the constant value to a var
helloWho = hello + " there!" // helloWho: String = Ola! there!
println (helloWho) // Ola! there!


// Data Types
val numberOne: Int = 1
val truth: Boolean = true
val letterA: Char = 'a'
val pi: Double = 3.14159265
val piSinglePrecision: Float = 3.14159265f // 3.1415927
val bigNumber: Long = 123456789
val smallNumber: Byte = 127 // -127 to 127


// Formatting
println ("Normal println with concat: " + helloWho + numberOne + truth) // Normal println with concat: Ola! there!1true
println (f"Pi is about $piSinglePrecision%.3f") // 3.142; we want to 3 decimal places only
println (f"Zero padding on the left: $numberOne%05d") // 00001
println (s"I can use the s prefix to use values such referencing them via dollar sign: $helloWho $numberOne $truth")


// Regex
val theUltimateAnswer: String = "To life, the universe, and everything is 42."
val pattern = """.* ([\d]+).*""".r // .r means regex
val pattern(answerString) = theUltimateAnswer // 42; take theUltimateAnswer, eval the regex and answer goes back to answerString in ()
println (answerString) // 42


// Boolean
val isGreater = 1 > 2 // false
val isLesser = 1 < 2 // true
val isImpossible = isGreater && isLesser // false
val anotherWay = isGreater || isLesser // true

val picard: String = "Picard"
val bestCaptain: String = "Picard"
val isBest = picard == bestCaptain // true; In Scala, '==' compares the value of the string and not the objects themselves!!!


// EXERCISE
// Take the value of pi, doubles it, then prints it within a string with
// three decimal places of precision on the right.
val doublePi: Double = pi * pi // 9.869604378534024
println (f"Double value of Pi with 3 decimal places precision is $doublePi%.3f") // 9.870
```
---

### Flow Control ###
```
// Flow control in Scala

// if-else is the same as other languages
if (1 > 3) {
  // some statements
} else {
  // some other statements
}

// switch statement uses 'match' as the control logic - not often seen in Scala
val num = 3
num match {
  case 1 => println ("One!")
  case 2 => println ("Two!")
  case 3 => println ("Three")
  case _ => println ("Default case") // _ underscore is the default switch case
}

// for loop
for (x<-1 to 4) { /// note the '<-' sign in the expression
  val squared = x * x
  println (squared)
}

// while loop
var x = 10
while (x >= 0) {
  println (x)
  x -= 1
}

// do-while loop. Scala has this!!!
x = 0
do {
  println (x)
  x += 1
} while (x <= 10)

// ---------------------------------------------------
// Expressions in Scala
// They are denoted as with curly brackets: { some expressions }
// It is important to note that expressions are functions too and they return a value after being evaluated.
// Because they are expressions, they can be passed into functions and control loops too.

{val x = 10; x + 20} // eval to 30
println ({val x = 10; x + 20})

// EXERCISE
// Write some code which will print the 1st 10 digits of the Fibonacci sequence.
// It should print: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
var x: Int = 0
var y: Int = 1
for (i<-1 to 5) {
  print (x + " ")
  print (y + " ")

  x = x + y
  y = y + x
}
```
---

### Functions ###
1. Format of a Scala function is:  
`def` function name `(`parameter name: `type ...) : return type = {` Some lines of code or expressions that evaluate to a return value `}`  
**IMPORTANT**: note the `=` in the function declaration!

```
// Functions in Scala

// format of a function
// def function-name (parameter-name: type ...) : return type = {some expressions here}

// take note of the '=' sign here
// it means assign the function to the expression in {}
def sqIt (x: Int) : Int = {
  // the last value evaluated from the code will be the implied return value of the function
  // we do not need to explicitly return a value using any return keyword
  x * x
}

def cubeIt (x: Int): Int = {
  x * x * x
}

println (sqIt(2))
println (cubeIt(3))

// Functions can also take other functions as parameters

// This function takes in an integer and any function that takes in an int and returns an int
// Denoted by f: Int (integer taken in) => Int (returns an integer)
def transformInt (x: Int, f: Int => Int) : Int = {
  f(x) // refers to any function that can take an integer and returns an integer
}

// Notice we just pass the function-name itself as the parameter
val result = transformInt(2, cubeIt)
println (result)

// Lambda functions == anonymous functions == function literals
// Basically declare a function inlined without giving it a name

transformInt (3, x => x * x * x) // returns 27
// the function literal here is x => x * x * x
// this is legit because the function transformInt expects the 2nd parameter to be any function
// that can take in an integer and return an integer.

// Another legit example
transformInt (10, x => x /2) // returns 5

// Another more complicated legit example; eventually the function literal returns an integer so its legit
transformInt (2, x => {val y = x *2; y * y}) // returns 16
transformInt (2, x => {
  val y = x *2; y * y
}) // same but indented form; returns 16


// EXERCISE
// Strings have a built-in .toUpperCase method. For example, "foo".toUpperCase gives you FOO.
// Write a function that converts a string to upper-case and use that function for a few test strings.
// Then, do the same but using a function literal instead of a separate named function.
"foo".toUpperCase

// Named function that converts a string to upper case.
def toCaps (text: String) : String = {
  text.toUpperCase
}
println (toCaps("this will be all caps in abit"))

// Using the named function
def allCaps (text: String, f: String => String): String = {
  f(text)
}
val result = allCaps("small caps", toCaps)
println (result)

// Using function literal
val anotherResult = allCaps("another small caps", t => t.toUpperCase)
println (anotherResult)
```
---

### Data Structure ###
```
// Data structures in Scala

// Tuples - Immutable lists
// Can be of different data types
// Refer to the individual elements based 1-based index; meaning the 1st element starts from index 1 not 0

val captainStuff = ("Picard", "Enterprise-D", "NCC-something")
println (captainStuff) // (Picard,Enterprise-D,NCC-something)
println (captainStuff._1) // Picard
println (captainStuff._2) // Enterprise-D
println (captainStuff._3) // NCC-something

// key-value pair; note the '->' syntax
val picardsShip = "Picard" -> "Enterprise-D"
println (picardsShip._1) // key - Picard
println (picardsShip._2) // value - Enterprise-D

val aMixedBag = ("Pang", 1974, true) // legit tuple even though they are of different data types

// Lists - like a tuple but it is full blown collection type
// Must of the SAME data type for its elements though
// List like good old collection types are zero-based index; that is starts at index 0
val shipList = List ("Enterprise", "Defiant", "Voyager", "Deep Space Nine")
// shipList: List[String] = List(Enterprise, Defiant, Voyager, Deep Space Nine)

println (shipList(1)) // Defiant
println (shipList.head) // Enterprise
println (shipList.tail) // Tail doesn't give us the last element; it gives us rest of all elements except the head element - weird!

// iterating through a list
// iterate through the ship list, assign each element to 'ship' variable and print it out
for (ship <- shipList) {println (ship)} // Enterprise Defiant Voyager Deep Space Nine

// Use map() to map each element in a list to a function to process it
// Note the inner ( ) for declaring the element parameter ref
val backwardShips = shipList.map ( (ship: String) => {ship.reverse})
for (ship <- backwardShips) {println (ship)} // esirpretnE tnaifeD regayoV eniN ecapS peeD

// Use reduce() to combine all together all the items in a collection using some function
// In this example, reduce() takes 2 numbers as parameters, combine them and use that sum to iterate
// with the next num in line and continue the addition.
val numList = List(1, 2, 3, 4, 5)
val sum = numList.reduce( (x: Int, y: Int) => x + y)
println (sum) // 15

// map() and reduce() are often use to iterate and combine results to get a final output


// filter() for removing stuff that we don't want
val iHate4s = numList.filter( (x: Int) => {x != 4} )
println (iHate4s) // List(1, 2, 3, 5)

// shorthand expression for filter(); the underscore char '_' is placeholder to represent every element
val iHate4Toos = numList.filter(_ != 4)
println (iHate4Toos) // List(1, 2, 3, 5)

// Concatenate lists using '++'
val moreNums = List (6, 7, 8)
val fullNums = numList ++ moreNums

// Other operations
val reversedNums = numList.reverse
val sortedNums = reversedNums.sorted
val duplicatedNums = numList ++ numList
val distinctNums = duplicatedNums.distinct
val maxValue = numList.max
val sumValue = numList.sum
val hasFours = numList.contains (4)

// Maps
val shipMap = Map("Kirk" -> "Enterprise", "Picard"-> "Enterprise-D")
println (shipMap("Kirk")) // get us value using a key
println (shipMap.contains("archer")) // false

// Handling exceptions - try to retrieve by a key and if cannot find return graceful message
// If not handled you will get java.util.NoSuchElementException: key not found: archer
val archerShip = util.Try(shipMap ("archer")) getOrElse "Unknown"
println (archerShip)


// EXERCISE
// Create a list of the numbers 1 to 20; print out the numbers that are evenly divisible by three.
// Use % modula operator, which if divisible by three will give you a remainder of zero.
// Do this by iterating over a list and testing each number. Then do it again using a filter function instead.
val someNumList = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20)
val mappedList = someNumList.map( f = (x: Int) => {
  if (x % 3 == 0) {println (x)} // 3, 6, 9, 12, 15, 18
})
val filteredList = someNumList.filter (_ % 3 == 0)
println (filteredList)
```
---

### Other notes ###
1. We can make our return parameters more explicit and also our more readable by declaring it this way:
```
def parseLine(line:String): (String, String, Float) = {
    val fields = line.split(",")
    val stationID = fields(0)
    val entryType = fields(2)
    val temperature = fields(3).toFloat * 0.1f * (9.0f / 5.0f) + 32.0f
    (stationID, entryType, temperature)
    // declaring the return parameters here though necessary
}
```

2. IMPORTANT !!!  
`map()` has the same number of input rows and output rows after the map operation. It takes every row on the input and apply the expression and produce a corresponding output. That is, one to one input to output.  
`flatMap()` on the other hand, takes an input row and potentially can output 0 to N rows and is not restricted to only 1 output row.

1. A shortcut way to print each item in a list is to use the following: ` wordCounts.foreach(println)` where *wordCounts* is a list or map.

1. Declare a class definition using a one liner, e.g. `case class Person(id:Int, name:String, age:Int, friends:Int)` then during loading of data, call `as[Person]`.

