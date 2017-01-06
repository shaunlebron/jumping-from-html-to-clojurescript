# Using HTML to learn ClojureScript

_or why JSX does not have templating logic_

There is actually a pretty interesting path from HTML templating to
ClojureScript.  So if you're interested in learning ClojureScript, read on!

We will first talk about frustrations with the current state of HTML-templating
in the JS community.  And we will look at an alternative that quickly became
the status quo in the ClojureScript community.

### Extending HTML for templating

The following is a simple HTML example:

```html
<div>
 <img href="hi.jpg">
 <span>Hello</span>
</div>
```

We can inject __dynamic values__ into HTML using a templating language like
Handlebars.

```handlebars
<div>
 <img src="{{imageSource}}" />
 <span>{{message}}</span>
</div>
```

If we need conditional/generative __logic__:

```handlebars
<div>
 {{#if imageSource}}
   <img src="{{imageSource}}" />
 {{/if}}
 {{#each messages}}
   <span>{{.}}</span>
 {{/each}}
</div>
```

When template size and complexity grows, we start demanding more language
features in our templates, like [template
chunks](http://handlebarsjs.com/#helpers) to allow a form of function
composition.

We can continue adding custom syntax to our templates, but we may start feeling
weird about inventing our own fully-capable language around HTML.  Perhaps
there is a better way.

### Embedding HTML in JS

The __React__ team from Facebook recognized the awkwardness of using a language
on top of HTML for sufficiently complex templates, so they tried generating
HTML _inside_ of Javascript to benefit from a full language (among other
reasons related to virtual DOM construction):

```javascript
React.render(
  React.createElement("div", null,
    React.createElement("img", {src: imageSource}),
    React.createElement("span", null, message)));
```

For brevity, they added an __optional sugar__ on top of JS called JSX,
essentially allowing us to use HTML tags inside JS. (JS expressions are also
allowed in the HTML tags by using curly braces.)

```javascript
// this desugars into the previous code example
React.render(
  <div>
    <img src={imageSource} />
    <span>{message}</span>
  </div>);
```

We get function composition without any special syntax since we're in JS:

```javascript
function myImage(imageSource) {
   return <img src={imageSource} />;
}

React.render(
  <div>
    {myImage(imageSource)}
    <span>{message}</span>
  </div>);
```

And we can use Javascript's ternary expressions for conditionals and the native
`map` function for sequence generation:

```javascript
React.render(
  <div>
    {imageSource ? <img src={imageSource} /> : null}
    {messages.map(m => <span>{m}</span>)}
  </div>);
```

Alternatively, we can use [JSX Control
Statements](https://github.com/valtech-au/jsx-control-statements) which
overload the nature of a JSX tag:

```javascript
React.render(
  <div>
    <If condition={imageSource}>
      <img src={imageSource} />
    </If>
    <For each="m" of={messages}>
      <span>{m}</span>
    </For>
  </div>);
```

So, we gained the full power of a real language by _embedding_ HTML inside of
Javascript.  But we find ourselves again adding more syntax, this time for
brevity.  Is there a language that allows us to do this without adding a custom
syntax?

### Redefining our data

Let's go back to our simple HTML example:

```html
<div>
  <img src="hi.jpg" />
  <span>Hello</span>
</div>
```

This is fundamentally just a tree of data, which we can represent with a nested
bulleted list.

----

- __div__
  - __img__ (source=hi.jpg)
  - __span__
    - "Hello"

----

Trees are often represented as nested lists, so let's see it in JSON:

```json
["div",
  ["img", {"src": "hi.jpg"}]
  ["span",
    "Hello"]
]
```

We assume the first element of a list is the tag name.  Second element can be a
map of tag attributes.  Rest of the elements are child tags.

Can we add __logic syntax__ to this somehow?  Going the way of Javascript (e.g.
ternary operator and `map`) suffers from the same problems we saw in the
previous section-- a lack of brevity.  And we're specificially trying to not
invent new syntax.

#### Adding logic

It turns out there already exists a general purpose language that is like JSON,
but with logic as well.  Let's see an example with data first.  Notice the lack
of commas and colons:

```clojure
["div"
  ["img" {"src" "hi.jpg"}]
  ["span"
    "Hello"]
]
```

To see the logic, let's add a conditional around our `img`:

```clojure
["div"
  (if imageSource
    ["img" {"src" imageSource}]
  )
  ["span"
    "Hello"]
]
```

Notice we have inserted the `if` conditional as a list with parens.  You can
probably guess what it does, but you might still be confused.  Why is the `if`
logic represented as a list?

This is actually __very similar to what Handlebars and JSX Control Statements
are doing__; they represent logic as data.  For example, to represent
conditionals, Handlebars has the `{{#if}} {{/if}}` tags, and JSX Control
Statements has the `<If> </If>` tags.  And we already saw how a tag is simply a
list.  Likewise, we're using an `if` list to represent a conditional.

The only difference is that we didn't have to invent the `if` list syntax.  It
was actually here all along in the language, with brevity and all.

#### ClojureScript

This language is called Clojure, and its JavaScript target is called
ClojureScript.  Its syntax is a marriage of Lisp and JSON (sort of).  All paren
lists are interpreted as code, with the first argument being the function name,
and the rest as arguments.  Let's complete our example with a `for` loop
(simplified):

```clojure
["div"
  (if imageSource
    ["img" {"src" imageSource}]
  )
  (for m messages
    ["span" m]
  )
]
```

And if you're wondering how to turn this into an HTML markup string or a React
component tree, there are different `html` functions for doing that.  Here's
how you'd use it.  Remember, a paren list is just a function call:

```clojure
(html
  ["div"
    (if imageSource
      ["img" {"src" imageSource}]
    )
    (for m messages
      ["span" m]
    )
  ]
)
```

Clojure is not built specifically for HTML-templating, of course.  Rather, it
is a general-purpose language that embraces the fundamental nature of code as
evaluated data, and __simple HTML-templating just falls out of that idea__.

The popular templating style we just described is known as [Hiccup](#hiccup-style-templating)
and is the status quo for templating in ClojureScript.

#### In Practice

We left this out for simplicity, but _keywords_ are used instead of strings for
the tag names and attributes (e.g. `:div`, `:img`):

```clojure
[:div
  [:img {:src "hi.jpg"}]
  [:span "Hello"]]
```

And this convention is to never leave closing delimiters on their own line:

```clojure
;; conventional ClojureScript formatting
(html
  [:div
    (if imageSource
      [:img {:src imageSource}])
    (for [m messages]
      [:span m])])
```

It's also worth mentioning that [Hoplon] uses a different syntax with all
elements as idiomatic functions.  `[:div ...]` -> `(div ...)`

```clojure
(div
  (if imageSource
    (img :src imageSource))
  (for [m messages]
    (span m)))
```

[Hoplon]:http://hoplon.io/

Whatever way you represent the elements in ClojureScript, its notions are clear
and consistent with the rest of the language.  Syntax interpretation is a
matter of knowing how a function will interpret its arguments, which can only
take the shape of literal data.

But maybe too many parentheses?

### Why not indent? (e.g. Haml, Slim, Jade)

Nobody likes large amounts of unimportant stuff lying around in code, like
parentheses or closing tags. So it's no wonder that other popular templating
languages have eliminated these redundancies through enforced indentation, and
optional delimiting.  Afterall, indentation is the natural way that we
represent nested lists (as we showed before):

- __div__
  - __if__ imageSource
    - __img__ (source=imageSource)
  - __for__ m in messages
    - __span__
      - m

Deducing structure through indentation makes sense, but we must also deduce the
_types_ of elements to prevent ambiguity:

- html tags: __div__, __img__, __span__
- logic controls: __if__, __for__
- computed values: __imageSource__, __messages__, __m__

Let's see how the popular indentation-based templating languages deal with
ambiguity of element types:

---

__[Haml]__ is used in the Ruby community:

```haml
-# Haml
%div
  - if imageSource
    %img{:src => imageSource}
  - messages.each do |m|
    %span= m
```

- `%` denotes tags.
- Ruby map notation for tag attributes.
- `-` for embedded Ruby logic.
- `=` after a tag denotes Ruby evaluation.

[Haml]:http://haml.info/

---

__[Slim]__ is also used in the Ruby community, and perhaps looks simpler:

```slim
/ Slim
div
  - if imageSource
    img src=imageSource
  - messages.each do |m|
    span = m
```

- Plain words at start of line denote tags.
- Space-delimited `key=value` pairs for tag attributes.
- `-` for embedded Ruby logic.
- `=` after a tag denotes Ruby evaluation

[Slim]:http://slim-lang.com/

---

__[Jade]__ is a dominant version in the JS community:

```jade
// Jade
div
  if imageSource
    img(src=imageSource)
  each m in messages
    span= m
```

- Plain words at start of line denote tags or special logic `if`, `each`, etc.
- `(key=value, ...)` pairs for tag attributes
- `=` after a tag denotes JS evaluation

[Jade]:http://jade-lang.com/

---

Each language has its own syntax for dealing with how elements are evaluated.

TODO: conclusions by comparing syntax of evaluation semantics, and compare
to general purpose semantics of cljs

If we wish to get the benefits of a general purpose language (with consistent
evaluation semantics) and the conciseness of indentation, we can use
ClojureScript with an upcoming editor feature called [parinfer], which uses
indentation to infer structure, and shows proper delimiters without sacrificing
readability for ambiguity.

[parinfer]:https://github.com/shaunlebron/parinfer

---

## Appendix

### ClojureScript Syntax in 15 Minutes

To learn more about the syntax, you can check out [ClojureScript Syntax in 15
minutes](https://github.com/shaunlebron/ClojureScript-Syntax-in-15-minutes).

### Hiccup-style templating

The popular templating pattern used in Clojure is called "Hiccup"-style.  You
can use it through the following libraries:

- [Hiccup] and [Hiccups] both generate __DOM strings__ for Clojure and ClojureScript, respectively
- [Crate] and [Dommy] generate actual __DOM nodes__ instead of strings
- [Sablono] generates React __virtual DOM nodes__
- [Reagent] is a React wrapper that also generates its own __virtual DOM nodes__

[Hiccup]: https://github.com/weavejester/hiccup
[Hiccups]: https://github.com/teropa/hiccups
[Crate]: https://github.com/ibdknox/crate
[Dommy]: https://github.com/Prismatic/dommy
[Sablono]: https://github.com/r0man/sablono
[Reagent]: https://github.com/reagent-project/reagent

