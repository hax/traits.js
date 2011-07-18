# traits.js #

This is a fork of TraitsJS, please see [official site][traitsjs] for brief introduction.

## What added? ##

This fork introduce _augment(object, trait)_ method.
It's very like _create(proto, trait)_ , 
but it not create a new object using _proto_ , just augment _object_

## Why we need augment? ##

There are two cases which _create_ not align, but _augment_ help

### 1. To augment an array or function object ###

Array and Function is special in JavaScript, 
Trait.create() or Object.create() doesn't work for them

	var a = Object.create([1,2,3], {})
	a[4] = 4 // expect a.length == 4
	a.length // but still 3 
	
	var f = Object.create(function(x){ return x }, {})
	typeof f // return 'object', not 'function'
	f(1) // error, you can't call it
	
So instead of creating failed array/function, we can augment the original array/function

	var TContains = Trait({
		indexOf: Trait.required,
		contains: function(e) {
			return this.indexOf(e) !== -1
		}
	}
	
	var a = [1, 2, 3]
	
	Trait.augment(a, TContains)
	
	a.contains(2) // -> true
	
You can augment the Array.prototype as well, in fact, 
we definitily need a way to utilize traits composition for built-in Objects, 
because ES5 spec don't allow modify prototype of built-in Objects.

	// Array.prototype = Trait.create(Array.prototype, TContains) // doesn't work!
	
	// This works
	Trait.augment(Array.prototype, TContains)
	
	[4, 5, 6].contains(5) // -> true

### 2. To augment DOM objects ###

As ES5 spec, all Array methods are generic, so we may want to use them for DOM collections

	TIteration = Trait({
		length: Trait.required,
		forEach: Array.prototype.forEach,
		map: Array.prototype.map,
		filter: Array.prototype.filter,
		every: Array.prototype.every,
		some: Array.prototype.some,
		reduce: Array.prototype.reduce,
		reduceRight: Array.prototype.reduceRight
	})
	
But just like JavaScript built-in objects, most browser don't allow you to 
modify the prototype of DOM objects' prototype

	NodeList.prototype = Trait.create(NodeList.prototype, TIteration) // not allowed!

Below may work in morden browers, but may cause crash (in Chrome) or errors (in Firefox) sometimes,
and totally doesn't work in IE, becuase JScript doesn't allow using DOM object as prototype for JS object.

	var divs = document.getElementsByTagName('div')
	divs = Trait.create(divs, TIteration)
	divs.forEach(function(e) { console.log(e) })

But we can do this:

	// The third arg {required:false} means not check for remaining required prop,
	// because many browser doesn't implement "length" (which we declare being required
	// in TIteration trait) in NodeList/HTMLCollection prototype.
	Trait.augment(NodeList.prototype, TIteration, {required:false})
	Trait.augment(HTMLCollection.prototype, TIteration, {required:false})
	
	// expand all navigation list which contains links  	
	document.querySelectorAll('nav>ul, nav>ol').filter(function(list) {
		return list.childNodes.every(function(li){
			return li.firstChild.nodeName.toLowerCase() === 'a'
		})
	}).forEach(function(list){
		list.classList.remove('collapse')
	})

Such code works fine on all browsers from IE8 (standard mode) to FF, Chrome and Safari.
NOTE: For IE8 or some old WebKit-based browsers (not tested yet, but maybe 
Android 1.x - 2.0 browsers), you need [my revised es5-shim][es5-shim] to make it work.

## TODO ##

Nothing planned yet, advice are welcome.
	
[traitsjs]:http://www.traitsjs.org/
[es5-shim]:http://github.com/hax/es5-shim