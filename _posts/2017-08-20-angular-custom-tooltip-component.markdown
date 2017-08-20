---
layout: post
title:  "Angular Custom Tooltip Component"
date:   2017-08-20 13:59:30 -0700
permalink: /blog/:title
comments: true
---

At work I was recently tasked with the addition of a new (very minor) feature: a custom tooltip. At first I didn’t treat it like a feature (there must be something that already does what I need it to do right?) and assumed it would be a 5 minute bump in my day. Incorrect, somehow there was nothing I could find that gave me the ability to do precisely what this tooltip needed to do.

###### Constraints

I’ll start with the constraints of the task. Without going into too much detail, this is part of a business management application that allows managers to manage a collection of units and the daily tasks associated with them. A main theme of the application is displaying lists of units so that managers can access various information and perform various tasks on them. I needed to design a tooltip that, when hovering over a unit list item, would display contextual information about the unit as well as link to task options based on the status of the unit. This tooltip also needed to be generic enough to be applied to different situations, so it needed to take in custom input to display custom messages and options. The tooltip also needed to be clickable and hoverable.

Sounds trivial, but I quickly found that most tooltip libraries/npm modules available were missing at least one aspect of functionality that I needed. For example, there are dozens of Angular tooltip directives that take in a content string and a positioning string and then show a tooltip that displays the content string as a message. Great, but I needed to be able to show more than just a string message - think buttons, links, custom HTML content. I found [this](https://medium.com/@amcdnl/building-tooltips-for-angular2-396320fa938f) blog post that I thought was going to be my answer: it uses dynamic component injection to display custom components using a directive. I went through all of the steps as outlined and was confident that this was going to be the answer until I realized that the tutorial doesn’t touch on maybe the most crucial part of this whole tooltip thing: actually displaying the tooltip in the correct location. The dynamic component injection, while cool, also seemed overkill but I didn’t have a good alternative solution at this point.

###### Custom HTML Content

More searching and I found [this ngx-tooltip](https://github.com/pleerock/ngx-tooltip) module which allowed the insertion of custom HTML into a tooltip container component via <ng-content>. This seemed much more intuitive and seemed like it would fulfill all of my criteria. I implemented all of the tooltip components/directives by hand instead of installing through npm since I wanted to understand how it worked. Turns out this was a good idea since I realized that, while close, this tooltip module was not going to provide all of the functionality I needed. I was able to feed in custom HTML (ie custom Angular components) to the tooltip which made it perfectly generic and reusable, but I was unable to mouseover the tooltip itself. As soon as the mouse left the original hover location (ie to try to click a button in the tooltip) the tooltip would disappear.

###### Hoverable and Clickable

Thus the next challenge was to make the tooltip stay visible while hovering over the original host element AND while hovering over the tooltip itself. My solution is something that I’m not completely satisfied with, but works. In the component that is displaying the tooltip (ie the host component) I attached mouseover and mouseleave event listeners to the Tooltip Container element. I also created a Tooltip Service with a mouseInTooltip BehaviorSubject. To clarify, I will only be showing parts of the code that I have altered from the existing ngx-tooltip module, for which the code is available on [github](https://github.com/pleerock/ngx-tooltip).

<script src="https://gist.github.com/natmegs/b76c9dc1f498752d0f32f3500af1b011.js"></script>
<script src="https://gist.github.com/natmegs/707bc0340df4e660a1f1bbb6408def29.js"></script>

I used the mouseover and mouseleave handlers to emit either true or false from mouseInTooltip.

<script src="https://gist.github.com/natmegs/de61c47d350a6a58cf801b85971cf361.js"></script>

Then, in the Tooltip Directive hide function (called when the mouse leaves the original host element hover location), I check the current value of mouseInTooltip to see if the mouse is in the tooltip before it closes. In addition, I added a setTimeout to the hide function because otherwise the hide function would never see that the tooltip is being hovered, as it would close before the mouse reached the tooltip.

<script src="https://gist.github.com/natmegs/9f8d4ce21e5591a3092b4d43d99301fe.js"></script>

I also subscribe to mouseInTooltip to get notified when it changes, so that I can close the tooltip when the value changes to false.

<script src="https://gist.github.com/natmegs/cd972e21659cdebc59eec71f45b80f0a.js"></script>

The reason I’m not completely satisfied with this solution is that it needs to be implemented independently in every component that uses the tooltip and wants this functionality. Any host component would need to have mouseover/mouseleave event handlers and would need to have the tooltip service injected. I would like a solution where this is handled entirely in the directive/container component instead of involving the host component.


###### Remaining Challenges

An outstanding bug that I have not yet found a good solution for is dealing with border cases. For example, if the tooltip position is set to “bottom”, but this results in the tooltip being cut off, I would like the positioning to switch automatically to “top”. My direction so far is to get the ElementRef of the container component (where the edges are located), and use the difference between the trigger location and the edge location to determine whether the tooltip can be shown in the given positioning. Something along the lines of the following:

<script src="https://gist.github.com/natmegs/ebd6290a98d0cb4b9b262ef0566f4dfe.js"></script>

Am I being an idiot and missing something super simple that can help with the positioning issue? Have suggestions for how this could be improved? I want to hear from you!
