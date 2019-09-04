# function-decorators-proposal

That's my humble attempt to move the topic of function decorators from the dead point. I know, it's discussed many times but IMO [current implementation of decorators](https://github.com/tc39/proposal-decorators) cover only a tiny amount of needs and allows to power a small amount of specialized libraries (like [MobX](https://github.com/mobxjs/mobx)). 

Function decorators in their turn open a huge amount of scopes where they can be used. See my [comment](https://github.com/wycats/javascript-decorators/issues/4#issuecomment-526116110). We all love classess but we don't use them too often, but we do use functions a lot. I don't know any big JavaScript project where regular functions wouldn't be used. But I do know projects without classess at all or with a tiny amount of them.

As far as I know the main concern of not moving with this topic forward is the issue with hoisting of a decorated function declaration. 

> Since JS hoists up function declarations, those expressions that you use inside of decorators, depending on proposed "solution", might get hoisted up too, and thus break execution flow, or prevent function from hoisting, and thus break backwards compatibility. See Sebastian McKenzie's answer here for a complete example and explanation. (This is not an issue for classes since those are specified as non-hoistable)

https://rreverser.com/ecmascript-decorators-and-functions/ (Ingvar Stepanyan)

In my opinion we need to refuse any hoisting when decorators are used and as far as I understand the topic it **doesn't** break backwards compatibility.

```js
// hoists
function foo() {
}

// doesn't hoist
@decorator function bar() {
}
```


There are some noteworthy alternatives to decorators I'm going to mention shortly:

1. Wrappers (enhancers). This syntax is actually does the decorator's job but doesn't do it nicely. I would call it the only usable alternative to decorators by the time being.
```js
const foo = decorator(function foo() {});
```
2. Pipeline operator.

```js
const foo = (function foo() {}) |> decorator;
```

3. Via bind operator (taken from Ingvar Stepanyan's article).

```js
const foo = (function foo() {})::decorator();
```

The wrapper looks better than two other cases because it makes possible to see what does decorate the function before it was declared.

## Function decorators should behave as wrappers

(If "wrapper" is a good name here).

The main idea of moving forward with function decorators is to make them behave like there were defined and wrapped by another function, not more. Currently there is only a few cases we actually need to cover: 

- Function declarations (pure, async, generators);
- Function expressions (pure, async, generators);
- Arrow functions (pure, async, generators(?));

Let's imagine a function `decorate(decorators, func)` which applies decorators to a function. Let's imagine as we transpile function definitions and their decorators by replacing them by a call if this function.


### Function expression and arrow functions

```js
const foo = @decorator1 @decorator2 function bar() { return "Hello" }

// Will become:

const foo = decorate([decorator1, decorator2], function bar() { return "Hello" });
```

And

```js
const foo = @decorator1 @decorator2 () => "Hello"

// Will become:

const foo = decorate([decorator1, decorator2], () => "Hello");
```

### Function declarations

And this is the most important. I propose to make a decorated function declaration behave as `let` definition.

```js
@decorator1 @decorator2 function foo() { return "Hello" }

// Will become:

let foo = decorate([decorator1, decorator2], function foo() { return "Hello" });
```

### Async functions and generator functions

I wouldnt't call async functions and generator functions a very special case of this syntax. They also need to be put into `decorate` call as a second argument, without any change:

```js
@decorator1 @decorator2 function foo*() { return "Hello" }

// Will become:

let foo = decorate([decorator1, decorator2], function foo*() { return "Hello" });
```

```js
@decorator1 @decorator2 async function foo() { return "Hello" }

// Will become:

let foo = decorate([decorator1, decorator2], async function foo() { return "Hello" });
```

```js
const foo = @decorator1 @decorator2 async () => "Hello"

// Will become:

const foo = decorate([decorator1, decorator2], async () => "Hello");
```

