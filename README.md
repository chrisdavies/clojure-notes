# CLJ for JavaScripters

The core idea of Clojure is to compose programs by using a small set of data types and a broad range of general functions which can be applied to those data types. This is called data-oriented programming. The idea is that most (all?) programs are simply a function of data transformations over time.

If you can wrap your head around the basic types and some general rules of thumb for exploration, you can pick the language up very quickly.

## In this document

- Basic types
- Collections
- Functions
- Essential collection functions
- Exploring via the REPL
- Conditionals
- JavaScript interop
- Namespaces
- Misc
- Resources

## Types

There are only a few basic types you really need to care about

```clj
; Ints
10

; Floats
0.5

; Keywords, which are like symbols in Ruby, essentially
; represents a name in a less expensive way than a string
:foo

; Strings are double-quoted
"hello, world!"

; Regular expressions
#"[0-9]"

; Characters
\h ; The "h" character (= \h (nth "hello" 0)) ; => true

; nil is how Clojure expresses null, and unlike in JS,
; many (most?) functions accept nil and treat it like
; the empty type (which is handy, as will be shown)
nil

; Booleans
true
false

; Symbols are a way of representing something that would otherwise
; be evaluated as code. Generally useful in macros, and in defining
; lists that would otherwise be evaluated as functions. Symbols begin
; with a '
'(+ 1 2 3) ; Produces a list with the plus symbol as the first element
(+ 1 2 3)  ; Produces the value 6

```

## Collections

Nearly all of your Clojure code will be simple manipulation of raw data. That data may be in the form of the basic types described earlier, or it will be one of the following collections. There is a real strength to programming with only a handful of basic types. You get much greater reuse out of functions than you otherwise would.

Some of these collection types have rough analogs in JavaScript, but it should be noted that these are *very* rough analogs.

Note with all of these, the values are space-delimited, so commas are optional and are
usually avoided. Newlines are also optional, though they are a nice way to break up
the keys in a map, as shown below.

https://clojure.org/reference/data_structures

```clj

; Vectors, conceptually similar to arrays, but with efficient immutable
; manipulation and operations. Using an index to look up a value is a
; fast operation-- close to O(1)
[1 2 3 4 5]
[:not "all the same types" 12 true]

; Lists, linked lists, generally avoided in favor of vectors because
; unlike most languages, Clojure's vectors are efficiently manipulated
; Using an index to look up a value is O(N).
'(1 2 3 4)

; Maps, conceptually similar to a hash-table or dictionary, but with
; efficient immutable operations. Generally, it's nice to use keywords
; as the map keys, but you can use strings or nearly any type.
{:name "Chris"
 :age 37}

; Or with string keys
{"name" "Chris"
 "age" 37}

; Or with mixed keys (which should be avoided, probably)
{:name "Chris"
 "age" 37}


; Sets are like maps, except with no values, only keys. It is a way to represent
; a distinct collection of values.
#{1 2 3 4 :a "can be mixed types, too"}


```

## Functions

Clojure has a number of ways to define and invoke functions. Note that as in Ruby, return
values are implicit. The value produced by the last expression is what is returned.

