# delite - creating custom components

## delite background
delite is a new JavaScript library born out of the [Dojo Toolkit Dijit framework](http://dojotoolkit.org/reference-guide/1.10/dijit).
This isn't a replacement per se but a repository to be used as the core building blocks for leveraging current and future standards
in HTML, CSS & JavaScript to build reusable Web Components.

It can be used on its own but more likely used with other projects either from the [ibm-js repositories](https://github.com/ibm-js)
or other repositories.

More information can be found on the [delite website](http://ibm-js.github.io/delite/) explaining the standards this library aims to conform to.

## Tutorial details
In this tutorial you'll learn how to create your own custom elements, learn how to register them, learn how to use templates
and learn how you can bind data. It's a beginner tutorial so we won't be delving too deep into what delite provides (yet!!!)

## Getting started
To quickly get started, we're using [https://github.com/ibm-js/generator-delite-element](https://github.com/ibm-js/generator-delite-element)
to install the required dependencies and create a basic scaffold.

### create the scaffold

We'll let the `generator-delite-element` Yeoman generator do a basic setup for us which will quickly get us going.
(Start off with a simple custom element, no templating etc)

Install the `generator-delite-element` globally

    npm install -g generator-delite-element

And create a new directory (named `title-package`, which will also be our package name) and change directory to it using the command :

    mkdir -p title-package
    cd title-package

Run Yeoman to create our scaffold

    yo delite-element

You'll be prompted to enter the widget package name & the name of the custom widget element, enter the following choices shown in brackets below.

    ? What is the name of your delite widget element package? (title-package)
    ? What do you want to call your delite widget element (must contain a dash)? (title-widget)
    ? Would you like your delite element to be built on a template? (n)
    ? Would you like your delite element to providing theming capabilities? (n)
    ? Will your delite element require string internationalization? (n)
    ? Will your delite element require pointer management? (n)
    ? Do you want to use build version of delite package (instead of source version)? (n)

### A look through what's been generated
Let us look through what Yeoman created, again this is just a boilerplate setup but here are the important parts.

We've created a new package named `title-package` for new widgets that we'll create.

- `./TitleWidget.js` - this is our widget module
- `./TitleWidget/css/TitleWidget.css` - this is our widget css
- `./samples/TitleWidget.html` - this is a sample how to use our new widget

This is the most basic setup for a widget/custom component, you can view the sample generated HTML `./samples/TitleWidget.html`
in a browser to see what's been created.
We'll build upon this example HTML as we progress in the tutorial.

In case there's any confusion, the module name created `TitleWidget` bears no relation to the custom element name i.e. `title-widget`, the module `TitleWidget` registers the custom element tag name `title-widget`.

---

## Creating a custom element
Viewing the `./samples/TitleWidget.html` example HTML we can see we've (partly) created the custom element declaratively in markup via
```html
<title-widget id="element" value="The Title"></title-widget>
```
For those who used the Dojo Toolkit Dijit framework previously, an important conceptual difference in delite is that the widget is the DOM node.
Dijit widgets instead had a property which referenced the DOM node.

###Registering

The `<title-widget>` element doesn't constitute a custom element on its own; it first needs to go through a registration process which is achieved using
the `delite/register` module. This is analogous to the HTML specification for registering custom elements
i.e. `document.registerElement('title-widget');`

If we look at the custom element module `./TitleWidget.js` we see we register the custom element tag via:
```js
return register("title-widget", [HTMLElement, Widget], { .....
```

This is an important concept which sometimes isn't clear at a first glance. You can add any non-standard tag to an HTML page and the browser HTML parser
will not complain; this is because these elements will be defined as a native
[`HTMLUnknownElement`](http://www.whatwg.org/specs/web-apps/current-work/multipage/dom.html#htmlunknownelement).
To create a custom element it must be **upgraded** first; this is what `delite/register` does. `delite/register` supports browsers who natively
support `document.registerElement` and those who don't.

The registration process above using `delite/register`, creates a custom element by registering the tag name `"title-widget"` as the first
argument and then inheriting (prototyping) the `HTMLElement` native element (as well as the `"delite/Widget"` module).

Elements which inherit from `HTMLElement`
using [valid custom element names](http://www.w3.org/TR/2013/WD-custom-elements-20130514/#dfn-custom-element-name) are custom elements.
The most basic requirement for the tag name is it **MUST** contain a dash **(-)**.

###Declarative creation of custom elements
If we view the generated sample HTML `./samples/TitleWidget.html`, we see the following code:

```js
require(["delite/register", "title-package/TitleWidget"], function (register) {
    register.parse();
});
```

Declarative widgets (those created via markup in the page) need to be parsed in order to kick off the lifecycle of creating the widget.

###Programmatic creation of custom elements
The generated example in `./samples/TitleWidget.html` shows the declarative creation of custom elements.
You can do the same thing with programmatic creation:

edit `./samples/TitleWidget.html` i.e.

```js
require(["delite/register", "title-package/TitleWidget"], function (register) {
    register.parse();
});
```

to the following:
```js
require(["delite/register", "title-package/TitleWidget"], function (register, TitleWidget) {
    register.parse();
    var anotherTitleWidget = new TitleWidget({value : 'another custom element title'});
    // note you must call startup() for programmatically created widgets
    anotherTitleWidget.placeAt(document.body, 'last');
    anotherTitleWidget.startup();
});
```

Note that programmatically created widgets should always call `startup()`. A helper function is provided by `delite/Widget` to place it
somewhere in the DOM named `placeAt`
(see the [documentation](https://github.com/ibm-js/delite/blob/master/docs/Widget.md#placement) for it's usage).
We need to also require the module for the custom element i.e. `"title-package/TitleWidget"` because we need to create a new instance and then call its methods.


The above would render: (default image width 460x242)

<img src='./images/programmatic_custom_element.gif'/>


###A look at the widget lifecycle methods for our simple widget
If we look at `"title-package/TitleWidget"` we can see two methods have been created for us, `render` and `refreshRendering`.
`render` is the simplest of [lifecycle](https://github.com/ibm-js/delite/blob/master/docs/Widget.md#lifecycle)
methods we need to create our widget.

#### `render`
We normally wouldn't need to create a `render` method; typically we'd use templates to create our widget UI (which will be explained
later on) but because we aren't currently using a template we need to implement `render` to construct the widget UI for us.
In our sample `render` method we're adding `<span>title</span>` and `<h1></h1>` elements to our widget as well as assigning a property
to the widget named `_h1` i.e. via `this.appendChild(this._h = this.ownerDocument.createElement("h1"));` which we can use to update
it programmatically or declaratively.


#### `refreshRendering`
`refreshRendering` is also a lifecycle method but implemented in `decor/Invalidating`, which `delite/Widget` inherits from.
Its purpose is to observe changes to properties defined on the widget and update the UI. In your web browser developer tools, if
you place a breakpoint in that method and then click the "click to change title" button, you'll see this method is called
(because the button adds inline JavaScript to update the element's value property).

If we wanted to see what the old value was (and also display it to the DOM) we can change this method from

```js
refreshRendering: function (props) {
    // if the value change update the display
    if ("value" in props) {
        this._h.innerHTML = this.value;
    }
}
```

to the following:

```js
refreshRendering: function (props) {
    // if the value change update the display
    if ("value" in props) {
        this._h.innerText = "old= '" + props["value"] + "', new='" + this.value + "'";
    }
}
```

Notice when you first load the page, this method will be called for each widget and if you set a breakpoint in this method when you reload the page
you'll see that the `value` property of our widget is contained in the `props` argument. This is because we're setting the `value` property
on the declaratively written widget to `value="The Title"` and setting the value property on the programmatically written widget to `value : "another custom element title"`.
If you don't set the `value` property of the widget at construction time, the `value` property of our widget is NOT contained in the `props` argument.

Click the 'click to change title button' and the widget will render like:

<img src='./images/custom_element_old_new_props.gif'/>

If you still have a breakpoint set in `refreshRendering` you will see again that the `value` property of our widget is again contained in the `props`
argument.

Also, if you update the value `property` of `./TitleWidget.js` to:

```js
value: "The Title",
```

You'd notice again the `value` property of our widget is NOT contained in the `props` argument. This is because the property value hasn't changed.
The [decor/Invalidating](https://github.com/ibm-js/decor/blob/master/docs/Invalidating.md) documentation explains this behaviour.

###CSS

If we look at the `./TitleWidget.js` custom element module, we see there's a property defined named `baseClass` i.e. `baseClass: "title-widget"`.
This adds a class name to the root node of our custom element (which you can see in the DOM using your debugger tools). Also notice we include
in the `define` the `requirejs-dplugins/css!` plugin to load our css i.e. `"requirejs-dplugins/css!./TitleWidget/css/TitleWidget.css"`.
This plugin is obviously used to load CSS for our custom element. There's nothing much to say here apart from this is how you individually style
your components.

---

##Templates
What we've done so far is obviously a very rudimentary demonstration. We wouldn't expect to programmatically create DOM nodes & this is where
delite comes into it's own. Out of the box, delite supports templates using a built in implementation of [Handlebars](http://handlebarsjs.com/).
We won't need to programmatically create DOM nodes in `render` because creating a template will do all this work for us.

Note there are some limitations using the `delite/handlebars!` plugin in delite for templating, namely it doesn't support iterators or conditionals.
However in many cases this isn't a limiting factor. Support for this will be explained in a later more advanced tutorial when we discuss
[Liaison](https://github.com/ibm-js/liaison). The handlebars template implementation delite uses is primarily focused on performance.

What we've done so far isn't very exciting, so let's try and do something more 'real life', for example a blogging widget.

So we'll create a new delite custom element using Yeoman again (rather than modifying what we have already)

Create a new directory somewhere (named `blogging-package`) and change directory to it using the command :

    mkdir -p blogging-package
    cd blogging-package

Run Yeoman again to create our scaffold

    yo delite-element

You'll again be prompted to enter the widget package name and the name of the custom widget element. Set the following choices shown in brackets below

    ? What is the name of your delite widget element package? (blogging-package)
    ? What do you want to call your delite widget element (must contain a dash)? (blog-post)
    ? Would you like your delite element to be built on a template? (Y)
    ? Would you like your delite element to providing theming capabilities? (N)
    ? Will your delite element require string internationalization? (N)
    ? Will your delite element require pointer management? (N)
    ? Do you want to use build version of delite package (instead of source version)? (N)


This, as shown in the console output, creates:

- `./BlogPost.js` - this is our widget module
- `./BlogPost/css/BlogPost.css` - this is our widget css
- `./BlogPost/BlogPost.html` - this is our widget template
- `./samples/BlogPost.html` - this is a sample how to use our new widget


###Handlebars
If we look at the template Yeoman just created `./BlogPost/BlogPost.html` we can see it's created the following:

```html
<template>
    title:
    <h1>{{value}}</h1>
</template>
```

All templates must be enclosed in a `<template>` element. We can see all the work we did in the `render` lifecycle method becomes much
more simple because now we're just dealing with HTML; actually we don't need to implement a `render` method at all (see the `./TitleWidget.js`
widget module in the previous example to compare).
Instead we have:

```js

define([
	"delite/register",
	"delite/Widget",
	"delite/handlebars!./BlogPost/BlogPost.html",
    "requirejs-dplugins/css!./BlogPost/css/BlogPost.css"
], function (register, Widget, template) {
	return register("blog-post", [HTMLElement, Widget], {
		baseClass: "blog-post",
		value: "",
		template: template
	});
});

```

We just need to include the template using the handlebars plugin i.e.
`"delite/handlebars!./BlogPost/BlogPost.html"` and assign the resolved template to the `template` property of our widget i.e. `template: template`.

####Using handlebars templates
Imagining we need to implement this blogging widget, the widget needs to show the blog title (which we've already done with `{{value}}`, the date it was
published, the author and the content of the blog.

Let's make some changes:
#####Template
Change our template to add new properties for the blog author, when the blog was published and the text of the blog
in `./BlogPost/BlogPost.html`:
```html
<template>
    <article>
        <h3>{{value}}</h3>
        <p class='blogdetails'>Published at <span>{{publishDate}}</span> by <span>{{author}}</span></p>
        <div class='blog'>{{articleContent}}</div>
    </article>
</template>
```
#####Widget
So we've added some new properties, which you see is very easy to do. All we need to do now is map those properties in the widget:

```js
define([
	"delite/register",
	"delite/Widget",
	"delite/handlebars!./BlogPost/BlogPost.html",
    "requirejs-dplugins/css!./BlogPost/css/BlogPost.css"
], function (register, Widget, template) {
	return register("blog-post", [HTMLElement, Widget], {
		baseClass: "blog-post",
		value: "",
		publishDate: new Date().toString(),
		author: "",
		articleContent : "",
		template: template
	});
});

```

Note that I've added a default value for `publishDate`, to make setting the date optional; if unspecified, it will default to today's date.

#####Sample usage
So now if you change the body content of `./samples/BlogPost.html` to the following:

```html
<blog-post id="element" value="A very lazy day" publishDate="Nov 27th 2014" author="My good self" articleContent="Not doing much today because I've ate too much turkey."></blog-post>
<button onclick="element.value='Now sleeping!'; event.target.disabled=true">click to change title</button>
```

And updating the template CSS `./BlogPost/css/BlogPost.css` to make it slightly more interesting too:

```css
/* style for the custom element itself */
.blog-post {
    display: block;
}
.blog-post h3 {
    color: red;
}
.blog-post p.blogdetails span {
    font-weight: bold;
}
.blog-post div.blog {
    padding-left: 20px;
}
```

If you refresh the page you'll see it's becoming something more you'd envisage as a widget we may want to write.

####delite/Container and containerNode
Now is a good time to discuss the functionality provided by [delite/Container](https://github.com/ibm-js/delite/blob/master/docs/Container.md).
Looking at the widget we created, the `articleContent` property of our widget could be seen as a property which might be used to add arbitrary HTML
e.g. paragraph tags, list tags etc etc. If you try and add HTML content to the `articleContent` attribute of our sample `./samples/BlogPost.html`
you'll see that any HTML tags added in the attribute value are escaped and not rendered as HTML; this is expected. Try it out e.g.

```html
<blog-post id="element" value="A very lazy day" publishDate="Nov 27th 2014" author="My good self" articleContent="<b>my HTML tags are escaped</b>"></blog-post>
<button onclick="element.value='Now sleeping!'; event.target.disabled=true">click to change title</button>
```

As explained in the `Container` documentation, it's to be used as a base class for widgets that contain content; therefore it's also useful for our
intentions where we want to add arbitrary HTML for the `articleContent`.

#####Widget
Let's update our widget to use this:

```js
define([
	"delite/register",
	"delite/Widget",
	"delite/Container",
	"delite/handlebars!./BlogPost/BlogPost.html",
    "requirejs-dplugins/css!./BlogPost/css/BlogPost.css"
], function (register, Widget, Container, template) {
	return register("blog-post", [HTMLElement, Container], {
		baseClass: "blog-post",
		value: "",
		publishDate: new Date().toString(),
		author: "",
		template: template
	});
});

```

We've extended our widget using `delite/Container` and removed the `articleContent` property which we don't need anymore.
We also only need to extend `delite/Container` because it extends `delite/Widget`.

#####Widget template
Update ./BlogPost/BlogPost.html` to the following:

```html
<template>
    <article>
        <h3>{{value}}</h3>
        <div class='blog' attach-point="containerNode"></div>
        <p class='blogdetails'>Published at <span>{{publishDate}}</span> by <span>{{author}}</span></p>
    </article>
</template>
```

Notice the `attach-point="containerNode"` attribute. This is a special 'pointer' to a DOM node which is used by `delite/Container`. When you inherit from
`delite/Container`, it adds a property to our widget named `containerNode` and this maps any HTML (or widgets) as children of our widget.

#####Sample usage
Change the body content of `./samples/BlogPost.html` to the following:

```html
<blog-post id="element" value="A very lazy day" publishDate="Nov 27th 2014" author="My good self">
    <h4>So I ate too much</h4>
    <ol>
        <li>Turkey</li>
        <li>Cranberries</li>
        <li>Roast potatoes</li>
        <li>etc etc</li>
    </ol>
</blog-post>
<button onclick="element.value='Now sleeping!'; event.target.disabled=true">click to change title</button>
```

If you refresh your page now you should see something like the following:

<img src='./images/custom_templated_containernode.gif'/>

**TODO: recapture image, looks bad**

You can see that the `attach-point="containerNode"` reference we created will render our declarative content wherever we've placed it in the template.
If you open up your developer tools and in the console enter:

```js
document.getElementById('element').containerNode.innerHTML = "<i>And now we've replaced our containerNode content</i>"
```

You'll see that our widget containerNode `innerHTML` is updated to what we've added.


####Programmatic creation with containerNode
Based on the first example (the `TitleWidget` example), this should be straightforward to you now. If you wanted to programmatically create a widget, if
you update the `./samples/BlogPost.html` from:

```js
require(["delite/register", "blogging-package/BlogPost"], function (register) {
    register.parse();
});
```

to:

```js
require(["delite/register", "blogging-package/BlogPost"], function (register, BlogPost) {
    register.parse();
    var anotherCustomElement = new BlogPost({value : 'The day after', publishDate : 'Nov 28th 2014', author : "My good self"});
    // note you must call startup() for programmatically created widgets
    anotherCustomElement.placeAt(document.body, 'last');
    var containerNodeContent = "<b>boooooo</b> it's the day after, back to work soon :(" +
            "<pre># time to start thinking about code again</pre>";
    anotherCustomElement.containerNode.innerHTML = containerNodeContent;
    anotherCustomElement.startup();
});
```

and refresh the page, you can see how to add HTML to the `containerNode` of our widget.

## Round up
As you've seen, the basics of delite are very easy when building a custom element, keeping in mind we've not really touched on many other capabilities of the project.
We'll expand on this in a later tutorial.
