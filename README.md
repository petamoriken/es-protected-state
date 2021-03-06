#Protected State for ECMAScript

Stage NaN Proposal (My Personal Strawman)

##Overview

* Provide `protected` properties & methods for userland object
* Only accessible in instance of class defined `protected` properties & methods and that's subclass

##Example

###Single Protected Property

```js
class Foo {
	protected bar;

	constructor() {
		// shorthand for `this##bar = 1;`
		##bar = 1;
	}

	getBar() {
		return ##bar;
	}
}

class SubFoo extends Foo {
	getBarAndIncliment() {
		return ##bar++;
	}
}
```

* This code supported as Syntax Sugar for below ES2015 code (using [Class Fields](https://github.com/jeffmo/es-class-fields-and-static-properties) for the sake of clarity):
	
	```js
	// utility
	function createProtectedStorage() {
		"use strict";
		
		const wm = new WeakMap();
	
		return (self, protectedClass) => {
			const obj = wm.get(self);
	
			if(protectedClass == null) {
				if(obj) {
					return obj;
				} else {
					const ret = Object.create(null);
					wm.set(self, ret);
					return ret;
				}
			}
	
			const p = new protectedClass(self);
			if(obj) {
				for(let key of Reflect.ownKeys(obj)) {
					const descriptor = Object.getOwnPropertyDescriptor(obj, key);
					Object.defineProperty(p, key, descriptor);
				}
			}
			wm.set(self, p);
			return p;
		}
	}
	
	const _ = createProtectedStorage();
	```
	
	```js
	class Protected_Foo {
		bar;
	
		constructor(publicThis) {
			this.public = publicThis;
		}
	}
	
	class Foo {
		constructor() {
			// protected instance
			if(new.target === Foo) {
				_(this, Protected_Foo);
			}
	
			// protected property
			_(this).bar = 1;
		}
	
		getBar() {
			return _(this).bar;
		}
	}
	
	class Protected_SubFoo extends Protected_Foo {
	}
	
	class SubFoo extends Foo {
		constructor() {
			super();
	
			// protected instance
			if(new.target === SubFoo) {
				_(this, Protected_SubFoo);
			}
		}
	
		getBarAndIncliment() {
			return _(this).bar++;
		}
	}
	```

###Override Protected Methods

proposal code:

```js
class Foo {
	protected bar;

	constructor() {
		##bar = 1;
	}

	getBar() {
		return ##bar;
	}

	protected baz() {
		++##bar;
	}
}

class SubFoo extends Foo {
	getBarAndIncliment() {
		return ##bar++;
	}

	protected baz() {
		super##baz();
		++##bar;
	}

	callBaz() {
		##baz();
	}
}
```

* in ES2015 (omitted part of the utility code):

	```js
	class Protected_Foo {
		bar;
	
		constructor(publicThis) {
			this.public = publicThis;
		}
	
		baz() {
			++this.bar;
		}
	}
	
	class Foo {
		constructor() {
			// protected instance
			if(new.target === Foo) {
				_(this, Protected_Foo);
			}
	
			// protected property
			_(this).bar = 1;
		}
	
		getBar() {
			return _(this).bar;
		}
	}
	
	class Protected_SubFoo extends Protected_Foo {
		baz() {
			super.baz();
			++this.bar;
		}
	}
	
	class SubFoo extends Foo {
		constructor() {
			super();
	
			// protected instance
			if(new.target === SubFoo) {
				_(this, Protected_SubFoo);
			}
		}
	
		getBarAndIncliment() {
			return _(this).bar++;
		}
	
		callBaz() {
			_(this).baz();
		}
	}
	```

###Using Public Property in Private Methods

proposal code:

```js
class Name {
	name;

	constructor(name) {
		this.name = name;
	}

	getDecoratedName(type) {
		if(type === "upper") {
			return ##upper();
		} else if(type === "star") {
			return ##star();
		} else if(type === "dagger") {
			return ##dagger();
		}
	}

	protected upper() {
		return this.name.toUpperCase();
	}

	protected star() {
		return `☆${ this.name }☆`;
	}

	protected dagger() {
		return `†${ this.name }†`;
	}
}
```

* in ES2015 (omitted part of the utility code):

	```js
	class Protected_Name {
		constructor(publicThis) {
			this.public = publicThis;
		}
	
		upper() {
			return this.public.name.toUpperCase();
		}
	
		star() {
			return `☆${ this.public.name }☆`;
		}
	
		dagger() {
			return `†${ this.public.name }†`;
		}
	}
	
	class Name {
		name;
	
		constructor(name) {
			// protected instance
			if(new.target === Name) {
				_(this, Protected_Name);
			}
	
			// public property
			this.name = name;
		}
	
		getDecoratedName(type) {
			if(type === "upper") {
				return _(this).upper();
			} else if(type === "star") {
				return _(this).star();
			} else if(type === "dagger") {
				return _(this).dagger();
			}
		}
	}
	```

##Private/Protected Library by Other Thoughts

<a href="https://github.com/gaogao-9/access-modifiers" target="_blank">access-modifiers</a> (npm module): based on Symbol, non-strict private/protected state but reflectable

##Related Proposals in ES.next

* <a href="https://github.com/jeffmo/es-class-fields-and-static-properties" target="_blank">Class Property Declarations</a> (Stage 1)
* <a href="https://github.com/wycats/javascript-private-state" target="_blank">Private State for objects</a>, <a href="https://github.com/zenparsing/es-private-fields" target="_blank">Private Fields</a> (Stage 0)
