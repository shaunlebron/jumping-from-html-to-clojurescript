# HTML templating - a problem and solution

### Extending HTML for templating

The following is a simple HTML example:

```html
<div>
 <img href="hi.jpg">
 <span>Hello</span>
</div>
```

Sometimes, we want to inject __dynamic values__ into HTML.  This is commonly solved using a templating language like Mustache.

```handlebars
<div>
 <img src="{{imageSource}}" />
 <span>{{message}}</span>
</div>
```

If we need conditional/generative __logic__ in our templates, we can use something like Handlebar syntax:

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

Once templates get large, we start demanding more language features in our templates.  For example, Handlebars allows [template chunks](http://handlebarsjs.com/#helpers) to allow a form of function composition.  But the point is that we can continue adding custom syntax to our templates, but we may start feeling weird about inventing our own fully-capable language around HTML.  Perhaps there is a better way.

### Embedding HTML in JS

The __React__ team from Facebook recognized the awkwardness of using a language on top of HTML when creating sufficiently complex templates, so they made a controversial move to generate HTML inside of Javascript (among other reasons related to virtual DOM construction):

```javascript
React.render(
  React.createComponent("div", null,
    React.createComponent("img", {src: imageSource}),
    React.createComponent("span", null, message)));
```

What they gained with the full power of JS, they lost in brevity.  So they added an __optional sugar__ on top of JS called JSX, essentially allowing us to write HTML inside JS. (JS expressions are also allowed in the HTML tags by using curly braces.)

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

And we can use Javascript's ternary expressions for conditionals and the native `map` function for sequence generation:

```javascript
React.render(
  <div>
    {imageSource ? <img src={imageSource} /> : null}
    {messages.map(m => <span>{m}</span>)}
  </div>);
```

[JSX Control Statements](https://github.com/valtech-au/jsx-control-statements) offer simpler syntax for the previous example by overriding the nature of a tag in JSX:

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

__To review__, we gained the full power of a real language by _embedding_ HTML inside of Javascript.  But yet again, we are forced to add additional syntax to regain the brevity of traditional templates.  Perhaps there is a better way.

### From first principles to simplicity

Let's go back to our simple HTML example:

```html
<div>
  <img src="hi.jpg" />
  <span>Hello</span>
</div>
```

This is fundamentally just a tree of data, which we can represent with a nested bulleted list.

----

- __div__
  - __img__ (source=hi.jpg)
  - __span__
    - "Hello"

----

Incidentally, trees are often represented as nested lists in computer science, so let's imagine what it would look like as literal lists in JSON form:

```json
["div",
  ["img", {"src": "hi.jpg"}]
  ["span",
    "Hello"]
]
```

The first element of every list is the name of the tag.  The second element can be a map of attributes for that tag.  The rest of the elements are child tags.

Can we add logic to it somehow?  Using Javascript suffers from the same problems we saw in the previous section-- the awkwardness of the ternary operator and `map` function.  Is there a better way?

----

#### JSON+logic = ?

Actually, the simplest thing we can do is to use a language that is __sort of like JSON__.  Notice the only difference is the lack of commas and colons:

```clojure
["div"
  ["img" {"src" "hi.jpg"}]
  ["span"
    "Hello"]
]
```

This language actually allows us to embed logic in our data.  In fact, logic is represented as a new kind of list:

```clojure
["div"
  (if imageSource
    ["img" {"src" imageSource}]
  )
  ["span"
    "Hello"]
]
```

Notice we have inserted the `if` conditional as a list with parens.  You can probably guess what it does, but you might be confused at why the distinction between logic and data has been blurred.  Well, this is already what Handlebars and JSX Control Statements are doing; they create custom data tags for logic.  Handlebars adds the `{{#each}} {{/each}}` tags, and JSX Control Statements creates adds the `<For> </For>` tags; logic as data.  Likewise, we're using an `if` list.

This language is called Clojure.  Its syntax is a marriage of Lisp and JSON (sort of).  All paren lists are interpreted as code, with the first argument being the function name, and the rest as arguments.  Let's complete our example with a `for` loop:

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
