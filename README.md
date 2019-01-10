# Freezing prototypes

## Current status

Stage 0 - has not been presented to the committee.


## Proposal

Currently it is impossible to freeze an object's `[[Prototype]]` internal slot without also making the object non-extensible, i.e., preventing new properties from being added to it: all of `Object.preventExtensions`, `Object.seal`, and `Object.freeze` also mark their argument as non-extensible. (In fact, the spec mechanism for a frozen prototype on a non-exotic object is to set the `[[Extensible]]` internal slot to false.)

ECMA-262 defines [Immutable Prototype Exotic Objects](https://tc39.github.io/ecma262/#sec-immutable-prototype-exotic-objects), which are objects whose prototype cannot be changed but which still may be extensible. The only one in ECMAScript is `Object.prototype`, though the web platform's [WindowProxy exotic object](https://html.spec.whatwg.org/#windowproxy-setprototypeof) behaves similarly. There is no way to create these objects.

This proposal would add some way to freeze an object's `[[Prototype]]` without making the object non-extensible.

See [tc39/ecma262#538](https://github.com/tc39/ecma262/issues/538) for issue discussing this.


## Motivation

One major motivating case is derived classes. Calling `super()` in a derived class will invoke whatever that class's `[[Prototype]]` is at the time of the `super()` call. But except in rare circumstances the author of the class would generally expect this to be one thing which did not change over the class's lifetime, and may wish to enforce this without necessarily preventing new static methods from being added to the class later.

Web platform classes like `TextNode` differ from ECMAScript `class {}` classes in that they always invoke their original superconstructor. It would perhaps be more sensible if they could have their `[[Prototype]]` frozen, but it would be undesirable to make them exotic objects - it would be better if the language itself could explain their behavior without anything exotic.


## Details / Open questions

It's not clear what the API for this would be. Two  possible designs (a non-exhaustive list):

- Two new Reflect methods (and corresponding traps) for setting and querying prototype immutability
  - How would / could / should these be made consistent with existing traps, particularly the `isExtensible` trap?
- A new options-bag parameter to `preventExtensions` and `isExtensible` which would specify that these should only apply to the `[[Prototype]]`
  - Harder to feature detect; if you pass the option on an implementation which didn't support it, you'd freeze more than intended
