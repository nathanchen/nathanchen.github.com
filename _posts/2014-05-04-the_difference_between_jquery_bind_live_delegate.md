---
layout: post
title: "The difference between jQuery's bind() live() and delegate()"
category: Reading Notes
tags: ["读文章", "JavaScript"]
---
{% include JB/setup %}

A simple HTML page would look like this:

[img/bind01.png](/img/bind01.png)

#### .bind()

	$('a').bind('click', function() { alert("That tickles!") });
	
jQuery scans the document for all `$('a')` elements and binds the alert function to each of their `click` events

#### .live()

	$('a').live('click', function() { alert("That tickles!") });
	
jQuery binds the alert function to the `$(document)` element, along with `click` and `a` as parameters. Anytime an event bubbles up to the document node, it checks to see if the event was a click and if the target element of that event matches the `a` CSS selector.

The live method can also be bound to a specific element other that `document`

	$('a', $('#container')[0]).live(...);
	
#### .delegate()

	$('#container').delegate('a', 'click', function() { alert("That tickles!") });
	
jQuery scans the document for `$('#container')`, and binds the alert function along with the `click` event and `a` CSS selector as parameters. Anytime an event bubbles up to `$('#container')`, it checks to see if the event was a click and if the target element of that event matches the CSS selector.

### Why .delegate() is better than .live()

### Why .live() or .delegate() instead of .bind()

There are 2 reasons we prefer `delegate` or `live` to `bind`:

- To attach handlers to DOM elements that may not yet exist in the DOM. Because `bind` directly binds handlers to the individual element, it cannot bind them to elements that aren't on the page yet.

If you were to run `$('a').bind(...)`, and then new links were added to the page via AJAX, your bind handler would not work for these.

- Or to attach a handler to a single element or small group of elements, listening for events on descendent elements, instead of looping through and attaching the same function to 100 individual elements in the DOM. This would be the performance benefit of attaching a handler to one ancestor element(s) instead of directly attaching handlers to all elements on the page.



#### Reference

http://www.alfajango.com/blog/the-difference-between-jquerys-bind-live-and-delegate/