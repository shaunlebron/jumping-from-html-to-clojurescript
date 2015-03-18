## The evolution of HTML templating

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

Once templates get large, we start demanding more language features in our templates (e.g. function composition).  We can continue adding custom syntax to our templates, but we may start feeling weird about inventing our own fully-capable language around HTML.  Perhaps there is a better way.

## Embracing a real language

The __React__ team from Facebook recognized the awkwardness of using a language on top of HTML when creating sufficiently complex templates, so they made a controversial move to generate HTML inside of Javascript.

```javascript
React.render(
  React.createComponent("div", null,
    React.createComponent("img", {src: imageSource}),
    React.createComponent("span", null, message)));
```

What they gained with the full power of JS, they lost in brevity.  So they added an __optional sugar__ on top of JS called JSX, which allows you to use HTML with Mustache-like expressions inside them.

```javascript
React.render(
  <div>
    <img src={imageSource} />
    <span>{message}</span>
  </div>);
```

There is currently a 50/50 mindshare among using JSX versus the usual JS calls.


## Could it be better?

Let's suppose we want to represent HTML as a data structure in some existing programming language instead of grafting one onto HTML.

In computer science, trees are often represented as nested arrays.

```clojure
[:div "Hello"]
```