```clj

; Export a function from the current namespace (think of namespaces like modules
; / files).
; JS:
; export function add(x, y) { return x + y; }
(defn add[x y]
  (+ x y))

; Private function (local to the current namespace / file)
; JS:
; function subtract(x, y) { return x - y; }
(defn- subtract[x y]
  (- x y))

; You can create anonymous functions, mkay?
; JS:
; (x, y) => x + y;
(fn [x y] (+ x y))

; Functions can have documentation strings
(defn plusPlus
  "Adds one to x, exactly the same as inc"
  [x]
  (inc x))

; And for really simple functions, there is a shorthand
; JS:
; (x, y) => x + y;
#(+ %1 %2)

; In the case of anonymous functions, I'd only use them for 0-2 argument functions.
; If a function only has one argument, you don't need to refer to it by number, but
; can just use %, which is always the same as %1
#(+ 1 %)

; Functions are almost always invoked via an open parenthesis, followed by the
; function name, followed by the function arguments, so:
(+ 1 2 3 4)
(add 8 9)
(subtract 7 2)
(add 2 (subtract 8 4))

; Rest params
; JS:
; function sumAll(a, b, ...coll) {
;   return `A=${a} B=${b} and coll is a list... (${coll.join(',')})`;
; }
(defn sum-all [a b & coll]
  (str "A=" a " B=" b " and coll is a list... " coll))

(sum-all :hi :there :you :guys) ; => "A=:hi B=:there and coll is a list... (:you :guys)"

; Destructuring maps
; JS:
; function hello({name, age}) {
;   return `Hi, ${name}, you are ${age} years old`;
; }
(defn hello [{:keys [name age]}]
  (str "Hi, " name ", you are " age " years old"))

(hello {:name "Joe" :age 3}) ; => "Hi, Joe, you are 3 years old"

; Destructuring arrays
; JS:
; function baz([a, b, ...c]) {
;   return a + b + c.reduce((a, b) => a + b);
; }
(defn baz [a b & c]
  (+ a b (reduce + c)))

(baz 1 2 3 4) ; => 10
(baz 1 2)     ; => 3
(baz 1)       ; => error!!! All non-rest arguments are required

; You can combine destructuring techniques, but it gets messy pretty fast, so don't

; Functions can have multiple signatures
(defn add
  ([x] (+ 1 x))
  ([x y] (+ x y)))

(add 2)   ; => 3
(add 2 9) ; => 11

; Recursion is supported, but tail-optimized recursion needs to be done with the recurse
; keyword or you'll get a stack-overflow if the recursion goes too deep.

; To define local constants (not really variables), use the let macro, like so:
(defn foo [m]
  (let [name (-> (:name m) (.toLowerCase))
        age (:age m)]
    (str name " is " age " years old")))

```

You've probably also heard of [macros](https://www.braveclojure.com/writing-macros/). You can think of them as little code-generators built into Clojure. They can be used to greatly reduce boilerplate in certain situations (see the `->` macro, for a good example of this). From a consumption point of view, they are almost indistinguishable from functions. They are, however, more complex to write, so steer clear of them until you have a better handle on the language.


## Handy collection functions and behaviors

Clojure ships with a bunch of core functions that operate on collections. Many of
these functions work on any type of collection (lists, vectors, maps, sets, etc). And
they compose/combine to produce quite elegant and concise code.

- https://clojuredocs.org/clojure.core
- https://clojure.github.io/clojure/clojure.core-api.html

```clj
; Collections are deeply compared, but this is done fairly efficiently, caching
; the collection value in a hash which makes subsequent comparisons fast.
(= '(1 2 3) [1 2 3])   ; true
(= '(1 2 "3") [1 2 3]) ; false

; When dealing with a map or a set, you can look up a key in several ways.
; Let's create a sample map:
(def state
  {:name "Chris"
   :age 37
   :some-numbers [0 1 2 3 4 5]})

; Let's look up name
; keywords can be used as accessor functions
(:name state)           ; => "Chris"

; If the key isn't found, nil is returned
(:foo state)            ; => nil

; Adding a third argument gives it a default value if the key isn't found
(:foo state "bar")      ; => "bar"

; Youc an also look up via the "get" function
(get state :name)       ; => "Chris"
(get state :foo "bar")  ; => "bar"

; As shown above, keywords can be used as accessor functions, but strings
; cannot, so this would be an error ("name" state), but maps and sets can
; also be used as a function, so this is valid:
(state "name")          ; => nil
(state "name" "bar")    ; => "bar"

; If you have a deep map like, you can use get-in to dig down into it
(def shtuff
  {:name "Dabo"
   :age 45
   :employer {:name "Clemson"
              :role "Coach"}})

(get-in shtuff [:employer :name]) ; => "Clemson"

; Associate a new key/value with a map, can also do multiple associations at once.
; Note that this (as with nearly all functions) does not mutate the map, but returns
; a new one with the specified key/values.
(assoc shtuff :name "Joe") ; => {:name "Joe", :age 45, ...etc...}
(assoc {} :f-name "Joe" :l-name "Shmo") ; => {:f-name "Joe" :l-name "Shmo"}
(assoc-in shtuff [:employer :role] "Head Coach")
; => {:name "Dabo", :age 45, :employer {:name "Clemson", :role "Head Coach"}}

; You can update a value, which runs a function on it and returns the result
(update {:x 1} :x inc) ; => {:x 2}
(update-in {:foo {:bar 1}} [:foo :bar] dec) ; => {:foo {:bar 0}}


; Map/reduce/filter, etc are all supported, and work on maps, sets, lists, etc.
; As in many cases, Clojure is a good bit more terse than JavaScript.

; [0, 1, 2, 3].map(x => x + 1)
(map inc [0 1 2 3]) ; => [1 2 3 4]

; [1, 2, 3, 4].reduce((acc, x) => { acc += x; return acc; });
(reduce + [1 2 3 4]) ; => 10

; [1, 2, 3, 4].filter(x => x % 2 === 1)
(filter odd? [1 2 3 4]) ; => '(1 3 5)

; Check if a set contains a value
(contains? #{1 2 3} 3) ; => true

; Add an item to a list, vector, or set, appends or prepends based on what the
; most efficient operation would be...
(conj #{1 2} 3) ; => #{1 2 3}
(conj [1 2] 3)  ; => [1 2 3]
(conj '(1 2) 3) ; => '(3 1 2)

; Look up an value in a vector
(["hello" "world"] 1)     ; => "world"
(["hello" "world"] 2)     ; => throws java.lang.IndexOutOfBoundsException:
(nth ["hello" "world"] 1) ; => "world"

; Get the length of any collection
(count [:a :b :c]) ; => 3
(count {:a :b :c :d}) ; => 2

```

