# HTML templating - a problem and solution

Popular HTML templating solutions suffer some awkward, syntactic problems.  We will cover a simplified history, with each section representing an idea that builds on the previous.  And we will end with a solution that we believe to be most useful.  Section topics follow:

- Extending HTML for templating (the Handlebars way)
- Embedding HTML in JS (the React way)
- Simplicity through fundamentals (a new way)

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
 <img src="{{imageSource}}">
 <span>{{message}}</span>
</div>
```

If we need conditional/generative __logic__ in our templates, we can use something like Handlebar syntax:

```handlebars
<div>
 {{#if imageSource}}
   <img src="{{imageSource}}">
 {{/if}}
 {{#each messages}}
   <span>{{.}}</span>
 {{/each}}
</div>
```

Once templates get large, we start demanding more language features in our templates.  For example, Handlebars allows [template chunks](http://handlebarsjs.com/#helpers) to allow a form of function composition.  But the point is that we can continue adding custom syntax to our templates, but we may start feeling weird about inventing our own fully-capable language around HTML.  Perhaps there is a better way.

## Embedding HTML in JS

The __React__ team from Facebook recognized the awkwardness of using a language on top of HTML when creating sufficiently complex templates, so they made a controversial move to generate HTML inside of Javascript (among other reasons related to virtual DOM construction):

```javascript
React.render(
  React.createComponent("div", null,
    React.createComponent("img", {src: imageSource}),
    React.createComponent("span", null, message)));
```

What they gained with the full power of JS, they lost in brevity.  So they added an __optional sugar__ on top of JS called JSX, which allows you to construct a virtual DOM more easily.  It essentially allows us to write HTML inside JS. (JS expressions are also allowed in the HTML tags by using curly braces.)

```javascript
// this desugars into the previous code example
React.render(
  <div>
    <img src={imageSource} />
    <span>{message}</span>
  </div>);
```

We can use Javascript's ternary expressions for conditionals and the native `map` function for sequence generation:

```javascript
React.render(
  <div>
    {imageSource ? <img src={imageSource} /> : null}
    {messages.map(m => <span>{m}</span>)}
  </div>);
```

[JSX Control Statements](https://github.com/valtech-au/jsx-control-statements) offer simpler syntax by overriding the nature of a tag in JSX:

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

## Returning to fundamentals to find simpler syntax

Let's suppose we want to represent HTML as a data structure in some existing programming language instead of grafting one onto HTML.

In computer science, trees are often represented as nested arrays.

```clojure
[:div "Hello"]
```