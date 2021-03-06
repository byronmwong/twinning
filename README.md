# twinning
Compare the result of a new function against an existing function.  
```js

const findCatsById = twinning({
  name: "findCatsById",
  oldFn: OldDatabase.findCatsById,
  newFn: NewDatabase.findCatsById,
  onDiff: onDiffs,
  onError: onError
});

findCatsById("1234", function(err, result) {
  /*
    `err` and `result` come from `OldDatabase.findCatsById`,
    If either function errors, their errors will be passed to `onError` (below).
    If no error occurs, but results are different, diffs will be passed to `onDiffs` (below).
    If either `onDiffs` or `onError` throws, that will become the error.
  */
});

function onDiffs (name, diffs) {
  diffs.forEach(function(diff) {
    console.log("Different result for", name, JSON.stringify(diff));
  });
}

function onError (name, oldError, newError) {
  if (oldError) {
    console.error("Error for old function in", name, oldError);
  }
  if (newError) {
    console.error("Error for new function in", name, oldError);
  }
}

```

## Setup

### Install

```
$ npm install twinning
```

## Options

### Parameters:
- `name` *(required)*: A label for this comparison. This so you can re-use `onDiffs` or `onError` between multiple comparison.
- `oldFn` *(required)*: The original function. It's assumed that this function is currently being used in production, and the results can be trusted. The function must take a callback as its last argument unless `promises` or `sync` is specified. Both this function and `newFn` must complete before the comparison will complete.
- `newFn` *(required)*: The new function to compare. We assume this function is not yet reliable, so its results will be thrown away after the comparison. The function should match the type (callback, promise, synchronous) of `oldFn`. Both this function and `oldFn` must complete before the comparison will complete.
- `onDiffs`: A function that will be called when `newFn` yields a different result than `oldFn`, but neither function errors. You might want to use this function to log differences, or perhaps throw. If this function throws, the comparison will return/yield that error. `onDiffs` will be called with the following arguments:
  - `name`: See above.
  - `diffs`: An array of change records between the results of `oldFn` and `newFn`. We've used [deep-diff](https://github.com/flitbit/diff) to implement the comparison; see their API for an overview of the structure of change records.
- `onError`: A function that will be called when either function errors. If `onError` is called, `onDiffs` will not be. You might want to use this option to log or throw the error. If this function throws, the comparison will return/yield that error. `onError` will be called with the following arguments
  - `name`: See above.
  - `oldError`: The error, if any, from `oldFn`.
  - `newError`: The error, if any, from `newFn`.
- `before`: If provided, this function will be called with the provided arguments. Its return value will then be used as the argument to `oldFn` and `newFn`. Note that this means that if a `before` block is used, `oldFn` and `newFn` can only take a single argument. If `before` throws an error, `oldFn` and `newFn` will not be run.
- `after`: If provided, this function will be called with the result of the comparison, and its return value will be returned instead. If an error occurs, `after` will not be called.
- `disabled`: If true, `newFn` will not be called, and no diffs will be calculated. `oldFn`, and `before` and `after` if provided, will still be called.
- `promises`: Set this to `true` if your `oldFn` and `newFn` return a promise instead of using callbacks.
- `sync`: Set this to `true` if youre `oldFn` and `newFn` are synchronous and do not use callbacks.


### Configuring defaults:
The `defaults` method returns a function that is pre-configured with certain options. This function can then be used exactly like the base method.
```js
const twinning = require("twinning");
const withDefaults = twinning.defaults({
  onDiffs: myDiffLogger,
  onError: myErrorHandler
});
const findCatsById = withDefaults({
  name: "findCatsById",
  oldFn: OldDatabase.findCatsById,
  newFn: NewDatabase.findCatsById
});
```