## Exploring

Clojure ships with a fair number of functions, which means you're going to run across
code that you don't understand. There are a number of ways to solve this problem.

First, start a repl, and explore the code a piece at a time. I find Clojure's repl to be
more useful than Ruby's thanks to the functional nature of Clojure. You don't need to set
up complex object state in order to get useful info out of the repl.

Let's take an example of some dense code.

```clj
(defn nth-row [n]
  (nth (iterate #(concat [1] (map + % (rest %)) [1]) [1]) (dec n)))
```

Well, in this case, you can just paste that into the REPL and run it to see what it does:

```clj
(nth-row 5) ; => (1 4 6 4 1)
```

But there are new terms we haven't seen before. You can Google them, of course, but sometimes,
that doesn't actually yield great results. Here are a few useful ways to explore in the repl:

```clj
; Doc is a repl function that shows the documentation for a function
(doc rest)

; Source shows the source code for a specific function
(source rest)

; Dir shows all public vars in a namespace (e.g. all of the functions and consts),
; so for example, the following lists all of the vars defined in Clojure's core namespace,
; which is automatically available in any Clojure code you write.
(dir clojure.core)
```

## Threading operator

Understanding deeply nested function calls is difficult in any language. Good languages provide
various ways to express the same ideas without the need for deep nesting.

See: `->` `->>` `some->` `some->>` `as->` `cond->` `cond->>`

```clj
; This expression
(reduce + (map dec (take 3 [1 2 3 4])))

; Is made clearer using the threading operator, which passes the result of each line
; as the last position in the next line
(->> [1 2 3 4 5]
     (take 3)
     (map dec)
     (reduce +))

```

## Comparisons and Conditionals

```clj
; Comparisons take any number of args
; JS:
; ((1 < 2) && (2 < 3) && (3 < 4))
(< 1 2 3 4)  ; => true

; JS:
; ((5 > 4) && (4 > -3) && (-3 > 7))
(> 5 4 -3 7) ; => false

; JS:
; ((2 > 1) && ('yes' === 'yes'))
(and (> 2 1) (= :yes :yes)) ; => true

; JS:
; ((2 < 1) || ('yes' === 'yes'))
(or (< 2 1) (= :yes :yes))  ; => true

; JS:
; (x > y ? "yup" : "nope")
(if (> x y) "yup" "nope")

; Only nil and false are falsey
(if nil :yes :no)    ; => :no
(if false :yes :no)  ; => :no
(if 0 :yes :no)      ; => :yes
(if "" :yes :no)     ; => :yes

; JS:
; (x > y ? "yup" : null)
(when (> x y) "yup")

; cond is a good way conditionally run functions over a value, and return
; the result of the first truthy match.
(defn grade-letter [grade]
  (cond
    (>= grade 90) "A"
    (>= grade 80) "B"
    (>= grade 70) "C"
    (>= grade 60) "D"
    :else "F"))

; condp takes a predecate and then switches on the resulting comparison
(defn num-to-text [n]
  (condp = n
    1 "one"
    2 "two"
    3 "three"
    (str "unexpected value, \"" value \")))

; Case is a lot like a switch statement, but often could be written as a simple
; hash lookup instead.
(defn hex-char-to-num [c]
  (case c
    \0 0
    \1 1
    ; etc, etc, etc
    \E 14
    \F 15))
```

## JavaScript interop

Since we're currently only evaluating ClojureScript usage, we should have a peek at how to interop with JavaScript!

