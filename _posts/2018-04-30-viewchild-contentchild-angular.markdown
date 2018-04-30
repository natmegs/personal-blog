---
layout: post
title:  "ViewChild(ren) and ContentChild(ren) in Angular"
date:   2018-04-30 14:59:30 -0700
permalink: /blog/:title
comments: true
---

ViewChild, ViewChildren, ContentChild, and ContentChildren are all decorators used to get access to elements, directives and/or components in the view DOM. Let’s break down why there are four decorators to do (seemingly) the similar tasks.

###### ViewChild

ViewChild is used to access to first element, component or directive matching the selector from the view DOM. The key here is it will only ever return one matched object even if multiple matching elements exist in the view DOM. If the view DOM changes and a new child matches the selector, the property will be updated. View queries are set before the AfterViewInit lifecycle hook, so if you are going to manipulate the ViewChild property in the component, it would be appropriate to do so from the ngAfterViewInit method.

The ViewChild decorator takes a parameter we can use to specify the selector by:
- Type: Component or Directive class name (<span class="code">@ViewChild(ExampleComponent)</span>)
- String: template variable reference (<span class="code">@ViewChild(‘templateVarName’)</span>)

The decorator also takes an optional second parameter which is an object with a single read property. We use this property if we want to read a different token from the queried elements. This is useful when we want to get the ViewContainerRef from a template in the view rather than the ComponentRef, TemplateRef or ElementRef.

Below are some examples of how we would use these different methods in practice.

<script src="https://gist.github.com/natmegs/7157815465d61e53aba75c214df81ab7.js"></script>

<script src="https://gist.github.com/natmegs/677a1b212d22d751485774fb9b6a0d0a.js"></script>

Since we have two HelloComponents in the template, the logging statement will print out the information for the first HelloComponent (with name Jelly Donut), since the ViewChild will look for the first matching selector.

###### ViewChildren

ViewChildren works the exact same way as ViewChild, except it returns a QueryList of all elements or directives that match the given selector, instead of just the first match. Similarly to ViewChild, if the view DOM changes and new children match the given selector, the property will be updated.

If we have a simple <span class="code">HelloComponent</span> that is used multiple times in a view (ie the same view as the last example), we can target either the first instance or all instances via ViewChild and ViewChildren.

<script src="https://gist.github.com/natmegs/fc44743fd4ba73af68aa1459f2cad7da.js"></script>


###### ContentChild and ContentChildren

ContentChild and ContentChildren work the same way as ViewChild and ViewChildren, but instead of querying the view DOM, they query the content DOM. We use these decorators when we want to access an element added to the component through an ngContent projection. Content queries are set before the AfterContentInit lifecycle hook.

###### QueryList

Since ViewChildren and ContentChildren return a QueryList of a specified type, it’s helpful to know some of the properties that exist on the QueryList type. The QueryList is an unmodifiable, iterable list of items that Angular keeps up to date whenever state changes (ie a fancy array). QueryList contains a changes Observable that can be subscribed to to be notified of any changes to the QueryList.

Some properties that can be accessed on a QueryList<T>:
- dirty: boolean
- changes: Observable<any>
- length: number
- first: T
- last: T

And some methods available on a QueryList<T>:
- toArray(): T[]
- toString(): string
- reset(res: Array<T | any[]>): void
- notifyOnChanges(): void
- setDirty(): void
- destroy(): void

QueryList also implements its own (or similar) versions of many array methods.
