
# A Dream of a Machine
<sub><sup>(or, why yet another programming language)
</sub></sub>

Stephen Leach, Sept 2025

---

<!--
From the start of my adventures into coding some fifty years ago, I have been captivated by the many and varied programming languages there are - and how each language encapsulates brilliant insights into how to express computation. And yet over the decades of practice, their shortcomings become less and less bearable. Surely it is not beyond the wit of humanity to build a better mousetrap?!

In this week's decodering I share some of the cool ideas that myself and my close colleague Chris Dollin have developed over the years and embodied in various language experiments, including Nutmeg, my latest and most serious attempt at realising our "dream of a machine, perfect in every respect." 

You may laugh, you may cry, but you will be amazed … 

--->


## A dream of a machine, perfect in every respect

> "Not far from here, by a white sun, behind a green star, lived the Steelypips, illustrious, industrious, and they hadn't a care: no spats in their vats, no rules, no schools, no gloom, no evil influence of the moon, no trouble from matter or antimatter - for they had a machine, a dream of a machine, with springs and gears and perfect in every respect. And they lived with it, and on it, and under it, and inside it, for it was all they had - first they saved up all their atoms, then they put them all together, and if one didn't fit, why they chipped at it a bit, and everything was just fine."
<!-- .element: class="r-fit-text" -->

Stanisław Lem, The Cyberiad

---

## Focus on features that are unlikely to be familiar

|     |     |
| --- | --- |
| Qualifiers const, val, var and friends | Exceptions
| Procedures, functions and finesses | Concatenative semantics
| Queries | Machines
| Autoloading | Synchronous coroutines
| Pluggable syntax | Symbols
| Capsules | Elements
| Configuration | Series & Chains
<!-- .element: class="r-fit-text" -->

---

## Qualifiers

It is all about const

<br/><br/>
<sub><sup>
Deep immutability is the central feature of Nutmeg, embracing the essential difference between functional and imperative programming. </sub></sup>

