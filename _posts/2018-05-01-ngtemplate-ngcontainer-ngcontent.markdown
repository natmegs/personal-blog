---
layout: post
title:  "ng-template, ng-container, and ng-content in Angular"
date:   2018-04-30 14:59:30 -0700
permalink: /blog/:title
comments: true
---

<span class="code">`<ng-template>`</span>, <span class="code">`<ng-container>`</span> and <span class="code">`<ng-content>`</span> are all Angular elements used for rendering HTML, but what do they actually do? Why do we need three, and why can’t I just use regular HTML elements like divs? I’ve been burned and confused by these elements in the past so I endeavoured to try to understand the reasoning behind them.

###### `<ng-template>`

Ever seen <span class="code">`<ng-template>`</span>s in a code base and thought they looked cool, so you put some HTML into one to try to look cool too, only to be completely baffled by why it wasn’t showing up on the screen or in the DOM? Only me? Cool...

The ng-template element is conceptually similar to the HTML <span class="code">`<template>`</span> element. From [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template):

“The HTML `<template>` element is a mechanism for holding client-side content that is not to be rendered when a page is loaded but may subsequently be instantiated during runtime using Javascript.”

So why would we ever want to write HTML that’s not rendered on page load? A really good use case is when creating reusable templates that can be used as needed and instantiated dynamically based on data or other input retrieved at runtime. 

The same concepts apply to Angular’s <span class="code">`<ng-template>`</span>. If you put some HTML inside of an <span class="code">`<ng-template>`</span> tag, it not only won’t be on the screen, but it won’t be in the DOM either. Angular will replace the <span class="code">`<ng-template>`</span> tag and its contents with a comment. The key is that <span class="code">`<ng-template>`</span>s will only be displayed if used in partnership with a structural directive. We need something to tell Angular that we want to use this template. This can be easily accomplished using template variables.

<script src="https://gist.github.com/natmegs/11d06af1ad6f85960eb608ed03c3ea5a.js"></script>

So just putting a structural directive on an <span class="code">`<ng-template>`</span> isn’t enough to tell Angular we want to use the template, but we can use other elements to specify when they want to show our <span class="code">`<ng-template>`</span> by attaching our <span class="code">#template</span> local template variable.

A rendered <span class="code">`<ng-template>`</span> doesn’t itself get turned into a DOM element, only the contents are rendered to the DOM. So when the above example is rendered, the <span class="code">`<ng-template>`</span> wrapper element does not get turned into a DOM element in the same way a container div would; only the content text ends up on the DOM.



###### `<ng-container>`

<span class="code">`<ng-container>`</span> is an Angular grouping element that is similar to <span class="code">`<ng-container>`</span> in that it doesn’t represent a DOM element. The difference is that it will always be rendered, whereas an <span class="code">`<ng-template>`</span> will only be rendered if it is explicitly requested. <span class="code">`<ng-container>`</span>s are useful anywhere you need an extra container for some template elements, but don’t want to (or can’t) create a container such as a div to hold them with due to syntax or style constraints.

For example, it is not allowed in Angular to put two structural directives on the same element. If you needed to loop through an array and display a <span class="code">`<tr>`</span> for each element, but only if a different condition was met, you may want to put both an *ngFor and *ngIf on the <span class="code">`<tr>`</span> element. Angular does not allow this, however, and wrapping the <span class="code">`<tr>`</span> in a <span class="code">`<div>`</span> to hold one of the structural directives is not valid HTML. The utility of <span class="code">`<ng-container>`</span> shines here, where we can use the <span class="code">`<ng-container>`</span> to hold a structural directive and contain the <span class="code">`<tr><`/span> without breaking the HTML layout.

<script src="https://gist.github.com/natmegs/eeb671314212e1b141aafdb120d5cb2c.js"></script>

And despite the fact that the <span class="code">`<ng-container>`</span> will always be rendered, even if the item doesn’t meet the inner condition, no extra elements will be added to the DOM since the <span class="code">`<ng-container>`</span> alone doesn’t represent a DOM element.

###### ngTemplateOutlet

We can bring together the <span class="code">`<ng-container>`</span> and <span class="code">`<ng-template>`</span> elements using the <span class="code">*ngTemplateOutlet</span> structural directive. This directive allows us to instantiate a <span class="code">`<ng-template>`</span> anywhere on the page.

<script src="https://gist.github.com/natmegs/01df668e02ea3ecc3d937ee3ea92e478.js"></script>

Placing this directive on an <span class="code">`<ng-container>`</span> element allows us to place an instance of a template wherever we need it. <span class="code">*ngTemplateOutlet</span> can also take a context object as input, which becomes available for binding by the local template let declarations.

<script src="https://gist.github.com/natmegs/c1b52a1d2084b9cb299df6262159ae5d.js"></script>

<script src="https://gist.github.com/natmegs/74c1999693abbfe987c3fea4b9c23680.js"></script>


###### `<ng-content>`

<span class="code">`<ng-template>`</span>, <span class="code">`<ng-container>`</span>, and<span class="code">ngTemplateOutlet</span> are all used (sometimes in combination) to instantiate and place HTML templates in a view, and <span class="code">`<ng-content>`</span> serves a similar role in a slightly different way. <span class="code">`<ng-content>`</span> is used for projecting content into components. Any component that will accept other components between its opening and closing tags will use an <span class="code">`<ng-content>`</span> to indicate where that content should be placed.

If we create a SomeComponent component, we can indicate that it can take projected content by using the <span class="code">`<ng-content>`</span> tag to indicate where the projected content should go.

<script src="https://gist.github.com/natmegs/b378834479a4b4db7a063e6943032dbe.js"></script>

And then wherever we use SomeComponent, we can insert any template elements between its opening and closing tags as the projected content.

<script src="https://gist.github.com/natmegs/204622e774de4e9e3d332ba91888cf37.js"></script>

