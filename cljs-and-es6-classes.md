# ES6 cClasses from ClojureScript

Defining ES6 classes in ClojureScript is a pain. Honestly, I don't blame ClojureScript. I blame ES for introducing this feature in the first place. *rant*

Mkay, that said, here's how I create ES6-compatible class definitions (as essentially required by React/Preact these days). That said, if using React, at least, you should just go ahead and use Reagent or re-frame.

```js
// Let's convert this abomination into ClojureScript
class Foo extends Component {
  constructor({name}) {
    this.name = name;
  }

  render() {
    return "Hello, " + this.name;
  }
}
```

The previous JavaScript can be done in ClojureScript as follows. It's a lot more code, but:

- You don't do it often
- If you do have to do it more often than you'd like, it's easy to abstract
- You should contact the author of whatever lib you're using and request them to stop requiring OOP nonsense, or at least to simplify it! :)

```cljs
; Define our constructor function
(defn foo [name]
  (this-as me
    (.call Component me) ; Muy importante!!!
    (set! (.-name me) name)))

; Define our render function, mkay?
(defn- render []
  (this-as me
    (str "Hello, " (.-name me))))

; Associate a copy of the Component's prototype with our constructor
(->> (.-prototype Component)
     (.create js/Object)
     (set! (.-prototype foo)))

; Set the prototype's constructor to be foo, not Component
(set! (-> foo .-prototype .-constructor) foo)

; Set the prototype's render method
(set! (-> foo .-prototype .-render) render)
```