[//]: # (Vertical slide)

### Qualifiers `const`, `val` and `var`

- `var`, an assignable variable that can have any value
    + `var x := List(1, 2, 3)`
- `val`, an unassignable variable that can have any value
    + `val y := List(1, 2, 3)`
- `const`, an unassignable variable that can only have a recursively immutable value
    + `const z := [1, 2, 3]`

[//]: # (Vertical slide)

### Snapshots and Freezing
<div class="r-fit-text">

- All datatypes come in immutable and mutable 'flavours' (shallow)
    + `VarList("abc", "xyz") vs. List("abc", "xyz")`
- You can freeze any mutable datatype and it becomes immutable
    + `my_list := VarList("abc", "xyz").freeze()`
- You can deep-freeze it and it becomes const (deep immutability)
    + `const my_list := VarList("abc", "xyz").freeze(--deep)`
- You can take a snapshot of it (a deeply immutable copy)
```Lua
val x := VarList()
const x1 := x.snapshot()  // x1 is const
x.Add(1, 2, 3)            // but x is not
```
</div>

---

### Procedures

It's all about the finesse

Finesses are procedures that can be safely invoked by functions and preserve all functional programming guarantees.

[//]: # (Vertical slide)

Procedures
```lua
def rev(val L):
    n := len(L)
    for i in [0 ..< n div 2]:
        L[i], L[n-i] <-- L[n-i], L[i]
    end
end
```

Functions
```Lua
[function]
def rev(const L):
   L.tail.rev ++ [L.head]
end
```

[//]: # (Vertical slide)

### Finesse - functional on the outside, crunchy on the inside

```Lua
[finesse]
def rev(x):
    L := x.snapshot(--mutable)
    n := len(L)
    for i in [0 ..< n div 2]:
        L[i], L[n-i] <-- L[n-i], L[i]
    end
    return(L.freeze)
end
```

---

## Query

Iteration is query-solving

```Lua
Q ::= P := E
    | P in E
    | V from E
    | Q where E
    | Q && Q
    | Q // Q
    | Q while E
    | Q until E
```

[//]: # (Vertical slide)

### Expressions vs Queries

An expression is a unit-of-code that is interpreted as meaning "evaluation attempts to generate a value"
```lua
b**2 - 4*a*c
```

A query is a unit-of-code that is interpreted as meaning "evaluation attempts to bind variables to values"
```Lua
discriminant := b**2 - 4*a*c
d := sqrt(discriminant)
a2 := 2 * a
[root1, root2] := [(-b+d)/a2, -b+d)/a2]   ### Pattern matching
```

[//]: # (Vertical slide)

### Loops
```Lua
x in [1, 2, 3, 4, 5] ### A query with 5 possible solutions

### for QUERY do STATEMENTS endfor
for x in L do
    println("A solution is:", x)
endfor
```

[//]: # (Vertical slide)

### Conditionals

query where expression
```Lua
if [x, y, …] := problem.solutions where x != y then
   println("There are at least two different roots")
   println("x: \(x), y: \(y)")
else
   println("No solution pairs exist")
end
```

[//]: # (Vertical slide)

```Lua
if [x, y, …] := problem.solutions where x != y then
###             ^^^^query^^^^^^^^ where ^expression
```

[//]: # (Vertical slide)

### Nested Loops and Early exit

```Lua
for x in range(1, 100) && y in range(1, 100) 
    where x + y == 50 
    while x < y 
    until x > 20 
do
    println("Found pair: (", x, ", ", y, ")")
endfor
```

[//]: # (Vertical slide)

### Only One Solution Needed?
```Lua
if x in range(1, 100) && y in range(1, 100) 
    where x + y == 50 
    while x < y 
    until x > 20 
then
    println("Found pair: (", x, ", ", y, ")")
endif
```

---

## Autoloading

Resources are program

Where do you put your program's data files? How do you find them? How do you load them? Autoloading eliminates all these issues by equating resources with variables.

[//]: # (Vertical slide)

```Lua
def generate_images_dropdown():
    for (k, v) in wallpapers.items():
        MenuItem(k, _ => ShowImage(v))
    endfor.Menu
enddef
```
For directory structure:
```
wallpapers.map
├── bufalo.gif
├── tiger.gif
├── anenome.gif
└── aardvark.gif
this.nutmeg
```
---

## Pluggable Syntax

Bring your own parser

Perhaps the first aspect of any programming language that becomes unbearable is syntax. It is the quirks and cute ideas that first become unbearable. As far back as 1990 we were committed to BYO parsers.

[//]: # (Vertical slide)

### BYO Parser

The first challenge in learning a programming language is the surface syntax.
Often distinctive and quirky e.g. `E1 ? E2 : E3`
Although there is a relationship between the language and its syntax, it is often very weak.
We resolved to make it possible for any programmer to completely replace the initial syntax ("common") by bringing their own parser.

[//]: # (Vertical slide)

### Pre-reqs

The internal abstract syntax tree has to be easy to understand, easy to create and easy to render
UNIX filter: Writeable in any language/toolkit and invoked by a command
AST in XML (Or JSON post-2010)

[//]: # (Vertical slide)

`echo 'x + 1' | nutmeg parse` prints:

```json
{
   "arguments": {
       "body": [
           {
               "kind": "id",
               "name": "x",
               "reftype": "get"
           },
           {
               "kind": "int",
               "value": "1"
           }
       ],
       "kind": "seq"
   },
   "kind": "syscall",
   "name": "+"
}
```
<!-- .element: class="r-fit-text" -->
<!-- Consider a small text rather than a fit -->

[//]: # (Vertical slide)

`echo 'x + 1' | monogram -f xml` prints:

```xml
 <operator name="+" syntax="infix">
   <apply kind="parentheses" separator="undefined">
     <identifier name="f" />
     <arguments>
       <identifier name="x" />
     </arguments>
   </apply>
   <number value="1" />
 </operator>
```

---

## Capsules

It's all about thread safety

The root problem of threads as a mechanism for concurrency is shared access to mutable memory. Nutmeg tasks strictly enforce access to mutable memory so it is thread local.

[//]: # (Vertical slide)

### Capsule Classes

```Lua
[capsule]
class Counter(const initial=0):
    var ^n := initial

    def ^get(const k):
        sleep(ms=10)
        ^n
        ^n <- ^n + k
    end
endclass
```

All parameters and return values are `const` (or "green").

Capsules are free to create mutable store internally.
Internal store is completely isolated.

[//]: # (Vertical slide)

### Tasks and Futures with forcing on-demand

```Lua
### Create a task.
counter := Task(Counter)

### Invoke a method and get a future.
n := counter.Get()
for _ in [0..<2] do println(n, --debug); sleep(ms=20) endfor
<future _>
<future 0>

println(n+100)   ### Implicit forcing
100
```


[//]: # (Vertical slide)

### Task

counter := Task(Counter)
```Lua
### f(% _, x, _ %) is partial application and shorthand for a lambda form:
###
###    fn (tmp1, tmp2) => f( tmp1, x, tmp2 ) endfn
###
let
    new_worker := Worker(% counter %) 
in
    worker1 := Task(new_worker)
    worker2 := Task(new_worker)
end
```
<!-- .element: class="r-fit-text" -->


[//]: # (Vertical slide)


### Functions and finesses counts as capsules

You don't need to do anything special to a function or finesse to run in a Task
```Lua
[capsule]
class EquivalentCapsuleClass(F):
    def ^run(args…):
        F(args…)
    end
end
```

[//]: # (Vertical slide)

![microsft c# async](https://learn.microsoft.com/en-us/dotnet/visual-basic/programming-guide/concepts/async/media/asynchronous-breakfast.png)

[//]: # (Vertical slide)

No async/await
```Lua
Coffee cup = PourCoffee()
Console.WriteLine("Coffee is ready")

eggs := FryEggs(%2%).RunAsTask
hashBrown := FryHashBrowns(%3%).RunAsTask
toast := ToastBread(%4%).RunAsTask

ApplyButter(toast)  ### Implicit await
ApplyJam(toast)
Console.WriteLine("Toast is ready")

Juice oj = PourOJ();
Console.WriteLine("Oj is ready");

eggs.Await() ### Wait if needed (a bit pointless)
Console.WriteLine("Eggs are ready");
hashBrown.Await() ### Equally pointless
Console.WriteLine("Hash browns are ready");
```
<!-- .element: class="r-stretch" -->

[//]: # (Vertical slide)


Whereas in C#:

```C#
Coffee cup = PourCoffee();
Console.WriteLine("Coffee is ready");

Task<Egg> eggsTask = FryEggsAsync(2);
Task<HashBrown> hashBrownTask = FryHashBrownsAsync(3);
Task<Toast> toastTask = ToastBreadAsync(2);

Toast toast = await toastTask;
ApplyButter(toast);
ApplyJam(toast);
Console.WriteLine("Toast is ready")

Juice oj = PourOJ();
Console.WriteLine("Oj is ready");

Egg eggs = await eggsTask;
Console.WriteLine("Eggs are ready");
HashBrown hashBrown = await hashBrownTask;
Console.WriteLine("Hash browns are ready");
```
<!-- .element: class="r-stretch" -->

---
## Configuration

It's just program

Why do configuration files even exist? Because programs need to be parameterised over elaborate datatypes. So why do we specify datatypes in a funny limited syntax when we have a full programming language available?

[//]: # (Vertical slide)

### Safety, Simplicity and Transparency

Sphinxdocs is a very widely used document generation system, the configuration file is `conf.py`, which is written in normal Python
But it can execute arbitrary code, not safe to share or even make available to customers
Might take unbounded time to execute, so ReadTheDocs cannot just trust it
No type-checking
No example generation

[//]: # (Vertical slide)


### `[configuration]`

The `[configuration]` attribute marks a const variable whose value can be overridden by a separate Nutmeg script at load time
```Lua
[configuration]
database_url := "localhost:5432"
```
Hence the compiler knows these variables and their types!

[//]: # (Vertical slide)

### Advantages

- Safe - script execution is resource limited (time and memory) and, being functional, unable to affect the rest of the system
- Natural to exploit auto-loading
- Easy to type-check 
    + `nutmeg config validate my_config.script.nutmeg`
- Easy to create 
    + `nutmeg config generate > example.script.nutmeg`