```cljs
; The #js prefix converts maps/vector literals to JS-native objects/arrays
; JS:
; {"hello": "World"}
#js{:hello "World"}  ; => {"hello": "World"}
#js["hello" "world"] ; => ["hello" "world"]

; Converting any structure to its JavaScript equivalent can be done with the
; clj->js function.
(clj->js {:hello "World"})

; This can be reversed like so, though if translating huge objects, there is a 
; library called "transit" which is more efficient.
; Note, by default, it doesn't keywordize the keys of the map being translated.
(js->clj #js{:hello "world"}) ; => {"hello" "world"}
(js->clj #js{:hello "world"} :keywordize-keys true) ; => {:hello "world"}

; Calling a JavaScript function is done using the js/ prefix, like so:
; JS:
; alert("Hello, World!");
(js/alert "Hello, World!")

; JS:
; console.log("Hello, world!");
(js/console.log "Hello, world!")

; You can create a new thingy
; JS:
; new Date()
(new js/Date)

; You can read properties of thingies
; JS:
; "hello".length
(.-length "hello")

; I often chain this with the -> macro when doing event handlers
; JS:
; function targetName(e) { return e.target.name; }
(defn target-name [e]
  (-> e .-target .-name))

; Sometimes, you need to look a value up by string
; JS:
; myObj["hello"]
(aget my-obj "hello")

; You can set values to thingies
; JS:
; function setVal(el, val) { el.value = val; }
(defn set-val [el val]
  (set! (.-value el) val))

; Sometimes, you need to set values by accessing the property via strings...
; JS:
; myObj["hello"] = "world"
(aset my-obj "hello" "world")

```

## Namespaces

Clojure has a thing called namespaces, which is essentially like an ES6 module. You have
one namespace per file. I think this is enforced, but if not, pretend it is.

So far, we've lived entirely in the realm of the `clojure.core` namespace, but every jar (library) that you use will define its own namespaces, and every file you write will have its own namespace.

```cljs
; A typical file will start something like this
(ns my-awesome-project.my-file-name)

; Very often, you'll want to pull in some functions from a
; namespace other than core. You have to do this explicitly, as in
; ES6, and unlike Rails (thank God!).
; Here, we require the clojure.string namespace and refer to it as str, so
; we could call its functions like this: (str/blank? "") ; => true
; JS:
; import str from 'example/string';
(ns my-awesome-project.my-file-name
  (:require [clojure.string :as str]))

; You can require as many things as you want
(ns my-awesome-project.my-file-name
  (:require [re-frame.core :as rf]
            [clojure.string :as str]))

; Sometimes, you want to pull in functions without prefixing them with the namespace
; JS:
; import {isBlank} from 'example/string';
(ns my-awesome-project.my-file-name
  (:require [clojure.string :refer [blank?]]))

(blank? "") ; => true

; You can both alias and refer at once
; JS:
; import str, {isBlank} from 'example/string';
(ns my-awesome-project.my-file-name
  (:require [clojure.string :as str :refer [blank?]]))

; While you can do the above in the REPL, doing so will change your namespace, which
; is not often what you want to do. So instead, call the require function directly.
; Note the slightly different syntax. (You need to pass it symbols, mkay?)
(require '[clojure.string :as str])

```

More:

- https://clojuredocs.org/clojure.core/require

## Misc

- `kebab-casing` is preferred for all the things (don't `camelCase`)
- Functions that return a bool should end with `?`

```clj
; Functions that return a bool generally end with ?
(defn old? [user]
  (-> user :age (> 90)))

(old? {:name "Sam" :age 100}) ; => true

```

- If you're about to write a data-manipulation function, have a quick scan of the core namespace to see if one already exists
- You probably won't need to create ES6-like clasess in ClojureScript, but if inter-operating with some JS libs, you might
  - It's not great. [See here for details](./cljs-and-es6-classes.md)

## Resources

By now, hopefully it's obvious that Clojure's syntax is simple. Almost everything you do is either a function or macro combining lists, vectors, maps, sets, or the basic types. As such, it's a highly explorable. Almost all syntactic weirdness can be looked up on the repl via `doc` or `source`. It's also a concise language with (usually) sane defaults.

Cheatsheet

- http://cljs.info/cheatsheet/

Books

- http://www.braveclojure.com/
- https://aphyr.com/posts/301-clojure-from-the-ground-up-welcome

Reading through the core library documentation, and playing around in the repl is a good way
to learn the building blocks of the language.

- https://clojuredocs.org/clojure.core
