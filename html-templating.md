## The problem with HTML templating

HTML is just a __tree__ of elements. For example:

```html
<div>
 <img href="hi.jpg">
 <span>Hello</span>
</div>
```

Sometimes, we want to inject __dynamic values__ into HTML.  This is commonly solved using a templating language like Mustache.

```mustache
<div>
 <img src="{{imageSource}}">
 <span>{{message}}</span>
</div>
```

And sometimes we even need __logic__ in our templates.  Something like Handlebars gives us `if` and `each` statements.

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

But once we start demanding something like function composition to help create/piece together larger templates, we may start feeling weird about inventing our own fully-capable language around HTML.  Perhaps there is a better way.

## Embracing the data

Let's suppose we want to represent HTML as a data structure in some existing programming language instead of grafting one onto HTML.

In computer science, trees are often represented as nested arrays.

```clojure
[:div "Hello"]
```