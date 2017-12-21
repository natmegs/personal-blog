---
layout: post
title:  "Dynamic Components in Angular"
date:   2017-12-20 14:59:30 -0700
permalink: /blog/:title
comments: true
---

Angular templates are often composed of many components, and these components are commonly inserted directly into a template using their selector tag. There are cases, however, when it would be useful to add components dynamically at runtime. Why? Modals and tooltips are two common examples of uses cases that can fit this description. In these situations we may want to programmatically create new components.

The basic flow goes something like this. We have a component, we’ll call it HostComponent. In HostComponent, we want to dynamically create components and then insert them into our view. Let’s start with a trivial example: every time the button in the HostComponent view is pressed, we want to create a dynamic component of type DynamicComponent and add it to our view. Let’s set up the skeleton components and templates.

<script src="https://gist.github.com/natmegs/d49d3f5788bfc11f549db49357617d00.js"></script>

<script src="https://gist.github.com/natmegs/f66f9308d457b77c28e03078db4eeb7f.js"></script>

<script src="https://gist.github.com/natmegs/f46cb5b253d7eb6f2812ec88bc61ad2d.js"></script>

The main players in the dynamic component game are <span class="code">ComponentFactory</span>, <span class="code">ComponentFactoryResolver</span>, <span class="code">ComponentRef</span>, and <span class="code">ViewContainerRef</span>. Using a combination of these utilities, we are able to create components and add them to a view from within a component.ts file. But WTF do they actually do?

###### ComponentFactoryResolver

The Angular docs provide no non-code description of the <span class="code">ComponentFactoryResolver</span>, but you can think of this as the bridge between a Component class, and the <span class="code">ComponentFactory</span> (we’ll get to that next). This is a service that we need to inject into our HostComponent to get access to, and it has one important method which is the <span class="code">resolveComponentFactory</span> method:

<span class="code">resolveComponentFactory<T>(component: Type<T>): ComponentFactory<T></span>

This method takes as a parameter the Component type you would like to create, and creates a <span class="code">ComponentFactory</span> for that Component type.

###### ComponentFactory

The Angular docs also provide no overview of what the <span class="code">ComponentFactory</span> is, but it pretty much does exactly what its name says. It’s a factory that can create Components. The most important method on this ComponentFactory class is the <span class="code">create</span> method:

<span class="code">create(injector: Injector, projectableNodes?: any[], rootSelectorORNode?: string | any, ngModule?: NgModuleRef<any>): ComponentRef<c></span>

And what this does is it creates a Component of whatever type we used in the <span class="code">resolveComponentFactory</span> method. This method returns the resulting <span class="code">ComponentRef</span>.

###### ComponentRef

So far we know our dynamic component creation flow starts with a Component type that we want to create, then uses the <span class="code">ComponentFactoryResolver</span> to create a <span class="code">ComponentFactory</span>, and then uses the <span class="code">ComponentFactory</span> to create a Component. And at the end of all of that we are left with a <span class="code">ComponentRef</span>. The <span class="code">ComponentRef</span> is a representation of the instance of a Component created by the factory. The <span class="code">ComponentRef</span> is incredibly useful, as it gives us access to the Component instance as well as some helpful utility properties and methods. One particularly important property is the <span class="code">hostView</span> property, which gives us access to the Component instance’s view. And one particularly important method on this class is the <span class="code">destroy</span> method, which we can use to destroy the Component instance.

###### ViewContainerRef

All of this dynamic component creation is not terribly helpful if we have no way of getting our new component into the view. The <span class="code">ViewContainerRef</span> represents a container where one or more Views can be attached. We saw that the <span class="code">ComponentRef</span> exposes the Component instance <span class="code">hostView</span>, and this is what we need to send to our <span class="code">ViewContainerRef</span> via its <span class="code">insert</span> method:

<span class="code">insert(viewRef: ViewRef, index?: number): ViewRef</span>

### Ok let's build something

Now that we’ve gotten past the (necessary but dense) jargon, we can actually get started. Our goal is to have a host component (HostComponent), which will create dynamic components (of type DynamicComponent) on the click of a button.

What we need before we go into the trenches are
- A predefined list of component types we want to dynamically create;
- Somewhere to put the dynamically created components in the view.

We already know that we are dynamically creating instances of DynamicComponent, but our HostComponent also needs to know this. We use the <span class="code">entryComponents</span> array in the HostComponent’s decorator metadata to indicate any components that will be dynamically added.

<script src="https://gist.github.com/natmegs/d2a605f8c1000647532c3b66351f6141.js"></script>

The <span class="code">entryComponents</span> array tells Angular about any other components that need to be compiled at the same time as the HostComponent. For each component in the array, Angular will create a <span class="code">ComponentFactory</span> and store it in the <span class="code">ComponentFactoryResolver</span>.

Now that our HostComponent knows which components we will be creating, we need somewhere to put their views. We can use any view element as the View Container, but for this example we will use an <span class="code">ng-container</span> element to avoid introducing redundant elements into the DOM.

<script src="https://gist.github.com/natmegs/f963fa8a71fb6e8133e02deec9a198dc.js"></script>

The local reference we added to the container element allows us to get access to this element in the Component. We can use the <span class="code">ViewChild</span> decorator in the component to get access to this element as a <span class="code">ViewContainerRef</span>:

