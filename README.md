# Velocity TFF

## Usecase
You want to use Javascript and [Velocity] not only for the typical UI / Effect animation, but for a more complex 3D-scenario.
In particular, you noticed that you often need to *animate multiple [CSS3 transform-functions] simultaneously on a single element*, like:

**Example 1**
```
transform: rotateZ(30deg) translateY(100px)
```

You certainly tried to accomplish this with Velocity's built-in support for CSS3-Transforms, but the limited ability to control the order in which transform-functions (let's call them *TFF*s from now) are applied led to undesired results. Simply put, `"move forward, then turn right"` is not the same as `"turn right, then move forward"`.

Maybe you also stumbled upon [this nice PR] that allows you to define the order of TFFs as a Velocity option. However you noticed that..

* passing this option on each call soon becomes painful,
* it currently does not support custom queues or the `queue:false` option and
* you sometimes need to apply *the same TFF multiple times on the same element*, like so:

    **Example 2**
    ```
    transform: rotateZ(30deg) translateY(100px) rotateX(45deg) translateY(60px)
    ```

This package allows you to

- have TFFs reliably applied in the order you specify,
- control animation of each of the TFFs individually, even if one transform name occurs multiple times,
while..
- preserving the behavior of your existing Velocity code (even transforms),
- maintaining Velocity's key functionalities to make transforms fast, value caching and the `mobileHA` option, and
- remaining as close as possible to Velocity's syntax.

## Basic usage
### Put multiple simultaneous TFFs in order
You can set an element's transformSequence via `$.Velocity.transformSequence()`. Pass an array describing the ordered list of TFFs. 

Say you want to animate an element from its initial position to the state described in Example 1 above.

*(Snippet 1)*
```javascript
$.Velocity.transformSequence($el, ['rotateZ', 'translateY']);
/* Now you can pass the properties in any order. */
$el.velocity({translateY: 100, rotateZ: 30});
```

Note: The TFFs of the transformSequence are initialized to their *identityValue*, that is 1 for the scale-transforms and 0 for all others.

### Append additional TFFs, some of which multiple times.
Now let's say later on you want to tilt the object and move it further towards its front, leading to the final state of Example 2.

Pass the additional effects as an option to `transformSequence()` and pass `'append'` as an additional parameter. This will append the sequence to the existing transformString, leaving the previous TFFs intact.

As there are now two TFFs of type `translateY` you have to refer to each single TFF by its (zero-based) index in the transformSequence like so:

*(Snippet 2)*
```javascript
$.Velocity.transformSequence($el, ['rotateX', 'translateY'], 'append');
/* Now you can refer to the second 'translateY' by its index (3). */
$el.velocity({rotateX: 45, tff3: 60});
```

### Modify TFF value later
The same technique (of referring to a TFF by index) allows you to modify an individual TFF later on. Say, *after* having called Snippet 2 you want to animate the *first* `translateY` TFF (which is at position 1) back to zero:

*(Snippet 3)*
```javascript
$el.velocity({tff1: 0});
```

### Various
The `Velocity.hook()` function is also supported (that is, you can pass the index-form as the hook name).

If you want to remove parts (or all) of the internal transformSequence, use `Velocity.reduceTransformSequence()`. It takes the first two arguments from [Array.prototype.splice()]. For example, to remove the last three TFFs:
```javascript
$.Velocity.reduceTransformSequence($el, -3);
```
Always make sure that the TFFs you are about to remove are at their identityValues.

## Advanced usage
You can also pass a transformSequence as an option to the `velocity()` call. It will be appended to the transform string, just as in Snippet 2.

But now all TFFs you specify in the property map will refer to the passed transformSequence and never alter the values in the previous transform string.

To achieve the same effect of Snippet 2 you can do the following.
*(Snippet 4)*
```javascript
$el.velocity({rotateX: 45, translateY: 60}, {transformSequence: ['rotateX', 'translateY']});
```
Note that `translateY` here unambiguously refers to the one in the transformSequence and not to the `translateY` TFF in the existing transform string. So this syntax can be useful to avoid the (admittedly ugly) index form.

## Best practices
The syntax is quite flexible and therefore can be used in a lot of different ways, good and bad.

This is an obviously debatable list of rules of thumb about how to make the best use of the package.
- *Setting* the `transformSequence` (Snippet 1) should only ever be done when the element is in its initial state (no transforms applied). So prefer to set the entire sequence in advance, even if some transforms are left at their identity values initially.
- If you want to initialize a transform property to a value other than its zero-value, use force-feeding (if you animate) or the `hook()` function (if you don't).
- For clarity of code, use the index-form only if necessary, that is if you have more than one occurrence of the TFF in the `transformSequence`. Even then 

## Caveats
- I could not resist to use some of the ES2015 syntax (arrow functions, `let`/`const`, template literals, etc.). So you need to use [Babel] or something similar to make it work on all the browsers supported by Velocity.

[Velocity]: http://julian.com/research/velocity/
[CSS3 transform-functions]: https://drafts.csswg.org/css-transforms/#transform-functions
[this nice PR]: https://github.com/julianshapiro/velocity/pull/459
[Array.prototype.splice()]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice
[Babel]: https://babeljs.io/
