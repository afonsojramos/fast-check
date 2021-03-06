# [:house:](../README.md) Arbitraries

Arbitraries are responsible for the random - *but deterministic* - generation and shrink of datatypes. [They can be combined together](./AdvancedArbitraries.md) to build more complex datatypes.

This documentation lists all the built-in arbitraries provided by fast-check.

You can refer to the [generated API docs](https://dubzzz.github.io/fast-check/#/2-API/fast-check) for more details.

## Table of contents

- [Boolean](#boolean-boolean)
- [Numeric](#numeric-number)
- [String](#string-string)
- [Date](#date-date)
- [Combinators](#combinators-t)
- [Objects](#objects-any)
- [Recursive structures](#recursive-structures)
- [Functions](#functions)
- [Extended tools](#extended-tools)
- [Model based testing](#model-based-testing)
  - [Commands](#commands)
  - [Arbitrary](#arbitrary)
  - [Model runner](#model-runner)
  - [Simplified structure](#simplified-structure)
- [Race conditions detection](#race-conditions-detection)
  - [Scheduling methods](#scheduling-methods)
  - [Wrapping calls automatically using act](#wrapping-calls-automatically-using-act)
  - [Model based testing and race conditions](#model-based-testing-and-race-conditions)

## Boolean (:boolean)

- `fc.boolean()` either `true` or `false`

## Numeric (:number)

Integer values:

- `fc.integer()` all possible integers ie. from -2147483648 (included) to 2147483647 (included)
- `fc.integer(max: number)` all possible integers between -2147483648 (included) and max (included)
- `fc.integer(min: number, max: number)` all possible integers between min (included) and max (included)
- `fc.nat()` all possible positive integers ie. from 0 (included) to 2147483647 (included)
- `fc.nat(max: number)` all possible positive integers between 0 (included) and max (included)
- `fc.maxSafeInteger()` all possible positive integers between `Number.MIN_SAFE_INTEGER` (included) and `Number.MAX_SAFE_INTEGER` (included)
- `fc.maxSafeNat()` all possible positive integers between 0 (included) and `Number.MAX_SAFE_INTEGER` (included)

Floating point numbers:

- `fc.float()` uniformly distributed `float` value between 0.0 (included) and 1.0 (excluded)
- `fc.float(max: number)` uniformly distributed `float` value between 0.0 (included) and max (excluded)
- `fc.float(min: number, max: number)` uniformly distributed `float` value between min (included) and max (excluded)
- `fc.double()`uniformly distributed `double` value between 0.0 (included) and 1.0 (excluded)
- `fc.double(max: number)`uniformly distributed `double` value between 0.0 (included) and max (excluded)
- `fc.double(min: number, max: number)`uniformly distributed `double` value between min (included) and max (excluded)

BigInt (if supported by your JavaScript interpreter):

- `fc.bigIntN(n: number)` all possible `bigint` between -2^(n-1) (included) and 2^(n-1)-1 (included)
- `fc.bigInt()` uniformly distributed `bigint` values
- `fc.bigInt(min: bigint, max: bigint)` all possible `bigint` between min (included) and max (included)
- `fc.bigUintN(n: number)` all possible `bigint` between 0 (included) and 2^n -1 (included)
- `fc.bigUint()` uniformly distributed `bigint` positive values
- `fc.bigUint(max: bigint)` all possible `bigint` between 0 (included) and max (included)

## String (:string)

Single character only:

- `fc.hexa()` one character in `0123456789abcdef` (lower case)
- `fc.base64()` one character in `A-Z`, `a-z`, `0-9`, `+` or `/`
- `fc.char()` between 0x20 (included) and 0x7e (included) , corresponding to printable characters (see https://www.ascii-code.com/)
- `fc.ascii()` between 0x00 (included) and 0x7f (included)
- `fc.unicode()` between 0x0000 (included) and 0xffff (included) but excluding surrogate pairs (between 0xd800 and 0xdfff). Generate any character of UCS-2 which is a subset of UTF-16 (restricted to BMP plan)
- `fc.char16bits()` between 0x0000 (included) and 0xffff (included). Generate any 16 bits character. Be aware the values within 0xd800 and 0xdfff which constitutes the surrogate pair characters are also generated meaning that some generated characters might appear invalid regarding UCS-2 and UTF-16 encoding
- `fc.fullUnicode()` between 0x0000 (included) and 0x10ffff (included) but excluding surrogate pairs (between 0xd800 and 0xdfff). Its length can be greater than one as it potentially contains multiple UTF-16 characters for a single glyph

Multiple characters:

- `fc.hexaString()`, `fc.hexaString(maxLength: number)` or `fc.hexaString(minLength: number, maxLength: number)` string based on characters generated by `fc.hexa()`
- `fc.base64String()`, `fc.base64String(maxLength: number)` or `fc.base64String(minLength: number, maxLength: number)` string based on characters generated by `fc.base64()`. Provide valid base64 strings: length always multiple of 4 padded with '=' characters. When using `minLength` and `maxLength` make sure that they are compatible together. For instance: asking for `minLength=2` and `maxLength=4` is impossible for base64 strings as produced by the framework
- `fc.string()`, `fc.string(maxLength: number)` or `fc.string(minLength: number, maxLength: number)` string based on characters generated by `fc.char()`
- `fc.asciiString()`, `fc.asciiString(maxLength: number)` or `fc.asciiString(minLength: number, maxLength: number)` string based on characters generated by `fc.ascii()`
- `fc.unicodeString()`, `fc.unicodeString(maxLength: number)` or `fc.unicodeString(minLength: number, maxLength: number)` string based on characters generated by `fc.unicode()`
- `fc.string16bits()`, `fc.string16bits(maxLength: number)` or `fc.string16bits(minLength: number, maxLength: number)` string based on characters generated by `fc.char16bits()`. Be aware that the generated string might appear invalid regarding the unicode standard as it might contain incomplete pairs of surrogate
- `fc.fullUnicodeString()`, `fc.fullUnicodeString(maxLength: number)` or `fc.fullUnicodeString(minLength: number, maxLength: number)` string based on characters generated by `fc.fullUnicode()`. Be aware that the length is considered in terms of the number of glyphs in the string and not the number of UTF-16 characters
- `fc.stringOf(charArb: Arbitrary<string>)`, `fc.stringOf(charArb: Arbitrary<string>, maxLength: number)` or `fc.stringOf(charArb: Arbitrary<string>, minLength: number, maxLength: number)` string based on characters generated by `charArb`. `minLength` and  `maxLength` define bounds of the number of elements generated using `charArb` that can be found in the final string

More specific strings:

- `fc.json()` or `fc.json(maxDepth: number)` json strings having keys generated using `fc.string()`. String values are also produced by `fc.string()`
- `fc.unicodeJson()` or `fc.unicodeJson(maxDepth: number)` json strings having keys generated using `fc.unicodeString()`. String values are also produced by `fc.unicodeString()`
- `fc.lorem()`, `fc.lorem(maxWordsCount: number)` or `fc.lorem(maxWordsCount: number, sentencesMode: boolean)` lorem ipsum strings. Generator can be configured by giving it a maximum number of characters by using `maxWordsCount` or switching the mode to sentences by setting `sentencesMode` to `true` in which case `maxWordsCount` is used to cap the number of sentences allowed
- `fc.ipV4()` IP v4 strings
- `fc.ipV4Extended()` IP v4 strings including all the formats supported by WhatWG standard (for instance: 0x6f.9)
- `fc.ipV6()` IP v6 strings
- `fc.uuid()` UUID strings having only digits in 0-9a-f (only versions in v1 to v5)
- `fc.uuidV(versionNumber: 1|2|3|4|5)` UUID strings for a specific UUID version only digits in 0-9a-f
- `fc.domain()` Domain name with extension following RFC 1034, RFC 1123 and WHATWG URL Standard
- `fc.webAuthority()` Web authority following RFC 3986
- `fc.webFragments()` Fragments to build an URI. Fragment is the optional part right after the # in an URI
- `fc.webQueryParameters()` Query parameters to build an URI. Fragment is the optional part right after the ? in an URI
- `fc.webSegment()` Web URL path segment
- `fc.webUrl()` Web URL following the specs specified by RFC 3986 and WHATWG URL Standard
- `fc.emailAddress()` Email address following RFC 1123 and RFC 5322
- `fc.mixedCase(stringArb: Arbitrary<string>)` or `fc.mixedCase(stringArb: Arbitrary<string>, constraints: MixedCaseConstraints)` Randomly switch the case of characters generated by `stringArb`

## Date (:Date)

- `fc.date()` or `fc.date({min?: Date, max?: Date})` any date between new Date(-8640000000000000) or min (included) to new Date(8640000000000000) or max (included)

## Combinators (:T)

- `fc.constant<T>(value: T): Arbitrary<T>` constant arbitrary only able to produce `value: T`
- `fc.constantFrom<T>(...values: T[]): Arbitrary<T>` randomly chooses among the values provided. It considers the first value as the default value so that in case of failure it will shrink to it. It expects a minimum of one value and throws whether it receives no value as parameters. It can easily be used on arrays with `fc.constantFrom(...myArray)` (or `fc.constantFrom.apply(null, myArray)` for older versions of TypeScript/JavaScript)
- `fc.clonedConstant<T>(value: T): Arbitrary<T>` constant arbitrary only able to produce `value: T`. If it exists, it called its `[fc.cloneMethod]` at each call to generate
- `fc.mapToConstant<T>(...entries: { num: number; build: (idInGroup: number) => T }[]): Arbitrary<T>` generates non-contiguous ranges of values by mapping integer values to constant
- `fc.oneof<T>(...arbs: Arbitrary<T>[]): Arbitrary<T>` randomly chooses an arbitrary at each new generation. Should be provided with at least one arbitrary. All arbitraries are equally probable and shrink is still working for the selected arbitrary. `fc.oneof` is able to shrink inside the failing arbitrary but not accross arbitraries (contrary to `fc.constantFrom` when dealing with constant arbitraries)
- `fc.frequency<T>(...warbs: WeightedArbitrary<T>[]): Arbitrary<T>` randomly chooses an arbitrary at each new generation. Should be provided with at least one arbitrary. Probability to select a specific arbitrary is based on its weight, the higher it is the more it will be probable. It preserves the shrinking capabilities of the underlying arbitrary
- `fc.option<T>(arb: Arbitrary<T>): Arbitrary<T | null>` or `fc.option<T>(arb: Arbitrary<T>, freq: number): Arbitrary<T | null>` arbitrary able to nullify its generated value. When provided a custom `freq` value it changes the frequency of `null` values so that they occur one time over `freq` tries (eg.: `freq=5` means that 20% of generated values will be `null` and 80% would be produced through `arb`). By default: `freq=5`
- `fc.subarray<T>(originalArray: T[]): Arbitrary<T[]>`, or `fc.subarray<T>(originalArray: T[], minLength: number, maxLength: number): Arbitrary<T[]>` subarray of `originalArray`. Values inside the subarray are ordered the same way they are in `originalArray`. By setting the parameters `minLength` and/or `maxLength`, the user can change the minimal (resp. maximal) size allowed for the generated subarray. By default: `minLength=0` and `maxLength=originalArray.length`
- `fc.shuffledSubarray<T>(originalArray: T[]): Arbitrary<T[]>`, or `fc.shuffledSubarray<T>(originalArray: T[], minLength: number, maxLength: number): Arbitrary<T[]>` subarray of `originalArray`. Values within the subarray are ordered randomly. By setting the parameters `minLength` and `maxLength`, the user can change the minimal and maximal size allowed for the generated subarray. By default: `minLength=0` and `maxLength=originalArray.length`
- `fc.array<T>(arb: Arbitrary<T>): Arbitrary<T[]>`, `fc.array<T>(arb: Arbitrary<T>, maxLength: number): Arbitrary<T[]>` or `fc.array<T>(arb: Arbitrary<T>, minLength: number, maxLength: number): Arbitrary<T[]>` array of random length containing values generated by `arb`. By setting the parameters `minLength` and `maxLength`, the user can change the minimal and maximal size allowed for the generated array. By default: `minLength=0` and `maxLength=10`
- `fc.set<T>(arb: Arbitrary<T>): Arbitrary<T[]>`, `fc.set<T>(arb: Arbitrary<T>, maxLength: number): Arbitrary<T[]>` or `fc.set<T>(arb: Arbitrary<T>, minLength: number, maxLength: number): Arbitrary<T[]>` set of random length containing unique values generated by `arb`. All the values in the set are unique given the default `comparator = (a: T, b: T) => a === b` which can be overriden by giving another comparator function as the last argument on previous signatures
- `fc.tuple<T1,T2,...>(arb1: Arbitrary<T1>, arb2: Arbitrary<T2>, ...): Arbitrary<[T1,T2,...]>` tuple generated by aggregating the values of `arbX` like `generate: () => [arb1.generate(), arb2.generate(), ...]`. This arbitrary perfectly handle shrinks and is able to shink on all the generators
- `fc.dictionary<T>(keyArb: Arbitrary<string>, valueArb: Arbitrary<T>): Arbitrary<{[Key:string]:T}>` dictionary containing keys generated using `keyArb` and values generated by `valueArb`
- `fc.record<T>(recordModel: {[Key:string]: Arbitrary<T>}): Arbitrary<{[Key:string]: T}>` or `fc.record<T>(recordModel: {[Key:string]: Arbitrary<T>}, constraints: RecordConstraints): Arbitrary<{[Key:string]: T}>` record using the incoming arbitraries to generate its values. It comes very useful when dealing with settings. It takes an optional parameter of type `RecordConstraints` to configure some of its properties. The setting `withDeletedKeys=true` instructs the record generator that it can omit some keys
- `fc.infiniteStream<T>(arb: Arbitrary<T>): Arbitrary<Stream<T>>` infinite `Stream` of values generated by `arb`. The `Stream` structure provided by fast-check implements `IterableIterator<T>` and comes with useful helpers to manipulate it
- `fc.dedup<T>(arb: Arbitrary<T>, numValues: number)` tuple containing `numValues` instances of the same value produced by `arb` - values are independent from each others

## Objects (:any)

The framework is able to generate totally random objects in order to adapt to programs that do not requires any specific data structure. All those custom types can be parametrized using `ObjectConstraints.Settings`.

```typescript
export module ObjectConstraints {
    export interface Settings {
        maxDepth?: number;          // maximal depth allowed for this object
        maxKeys?: number;           // maximal number of keys (and values)
        key?: Arbitrary<string>;    // arbitrary for key
        values?: Arbitrary<any>[];  // arbitrary responsible for base value
        withBoxedValues?: boolean;  // adapt all entries within `values` to generate boxed version of the value too
        withMap?: boolean;          // also generate Map
        withSet?: boolean;          // also generate Set
        withObjectString?: boolean; // also generate string representations of object instances
        withNullPrototype?: boolean;// also generate string representations of object instances
    };
};
```

Default for `key` is: `fc.string()`.

Default for `values` are: `fc.boolean()`, `fc.integer()`, `fc.double()`, `fc.string()` and constants among `null`, `undefined`, `Number.NaN`, `+0`, `-0`, `Number.EPSILON`, `Number.MIN_VALUE`, `Number.MAX_VALUE` , `Number.MIN_SAFE_INTEGER`, `Number.MAX_SAFE_INTEGER`, `Number.POSITIVE_INFINITY` or `Number.NEGATIVE_INFINITY`.

- `fc.anything()` or `fc.anything(settings: ObjectConstraints.Settings)` generate a possible values coming from Settings and all objects or arrays derived from those same settings
- `fc.object()` or `fc.object(settings: ObjectConstraints.Settings)` generate an object
- `fc.jsonObject()` or `fc.jsonObject(maxDepth: number)` generate an object that is eligible to be stringified and parsed back to itself (object compatible with json stringify)
- `fc.unicodeJsonObject()` or `fc.unicodeJsonObject(maxDepth: number)` generate an object with potentially unicode characters that is eligible to be stringified and parsed back to itself (object compatible with json stringify)

## Recursive structures

- `fc.letrec(builder: (tie) => { [arbitraryName: string]: Arbitrary<T> })` produce arbitraries as specified by builder function. The `tie` function given to builder should be used as a placeholder to handle the recursion. It takes as input the name of the arbitrary to use in the recursion

```typescript
const { tree } = fc.letrec(tie => ({
  // tree is 1 / 3 of node, 2 / 3 of leaf
  // Warning: as there is no control over the depth of the data-structures generated
  //   by letrec, high probability of node can lead to very deep trees
  //   thus we limit the probability of a node to p = 1 / 3 in this example
  // with p = 0.50 the probability to have a tree of depth above 10 is 13.9 %
  // with p = 0.33 the probability to have a tree of depth above 10 is  0.6 %
  tree: fc.oneof(tie('node'), tie('leaf'), tie('leaf')),
  node: fc.tuple(tie('tree'), tie('tree')),
  leaf: fc.nat()
}));
tree() // Is a tree arbitrary (as fc.nat() is an integer arbitrary)
```

- `fc.memo<T>(builder: (n: number) => Arbitrary<T>): ((n?: number) => Arbitrary<T>)` produce arbitraries as specified by builder function. Contrary to `fc.letrec`, `fc.memo` can control the maximal depth of your recursive structure by relying on the `n` parameter given as input of the `builder` function

```typescript
const tree: fc.Memo<Tree> = fc.memo(n => fc.oneof(node(n), leaf()));
const node: fc.Memo<Tree> = fc.memo(n => {
  if (n <= 1) return fc.record({ left: leaf(), right: leaf() });
  return fc.record({ left: tree(), right: tree() }); // tree() is equivalent to tree(n-1)
});
const leaf = fc.nat;
tree() // Is a tree arbitrary (as fc.nat() is an integer arbitrary)
       // with maximal depth of 10 (equivalent to tree(10))
```

## Functions

- `compareBooleanFunc()` generate a comparison function taking two parameters `a` and `b` and producing a boolean value. `true` means that `a < b`, `false` that `a = b` or `a > b`
- `compareFunc()` generate a comparison function taking two parameters `a` and `b` and producing an integer value. Output is zero when `a` and `b` are considered to be equivalent. Output is strictly inferior to zero means that `a` should be considered strictly inferior to `b` (similar for strictly superior to zero)
- `func(arb: Arbitrary<TOut>)` generate a function of type `(...args: TArgs) => TOut` outputing values generated using `arb`

## Extended tools

- `context()` generate a `Context` instance for each predicate run. `Context` can be used to log stuff within the run itself. In case of failure, the logs will be attached in the counterexample and visible in the stack trace

## Model based testing

Model based testing approach extends the power of property based testing to state machines - *eg.: UI, data-structures*.

See section [Model based testing or UI test](./Tips.md#model-based-testing-or-ui-test) in Tips for an in depth explanation.

### Commands

The approach relies on commands. Commands can be seen as operations a user can run on the system. Those commands have:
- pre-condition - *implemented by `check`* - confirming whether or not the command can be executed given the current context
- execution - *implemented by `run`* - responsible to update a simplified context while updating and checking the real system

Commands can either be synchronous - `fc.Command<Model, Real>` - or asynchronous - `fc.AsyncCommand<Model, Real>` or  `fc.AsyncCommand<Model, Real, true>`.

```typescript
// Real : system under test
// Model: simplified state for the system
export interface Command<Model extends object, Real> {
  // Check if the model is in the right state to apply the command
  // WARNING: does not change the model
  check(m: Readonly<Model>): boolean;

  // Execute on r and perform the checks - Throw in case of invalid state
  // Update the model - m - accordingly
  run(m: Model, r: Real): void;

  // Name of the command
  toString(): string;
}

export interface AsyncCommand<Model extends object, Real> {
  check(m: Readonly<Model>): boolean;
  run(m: Model, r: Real): Promise<void>;
  toString(): string;
}

export interface AsyncCommand<Model extends object, Real, true> {
  check(m: Readonly<Model>): Promise<boolean>;
  run(m: Model, r: Real): Promise<void>;
  toString(): string;
}
```

### Arbitrary

While `fc.array` or any other array arbitrary could be used to generate such data, it is highly recommended to rely on `fc.commands` to generate arrays of commands. Its shrinker would be more adapted for such cases.

Possible signatures:
- `fc.commands<Model, Real>(commandArbs: Arbitrary<Command<Model, Real>>[], maxCommands?: number)` arrays of `Command` that can be ingested by `fc.modelRun`
- `fc.commands<Model, Real>(commandArbs: Arbitrary<Command<Model, Real>>[], settings: CommandsSettings)` arrays of `Command` that can be ingested by `fc.modelRun`
- `fc.commands<Model, Real>(commandArbs: Arbitrary<AsyncCommand<Model, Real>>[], maxCommands?: number)` arrays of `AsyncCommand` that can be ingested by `fc.asyncModelRun`
- `fc.commands<Model, Real>(commandArbs: Arbitrary<AsyncCommand<Model, Real>>[], settings: CommandsSettings)` arrays of `AsyncCommand` that can be ingested by `fc.asyncModelRun`

Possible settings:
```typescript
interface CommandsSettings {
  maxCommands?: number;       // optional, maximal number of commands to generate per run: 10 by default
  disableReplayLog?: boolean; // optional, do not show replayPath in the output: false by default
  replayPath?: string;        // optional, hint for replay purposes only: '' by default
                              // should be used in conjonction with {seed, path} of fc.assert
}
```

### Model runner

In order to execute the commands properly a call to either `fc.modelRun`, `fc.asyncModelRun` or `fc.scheduledModelRun` as to be done within classical runners - *ie. `fc.assert` or `fc.check`*.

### Simplified structure

```typescript
type Model = { /* stuff */ };
type Real  = { /* stuff */ };

class CommandA extends Command { /* stuff */ };
class CommandB extends Command { /* stuff */ };
// other commands

const CommandsArbitrary = fc.commands([
  fc.constant(new CommandA()),        // no custom parameters
  fc.nat().map(s => new CommandB(s)), // with custom parameter
  // other commands
]);

fc.assert(
  fc.property(
    CommandsArbitrary,
    cmds => {
      const s = () => ({ // initial state builder
          model: /* new model */,
          real:  /* new system instance */
      });
      fc.modelRun(s, cmds);
    }
  )
);
```

## Race conditions detection

In order to ease the detection of race conditions in your code, `fast-check` comes with a built-in asynchronous scheduler.
The aim of the scheduler - `fc.scheduler()` - is to reorder the order in which your async calls will resolve.

By doing this it can highlight potential race conditions in your code. Please refer to [code snippets](https://codesandbox.io/s/github/dubzzz/fast-check/tree/master/example?hidenavigation=1&module=%2F005-race%2Fautocomplete%2Fmain.spec.tsx&previewwindow=tests) for more details.

`fc.scheduler()` is just an `Arbitrary` providing a `Scheduler` instance. The generated scheduler has the following interface:
- `schedule: <T>(task: Promise<T>, label?: string) => Promise<T>` - Wrap an existing promise using the scheduler. The newly created promise will resolve when the scheduler decides to resolve it (see `waitOne` and `waitAll` methods).
- `scheduleFunction: <TArgs extends any[], T>(asyncFunction: (...args: TArgs) => Promise<T>) => (...args: TArgs) => Promise<T>` - Wrap all the promise produced by an API using the scheduler. `scheduleFunction(callApi)`
- `scheduleSequence(sequenceBuilders: SchedulerSequenceItem[]): { done: boolean; faulty: boolean, task: Promise<{ done: boolean; faulty: boolean }> }` - Schedule a sequence of operations. Each operation requires the previous one to be resolved before being started. Each of the operations will be executed until its end before starting any other scheduled operation.
- `count(): number` - Number of pending tasks waiting to be scheduled by the scheduler.
- `waitOne: () => Promise<void>` - Wait one scheduled task to be executed. Throws if there is no more pending tasks.
- `waitAll: () => Promise<void>` - Wait all scheduled tasks, including the ones that might be created by one of the resolved task. Do not use if `waitAll` call has to be wrapped into an helper function such as `act` that can relaunch new tasks afterwards. In this specific case use a `while` loop running while `count() !== 0` and calling `waitOne` - *see CodeSandbox example on userProfile*.

With:
```ts
type SchedulerSequenceItem =
    { builder: () => Promise<any>; label: string } |
    (() => Promise<any>)
;
```

### Scheduling methods

#### `schedule`

Create a scheduled `Promise` based on an existing one - _aka. wrapped `Promise`_.
The life-cycle of the wrapped `Promise` will not be altered at all.
On its side the scheduled `Promise` will only resolve when the scheduler decides it to be resolved.

Once scheduled by the scheduler, the scheduler will wait the wrapped `Promise` to resolve - _if it was not already the case_ - before sheduling anything else.

**Signature:**

```ts
schedule: <T>(task: Promise<T>) => Promise<T>
schedule: <T>(task: Promise<T, label: string>) => Promise<T>
```

**Usages:**

Any algorithm taking raw `Promise` as input might be tested using this scheduler.

For instance, `Promise.all` and `Promise.race` are examples of such algorithms.

**More:**

```ts
// Let suppose:
// - s        : Scheduler
// - shortTask: Promise   - Very quick operation
// - longTask : Promise   - Relatively long operation

shortTask.then(() => {
  // not impacted by the scheduler
  // as it is directly using the original promise
})

const scheduledShortTask = s.schedule(shortTask)
const scheduledLongTask = s.schedule(longTask)

// Even if in practice, shortTask is quicker than longTask
// If the scheduler selected longTask to end first,
// it will wait longTask to end, then once ended it will resolve scheduledLongTask,
// while scheduledShortTask will still be pending until scheduled.
await s.waitOne()
```

#### `scheduleFunction`

Create a producer of scheduled `Promise`.

Lots of our asynchronous codes make use of functions able to generate `Promise` based on inputs.
Fetching from a REST API using `fetch("http://domain/")` or accessing data from a database `db.query("SELECT * FROM table")` are examples of such producers.

`scheduleFunction` makes it possible to re-order when those outputed `Promise` resolve by providing a function that under the hood **directly** calls the producer but schedules its resolution so that it has to be scheduled by the scheduler.

**Signature:**

```ts
scheduleFunction: <TArgs extends any[], T>(asyncFunction: (...args: TArgs) => Promise<T>) => (...args: TArgs) => Promise<T>
```

**Usages:**

Any algorithm making calls to asynchronous APIs can highly benefit from this wrapper to re-order calls.

WARNING: `scheduleFunction` is only postponing the resolution of the function. The call to the function itself is started immediately when the caller calls something on the scheduled function.

**More:**

```ts
// Let suppose:
// - s             : Scheduler
// - getUserDetails: (uid: string) => Promise - API call to get details for a User


const getUserDetailsScheduled = s.scheduleFunction(getUserDetails)

getUserDetailsScheduled('user-001')
// What happened under the hood?
// - A call to getUserDetails('user-001') has been triggered
// - The promise returned by the call to getUserDetails('user-001') has been registered to the scheduler
  .then((dataUser001) => {
    // This block will only be executed when the scheduler
    // will schedule this Promise
  })

// Unlock one of the scheduled Promise registered on s
// Not necessarily the first one that resolves
await s.waitOne()
```

#### `scheduleSequence`

A scheduled sequence can be seen as a sequence a asynchronous calls we want to run in a precise order.

One important fact about scheduled sequence is that whenever one task of the sequence gets scheduled, **no other scheduled task in the scheduler can be unqueued** while this task has not ended. It means that tasks defined within a scheduled sequence must not require other scheduled task to end to fulfill themselves - _it does not mean that they should not force the scheduling of other scheduled tasks_.

**Signature:**

```ts
type SchedulerSequenceItem =
    { builder: () => Promise<any>; label: string } |
    (() => Promise<any>)
;

scheduleSequence(sequenceBuilders: SchedulerSequenceItem[]): { done: boolean; faulty: boolean, task: Promise<{ done: boolean; faulty: boolean }> }
```

**Usages:**

You want to check the status of a database, a webpage after many known operations.

Most of the time, model based testing might be a better fit for that purpose.

**More:**

```jsx
// Let suppose:
// - s: Scheduler

const initialUserId = '001';
const otherUserId1 = '002';
const otherUserId2 = '003';

// render profile for user {initialUserId}
// Note: api calls to get back details for one user are also scheduled
const { rerender } = render(
  <UserProfilePage userId={initialUserId} />
)

s.scheduleSequence([
  async () => rerender(<UserProfilePage userId={otherUserId1} />),
  async () => rerender(<UserProfilePage userId={otherUserId2} />),
])

await s.waitAll()
// expect to see profile for user otherUserId2
```

#### Missing helpers

**Scheduling a function call**

In some tests, we want to try cases where we launch multiple concurrent queries towards our service in order to see how it behaves in the context of concurrent operations.

```ts
const scheduleCall = <T>(s: Scheduler, f: () => Promise<T>) => {
  s.schedule(Promise.resolve("Start the call"))
    .then(() => f());
}

// Calling doStuff will be part of the task scheduled in s
scheduleCall(s, () => doStuff())
```

**Scheduling a call to a mocked server**

Contrary the behaviour of `scheduleFunction`, real calls to servers are not immediate and you might want to also schedule when the call _reaches_ your mocked-server.

Let's imagine you are building a TODO-list app. Your users can add a TODO only if no other TODO has the same label. If you use the built-in `scheduleFunction` to test it, the mocked-server will always receive the calls in the same order as the one they were done.

```ts
const scheduleMockedServerFunction = <TArgs extends unknown[], TOut>(s: Scheduler, f: (...args: TArgs) => Promise<TOut>) => {
  return (...args: TArgs) => {
    return s.schedule(Promise.resolve("Server received the call"))
      .then(() => f(...args));
  }
}

const newAddTodo = scheduleMockedServerFunction(s, (label) => mockedApi.addTodo(label))
// With newAddTodo = s.scheduleFunction((label) => mockedApi.addTodo(label))
// The mockedApi would have received todo-1 first, followed by todo-2
// When each of those calls resolve would have been the responsability of s
// In the contrary, with scheduleMockedServerFunction, the mockedApi might receive todo-2 first.
newAddTodo('todo-1') // .then
newAddTodo('todo-2') // .then

// or...

const scheduleMockedServerFunction = <TArgs extends unknown[], TOut>(s: Scheduler, f: (...args: TArgs) => Promise<TOut>) => {
  const scheduledF = s.scheduleFunction(f);
  return (...args: TArgs) => {
    return s.schedule(Promise.resolve("Server received the call"))
      .then(() => scheduledF(...args));
  }
}
```

### Wrapping calls automatically using `act`

`fc.scheduler({ act })` can be given an `act` function that will be called in order to wrap all the scheduled tasks. A code like the following one:

```js
fc.assert(
  fc.asyncProperty(fc.scheduler(), async s => () {
    // Pushing tasks into the scheduler ...
    // ....................................
    while (s.count() !== 0) {
      await act(async () => {
        // This construct is mostly needed when you want to test stuff in React
        // In the context of act from React, using waitAll would not have worked
        // as some scheduled tasks are triggered after waitOne resolved
        // and because of act (effects...)
        await s.waitOne();
      });
    }
  }))
```

Is equivalent to:

```js
fc.assert(
  fc.asyncProperty(fc.scheduler({ act }), async s => () {
    // Pushing tasks into the scheduler ...
    // ....................................
    await s.waitAll();
  }))
```

A simplified implementation for `waitOne` would be:

```js
async waitOne() {
  await act(async () => {
    await getTaskToBeResolved();
  })
}
async waitAll() {
  while (count() !== 0) {
    await waitOne();
  }
}
```

### Model based testing and race conditions

Model based testing capabilities can be used to help race conditions detection by using the runner `fc.scheduledModelRun`.

By using `fc.scheduledModelRun` even the execution of the model is scheduled using the scheduler.

One important fact to know when mixing model based testing with schedulers is that neither `check` nor `run` should rely on the completion of other scheduled tasks to fulfill themselves but they can - _and most of the time have to_ - trigger new scheduled tasks. No other scheduled task will be resolved during the execution of `check` or `run`.