<script src="https://gist.github.com/natmegs/a690781c5e35b8596c16ee48db37497c.js"></script>

And now we’re pretty much ready to go. We’ll add a button to the host template to trigger component creation, and we’ll build the corresponding component method.

<script src="https://gist.github.com/natmegs/dfe10422190b31bdf2ef1fd78096c0ba.js"></script>

<script src="https://gist.github.com/natmegs/67dfef1eeab82181b0d9da2cc2c102e9.js"></script>

First we injected the <span class="code">ComponentFactoryResolver</span> service into the component. Let’s break this <span class="code">addComponent</span> method down.
- We use the resolver service and the imported DynamicComponent to create a <span class="code">ComponentFactory</span>
- We use the <span class="code">ComponentFactory</span> to create a <span class="code">ComponentRef</span>
- We insert the <span class="code">ComponentRef</span>’s <span class="code">hostView</span> into the <span class="code">ViewContainerRef</span>

It should be mentioned that we could have also used the <span class="code">ViewContainerRef</span> instead of the <span class="code">ComponentFactory</span> to create the component, which would have looked like the following:

<script src="https://gist.github.com/natmegs/6782ba1b539ca727c7ffd97164656de0.js"></script>

The <span class="code">createComponent</span> method creates the component and adds its view to the View Container.

Continuing using the first method, we now have successfully created a new component on button click and added it to the view. However, the DynamicComponent has an input property which we have not dealt with, so it will show only ‘Hello !’ instead of ‘Hello (name)!’. We will add an input to the host template so that the user can enter a name and then create a new DynamicComponent using that name.

<script src="https://gist.github.com/natmegs/5e3f23e1ff0f419a03904b0560c430fb.js"></script>

And in the <span class="code">addComponent</span> method we will add this name as the input to our DynamicComponents.

<script src="https://gist.github.com/natmegs/b1e7d3130ab7c5c0ae774a2ff0b1e386.js"></script>

Notice that we set the Component instance input property before we insert it into the view. Using the second method of <span class="code">ViewContainerRef.createComponent</span> we would have to set the input property after it has already been added to the view.

The <span class="code">ViewContainerRef</span> <span class="code">insert</span> method can take an optional second parameter for the index at which to insert the View in the View Container. If not specified, the View will be inserted as the last View in the container. Some more useful properties of <span class="code">ViewContainerRef</span>:

- <span class="code">get length: number</span>: Returns the number of Views currently attached to this container.
- <span class="code">get(index: number): ViewRef | null</span>: Returns the <span class="code">ViewRef</span> for the View located at the specified index in the container.
- <span class="code">move(viewRef: ViewRef, currentIndex: number): ViewRef</span>: Moves a View identified by a <span class="code">ViewRef</span> to the container at the specified index. Returns the inserted <span class="code">ViewRef</span>.
- <span class="code">indexOf(viewRef: ViewRef): number</span>: Returns the index of the View, specified via <span class="code">ViewRef</span>, within the current container or -1 if not found.
- <span class="code">remove(index?: number): void</span>: Destroys a View attached to this container at the specified index. If index not specified, last View in container will be removed.
- <span class="code">detach(index?: number): ViewRef | null</span>: you can use this with insert to move a View within the container. If index parameter is omitted, the last ViewRef is detached.

These are useful utility functions for moving and retrieving Views from a View Container. We will continue with our simple example where the Views are added to the end of the list, but it is helpful to know that these functions are available for more complicated use cases.

Now we can add unlimited instances of the DynamicComponent to the view, but maybe we want to be able to trim the list. We’ll add buttons to Delete the view at the end of the list and Clear all views.

<script src="https://gist.github.com/natmegs/771bae6dcc0411dbd25b8be366e51ca2.js"></script>

<script src="https://gist.github.com/natmegs/8b1993a5b3456c39bd15384c1f062374.js"></script>

When we remove a Component from the view, we need to ensure that the Component instance gets destroyed as well as its View. So instead of using the <span class="code">ViewContainerRef</span> remove method, we will use the <span class="code">ComponentRef</span> <span class="code">destroy</span> method. This takes care of the Component and the View, since we are attaching the <span class="code">hostView</span> from the <span class="code">ComponentRef</span>. 

To delete the <span class="code">ComponentRef</span>s, we need to have access to them and therefore keep track of them. So we create a <span class="code">dynamicComponents</span> array to which we add new <span class="code">ComponentRef</span>s in <span class="code">addComponent</span>. 

### Wrap it all up

So we started out with a HostComponent and a DynamicComponent. We got the HostComponent ready to create instances of DynamicComponent by modifying the <span class="code">entryComponents</span>, injecting the <span class="code">ComponentFactoryResolver</span>, and creating a <span class="code">ViewContainerRef</span> in the template.

We used the <span class="code">ComponentFactoryResolver</span> to create a <span class="code">ComponentFactory</span>, which we used to create Component instances. We modified the instances to add input properties and then used the <span class="code">ViewContainerRef</span> to insert the instance’s <span class="code">hostView</span> into our container.

Finally, we used an array to keep track of all of our <span class="code">ComponentRef</span>s and we used the <span class="code">ComponentRef</span> <span class="code">destroy</span> method to remove our dynamically added views and destroy the Component instances.

You can check it out further with [this](https://stackblitz.com/edit/angular-dimhko) StackBlitz working demo.  

<span class="emphasis">Dynamic AF!</span>
