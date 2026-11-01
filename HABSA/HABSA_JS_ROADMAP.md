# 🟨 to JavaScript Mastery


---

## What your Java notes already cover (Phase 1 inventory)

- **OOP**: classes, constructors, `this`/`super`, final, static, static/instance blocks, method & variable hiding, overloading vs overriding, covariant returns, interfaces + default/static methods, abstract classes, anonymous/inner classes, records, enums, sealed classes, diamond problem, composition vs aggregation, encapsulation, SOLID-flavored questions, Singleton/Factory/Builder/Strategy.
- **Collections & Generics**: List/ArrayList vs LinkedList, HashSet/LinkedHashSet/TreeSet, HashMap internals (buckets, load factor), LinkedHashMap/TreeMap, Queue/Deque, Iterator/ListIterator, Comparable vs Comparator, equals/hashCode contract, generic classes/methods, bounded types, wildcards, type erasure.
- **Java 8+**: lambdas, functional interfaces (Predicate/Function/Consumer/Supplier/Comparator...), method references, Streams (lazy, intermediate/terminal ops, reduce, Collectors, groupingBy, parallel streams), Optional, var, records, pattern matching.
- **Exceptions**: checked vs unchecked, throw/throws, try-catch-finally traps, custom exceptions, try-with-resources, suppressed exceptions.
- **Strings & values**: String immutability & pool, StringBuilder/StringBuffer, == vs equals, wrappers, autoboxing, Integer cache, pass-by-value, shallow vs deep copy.
- **Runtime/memory**: stack vs heap, references & GC, finalize, `.java`→`.class`→JVM/JIT, serialization/transient.
- **Tooling/Spring touch points**: Maven (POM, lifecycle, repositories), Spring DI/IoC, beans, lazy vs eager loading, JPA/JDBC basics, servlets.

---

# LEVEL 1 — Fundamentals (syntax, types, operators, control flow, functions)

### 1.1 var / let / const, Hoisting & TDZ
- **Java equivalent:** No equivalent (Java: declaration order strict, compile-time errors; closest analogy = uninitialized local variables being illegal).
- **Key difference:** `var` is function-scoped and hoisted with value `undefined`; `let`/`const` are block-scoped and hoisted but unusable before the declaration (Temporal Dead Zone) → ReferenceError. `const` binds the *reference*, not the object (like Java `final Dog d` — you can still mutate `d.name`). Default to `const`, use `let` only when reassigning; never use `var` in modern code.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Predict output of 5 snippets that reference `var`/`let`/`const` before assignment; rewrite each to avoid the trap.
- **Common interview question:** "What is the Temporal Dead Zone and how does it differ for `var` vs `let`?"

### 1.2 Primitive & reference types, typeof, null vs undefined, NaN
- **Java equivalent:** Partially covered — your notes on primitives vs wrappers, Integer cache, references vs objects (`56-Is Java Completely Object-Oriented?`, `78-Notes on reference and object`).
- **Key difference:** JS has 7 primitives (`number`, `string`, `boolean`, `undefined`, `symbol`, `bigint`, `null`) — **no int/long/double split, all numbers are IEEE 754 doubles** (`0.1 + 0.2 !== 0.3`). `typeof null === "object"` (historical bug), `typeof NaN === "number"`. Nothing is boxed unless you explicitly use wrapper objects (never do).
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Write a `safeDivide(a,b)` that detects NaN and Infinity from floating-point math.
- **Common interview question:** "What is the output of `typeof null`, `typeof NaN`, `typeof function(){}` and why?"

### 1.3 Type coercion, `==` vs `===`, truthy/falsy
- **Java equivalent:** Analog of `==` vs `.equals()` (`Sticky notes/== and .equals.md`) — but the mechanics are completely different.
- **Key difference:** `==` coerces ("1" == 1 → true, null == undefined → true), `===` does not. Falsy values: `false, 0, -0, "", null, undefined, NaN` — everything else (including `[]`, `{}`, `"0"`) is truthy, which shocks Java devs. Use `===` always, except `x == null` to check both null and undefined.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Build a small table of 20 `==` vs `===` comparisons and predict each before running.
- **Common interview question:** "Why does `[] == false` return true but `[]` is truthy in an if?"

### 1.4 Operators, template literals, short-circuit evaluation
- **Java equivalent:** Mostly covered — operators + logical operators appear in your notes (`11-SOme important Q.md`, `95-Composition vs Aggregation.md` embedded notes).
- **Key difference:** JS adds `**` (exponent), template literals with `${}`, ternary as expression, and `||`/`&&`/`??` return *values* (not booleans) — e.g. `name = user?.name ?? "guest"`. String `+` coercion: `"5" + 1 = "51"` but `"5" - 1 = 4`.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Rewrite 10 string-concatenation examples using template literals and `||`/`??` fallbacks.
- **Common interview question:** "What is the difference between `??` and `||`?"

### 1.5 Spread / rest & destructuring
- **Java equivalent:** No equivalent (closest: varargs `T...` from your Collections recap, and record unpacking).
- **Key difference:** Destructuring works on arrays AND objects (`const { name, age } = obj`), supports defaults/renaming/nesting; spread `[...arr]`, `{...obj}` makes shallow copies; rest `(...args)` collects params. Spread does **one level deep** only (shallow, like your deep/shallow copy notes).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Given a nested config object, extract/override 5 fields with destructuring + defaults in one statement.
- **Common interview question:** "How do you clone an object in JS, and why isn't `{...obj}` a deep clone?"

### 1.6 Control flow: if / switch / loops (for, for…of, for…in, while) + labels
- **Java equivalent:** Covered — `59-for vs while loop`, `94-break/continue/return`.
- **Key difference:** `for...of` iterates **values** of iterables (like enhanced-for); `for...in` iterates object **keys** (strings, includes inherited — don't use on arrays). `switch` matches with `===` (no fall-through bug when you always break). Labels let you break nested loops (same as Java).
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Convert a Java `for (Song s : songs)` loop into `for...of`, and iterate an object's key-value pairs properly.
- **Common interview question:** "What is the difference between `for...in` and `for...of`, and when does each fail?"

### 1.7 Functions: declarations, expressions, arrows, default/rest params, IIFE
- **Java equivalent:** Lambdas from your notes (`4-Lambdas.md`, `3-Functional Interfaces.md`) + anonymous inner classes (`77-Anonymous classes`).
- **Key difference:** JS functions are first-class objects — declared three ways (declaration → hoisted; expression → not hoisted; arrow → lexical `this`, no own `arguments`). Default params replace Java overloading: **JS has NO method overloading** — redeclaring a name overwrites the previous version.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Write the same utility 3 ways (declaration, arrow assigned to const, IIFE) and explain hoisting behavior of each.
- **Common interview question:** "JavaScript has no method overloading — how do you simulate optional parameters?"

---

# LEVEL 2 — Core JS (scope, closures, this, prototypes, coercion)

### 2.1 Execution context, scope chain, lexical environment
- **Java equivalent:** No equivalent (closest: method call stack frames from your stack/heap notes `74-Stack & Heap`).
- **Key difference:** Every function call creates an execution context with its own scope; inner functions keep a live reference to the outer scope (lexical scoping determined by *where code is written*, not where it's called). Extends your "stack frames" mental model to nested function references.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Draw the scope chain for a 3-level nested function accessing 4 variables at different levels.
- **Common interview question:** "What is lexical scope and how does it differ from dynamic scope?"

### 2.2 Hoisting & TDZ (deep: functions vs variables)
- **Java equivalent:** No equivalent.
- **Key difference:** Function *declarations* are fully hoisted (callable before definition); function *expressions* and `let/const` are not; `var` gives `undefined`. Class declarations are hoisted but in TDZ.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Write a file where a function is called before its declaration, then "fix" it 3 different ways and trace each.
- **Common interview question:** "Explain the output of calling a hoisted function that uses a `let` variable declared later."

### 2.3 Closures
- **Java equivalent:** Partially covered — lambdas capturing effectively-final variables (`1-Functional Programming.md`).
- **Key difference:** JS closures capture *variables* (mutable references), not copied values — a Java lambda can't mutate the captured local; a JS closure can and does see subsequent changes. Closures power: private state, factories, memoization, debounce/throttle, module pattern. They are also a memory-leak source (GC keeps captured scope alive).
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Implement `createCounter()` and a `memoize(fn)` using closured state — no classes.
- **Common interview question:** "What does the classic loop-with-`var`-and-`setTimeout` print, and how do `let` / an IIFE fix it?"

### 2.4 `this` binding + call / apply / bind
- **Java equivalent:** No equivalent (Java `this` is always the current instance — never rebound).
- **Key difference:** `this` is decided by **how a function is called**, not where it's defined: (1) `obj.fn()` → obj, (2) bare `fn()` → undefined (strict) / global, (3) `new` → new instance, (4) arrow → lexical (outer) `this`, (5) `call`/`apply` (spread args) /`bind` (returns new function) → explicit. This is the #1 JS interview topic.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Take an object method, call it 5 ways (direct, detached, call, bind, arrow) and predict `this` each time.
- **Common interview question:** "Explain all the rules that determine `this`, and how arrow functions behave differently."

### 2.5 Prototypes & prototype chain (vs Java classes)
- **Java equivalent:** Conceptually replaces Java classes/inheritance (`7-super` notes), but mechanically opposite.
- **Key difference:** JS has no classes at runtime — objects link to a prototype object; property lookup walks the chain (like walking up Java superclasses, but dynamic and mutable). `class` syntax desugars to prototype functions. `instanceof` checks the prototype chain (vs your `instanceof vs getClass` notes). `Object.create`, `__proto__`, `getPrototypeOf` are the raw tools.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Build a small inheritance chain with `Object.create` only (no `class`), then reimplement it with `class` and map the pieces.
- **Common interview question:** "How does `instanceof` work internally, and how do prototypes make inheritance different from Java?"

### 2.6 Memory model: stack/heap, garbage collection & leaks
- **Java equivalent:** Strongly covered — `74-Stack & Heap`, `79-Memory Management & GC`, `80-Garbage Collector`, `micro 1/1.1`.
- **Key difference:** Reference semantics and reachability-based GC feel identical to Java. Differences: JS GC is per-engine (V8 generational too) but **closures and DOM/timer/event listeners are leak sources Java devs don't expect**; no `finalize()` equivalent (your `Finalize().md` notes — deprecated in Java too).
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Write a closure + accumulated-cache example that leaks; find it with Chrome DevTools heap snapshot.
- **Common interview question:** "How can closures or forgotten event listeners cause memory leaks in JS?"

### 2.7 WeakMap / WeakSet & object identity
- **Java equivalent:** Partially covered — weak/strong references analogy from GC notes; `IdentityHashMap` concept.
- **Key difference:** Keys are held weakly (GC removes entries when the key dies) — like Java's `WeakHashMap`. Object/array/function comparisons are by **reference** (`===`), never by content — you must write deep-equal yourself (adjacent to your `equals/hashCode` notes).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Attach per-object metadata to DOM nodes via WeakMap without preventing GC.
- **Common interview question:** "When would you use a WeakMap instead of Map?"

---

# LEVEL 3 — OOP in JS (classes, inheritance, encapsulation, mixins)

### 3.1 ES6 classes & instances
- **Java equivalent:** Covered — OOP recap, classes/objects/constructors (`RECAP ON CORE.md`, `Interview_Simulation_OOP.md`).
- **Key difference:** `class` is syntactic sugar over prototypes — methods are non-enumerable prototype properties (they're shared instances like Java static-ish behavior), there is no compile-time type checking, and classes are not hoisted like function declarations.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Port your Java `BankAccount` encapsulation example (from `test.md` notes) to a JS class.
- **Common interview question:** "Is a JS class the same as a Java class? What does `class` compile to?"

### 3.2 Inheritance, extends & super
- **Java equivalent:** Covered — your `super` deep-dives (`7-`, `8-`, `8.1-super`, `5-Super`s.md`).
- **Key difference:** Single inheritance only (same as Java); `super(...)` must be called before using `this` in a constructor; no interfaces between parent/child — subclasses decide structure dynamically; parent methods are looked up at runtime on the prototype chain.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Port the `Vehicle → Bike` hierarchy you traced in variable-hiding notes to JS classes.
- **Common interview question:** "When is `super()` required in a subclass constructor and what happens without it?"

### 3.3 Encapsulation: `#private` fields & closure privacy
- **Java equivalent:** Covered — `87-Encapsulation is not just getters/setters`.
- **Key difference:** Two styles: `#field` (true runtime privacy) and closure-captured variables (module-level privacy). No access modifiers (public/protected don't exist); everything is public unless `#` or closure-hidden. Your "encapsulation = controlled behavior" lesson translates directly.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Implement `Account` with a `#balance` that only `deposit/withdraw` can touch (validate negative amounts).
- **Common interview question:** "How do you implement private members in JavaScript — two ways?"

### 3.4 Constructors, `new`, initialization order
- **Java equivalent:** Covered — constructor chaining, `this()`/`super()` rules, static vs instance blocks (`8-`, `31-`, `32-`, `97-`).
- **Key difference:** One constructor per class (no overloading — use default params); `new` does 4 steps: create object → set prototype → bind `this` → run constructor (return object unless you explicitly return an object). No static initializer blocks; static fields initialize at class definition.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Log step-by-step what `new Person("Ali")` does internally with a small prototype experiment.
- **Common interview question:** "What exactly does the `new` keyword do in JavaScript?"

### 3.5 Static members & "method hiding" analogy
- **Java equivalent:** Covered — your static-method hiding notes (`9-`, `83-`, `Item 33`).
- **Key difference:** `static` methods/fields live on the constructor function itself (inherited via the chain — `Child.staticInherited()` works, unlike Java's non-inherited statics). Beware: a static method calling `this` gets the *class*, and instance shadows in subclasses behave differently than Java hiding.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Add a static factory + static counter to a class; access it from a subclass and from an instance.
- **Common interview question:** "Are static methods inherited in JavaScript classes?"

### 3.6 Polymorphism & dynamic dispatch
- **Java equivalent:** Covered — overriding vs overloading, compile vs runtime polymorphism (`25-`, `1-`, `Item 9.1`).
- **Key difference:** JS is always dynamically dispatched — every method call goes through the prototype chain (no static dispatch, no overloading). Duck typing replaces compile-time polymorphism: "if it quacks" — shape-based, no type declarations.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Create `Dog`/`Bird` classes with `makeSound()` and call it through an array of mixed instances.
- **Common interview question:** "How does polymorphism work in JavaScript without inheritance?"

### 3.7 Interfaces & abstract classes → duck typing, mixins, TS interfaces
- **Java equivalent:** Partially covered — abstract vs interface, default methods, diamond conflict (`20-Java 8 interface`, `2 DEF methods`, `76-`, `77-`).
- **Key difference:** JS has **no interfaces and no abstract classes at runtime**. Substitutes: duck typing + JSDoc/TypeScript interfaces (compile-time only), mixins via `Object.assign(Class.prototype, {...})` or class expression factories. The diamond problem has no state conflict in JS (single inheritance), so mixin conflicts are resolved by overwrite order instead.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Build a `canFly`/`canSwim` mixin set and compose a `Duck` class from both.
- **Common interview question:** "JavaScript has no interfaces — how do you enforce contracts then?"

### 3.8 Records & enums equivalents
- **Java equivalent:** Covered — `7-records.md`, `73-Records`, `Sticky notes/ENUMS.md`.
- **Key difference:** No records: closest are plain object literals + `Object.freeze()` + `structuredClone()` for copy semantics (still no auto equals/hashCode — write a deepEqual). No true enums: use frozen objects or `Object.freeze({SMALL:0,...})` (often with TS `enum`).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Build an immutable `Point` record-style factory + `assertEquals`-style deep comparison helper.
- **Common interview question:** "How do you create truly immutable objects in JavaScript, and what does Object.freeze NOT protect?"

### 3.9 Composition, DI & factories in JS
- **Java equivalent:** Covered — `8-DI vs Composition`, `9-Composition with DI`, `95-Composition vs Aggregation`.
- **Key difference:** JS calls functions, passes deps as plain args + factories (no annotation framework unless NestJS). Constructor injection = passing deps to `new`; higher-order functions replace interface-based pluggability perfectly.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Refactor a `ReportService` that hard-instantiates a `PdfGenerator` into one that receives it.
- **Common interview question:** "How would you implement dependency injection in plain JavaScript without a framework?"

### 3.10 Design patterns in JS (Singleton, Factory, Builder, Strategy, Module)
- **Java equivalent:** Covered — Singleton/Factory/Builder/Strategy (`micro 1/1.1`, `Interview_Simulation_OOP`).
- **Key difference:** Singleton = module pattern or closure (modules are cached singletons in Node); Strategy = passing functions instead of interface impls; Builder → object spread/defaults; Observer → EventEmitter (Node) / DOM events.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Implement module-pattern `BasketAPI` exposing only `add` and `total`, hiding the array.
- **Common interview question:** "How does the module pattern implement the Singleton pattern?"

---

# LEVEL 4 — Data structures & built-ins (Array, Object, Map, Set, iterators, generators)

### 4.1 Arrays (≈ List/ArrayList)
- **Java equivalent:** Strongly covered — `Linked List and Array List.md`, `64-Arrays vs List`.
- **Key difference:** Arrays are objects, resizable, heterogeneous, sparse-allowed (`a[100]=1` leaves holes; `length` is highest index + 1). No `addAll`; mutating methods are common — `sort()`, `splice()`, `reverse()`, `push()` **mutate in place** (huge Java-mode trap).
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Implement `ArrayList`-like `add/removeAt/get` on top of a plain array; note the cost model vs your Java Big-O table.
- **Common interview question:** "Why is `arr.sort()` on numbers wrong by default?"

### 4.2 Array methods map / filter / reduce (Java Streams pipeline)
- **Java equivalent:** Strongly covered — entire Streams (`Explain.md`, `1-Functional Programming`, `collctions.md` Streams sections).
- **Key difference:** `map/filter/reduce/flatMap/find/first(find x findIndex)/some/every/forEach` are **eager** (evaluate the whole array immediately; no laziness/chaining optimization like `.filter().limit(1)`, no short-circuit mid-pipeline). Streams also had Optional returns and infinite sources — JS has neither; `reduce` takes an initial value.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Re-implement 10 of your Java Stream exercises using array methods (or generators for laziness).
- **Common interview question:** "Difference between `map` and `forEach`? Between `find` and `filter`?"

### 4.3 Mutation vs immutability: mutating vs non-mutating array APIs
- **Java equivalent:** Partially covered — immutability emphasis (`List.of`, records, String immutability).
- **Key difference:** Mutators: push/pop/shift/unshift/splice/sort/reverse/fill. Non-mutating (ES2023): `toSorted/toReversed/toSpliced/with`, plus `concat/slice/flat/map/filter`. Rule of functional style: **never mutate shared state** (your FP notes already preach this).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Sort an array of objects by two keys without mutating the original (twice: ES2023 and pre-ES2023).
- **Common interview question:** "Which array methods mutate the original array?"

### 4.4 Plain Objects as records; Object.keys/values/entries
- **Java equivalent:** Partial — Map usage in your notes, records.
- **Key difference:** Objects are hash-maps with string/symbol keys of insertion-ish order; key order quirks (numeric-like keys first). `hasOwnProperty` vs `in` (prototype chain, links to Level 2).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Implement `wordCount(text)` with a plain object, then with a Map; compare ergonomics.
- **Common interview question:** "How do you safely iterate an object's entries in JS?"

### 4.5 Map / Set (≈ HashMap/HashSet)
- **Java equivalent:** Strongly covered — `Map Interface.md`, `65-Dictionary`, `69-HashMap internals`, HashSet notes.
- **Key difference:** `Map` keeps **insertion order** always (no need for LinkedHashMap), keys can be **any value** (functions, NaN, objects — no hashCode needed; equality is SameValueZero), methods are `get/set/has/delete` (no `compute/merge` yet). `Set` = unique values with same equality; no sorting via comparator (sort by making an array).
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Implement `groupBy(items, keyFn)` returning a Map; add uniqueness/dedup with Set.
- **Common interview question:** "When is a Map better than an object as a dictionary?"

### 4.6 Iterators & iterables (Symbol.iterator)
- **Java equivalent:** Covered — `Iterators.md`, `ListIterator cursor mechanics`.
- **Key difference:** Protocol-based: any object with `[Symbol.iterator]() { return { next() { return {value, done} } } }` works with `for...of`, spread, destructuring (your cursor model maps 1:1). No `ConcurrentModificationException` — mutation during iteration just produces surprising results instead of failing fast.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Make a `range` object iterable; add it to a custom class so `for...of` and spread work.
- **Common interview question:** "What is the iterator protocol and why does spread work on Maps but not plain objects?" (spread on `{}` works from ES2018 rest/spread for objects, but not via iterator).

### 4.7 Generators (function*)
- **Java equivalent:** Partial — infinite/lazy streams and lazy evaluation from your notes (closest cousin).
- **Key difference:** `function*` + `yield` pauses/resumes execution — perfect for laziness (your "Stream is lazy" notes) and infinite sequences, but blue-collar: linear pull-based, no parallelism, no collect built-ins.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Build infinite `fibonacci` generator + `take(n, iter)` helper; compare with Java `Stream.iterate`.
- **Common interview question:** "What does `yield` do and how does a generator maintain its state between calls?"

### 4.8 Sorting & comparators (CustomComparator)
- **Java equivalent:** Covered — Comparable vs Comparator, `thenComparing` (`RECAP ON CORE`, Collections notes).
- **Key difference:** `arr.sort(cmp)`: negative/0/positive convention instead of Java's same signed-int contract; default sort is **]string comparison** (`[10,9,1].sort() → [1,10,9]` chaos). Chaining instead of `thenComparing`: `(a,b)=> r || r2`.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Sort products by price asc, then name desc, without mutating the source array.
- **Common interview question:** "What is the default sort behavior and how would you sort object arrays by multiple keys?"

### 4.9 Strings, template literals, regex
- **Java equivalent:** Partially covered — String methods, StringBuilder, immutability & pool notes.
- **Key difference:** Immutable like Java (good); but no StringBuilder — build with array+`join` or template literals; **no String.format**; regex is a first-class literal `/pattern/flags`; full Unicode-aware `[...str]` iteration (surrogate pairs).
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Implement `pad`, `truncate`, `slugify` with template literals + a small regex.
- **Common interview question:** "How do you reverse a string containing emoji correctly?"

### 4.10 JSON (parse / stringify / structures)
- **Java equivalent:** No equivalent in notes (Jackson is the analog, but not covered).
- **Key difference:** `JSON.stringify` drops `undefined`, functions, `Map/Set` (serialize manually), cycles → error (use replacer or libraries); `JSON.parse` throws SyntaxError — always try/catch (links to Level 6 error handling). Deep clone trick (clone only JSON-safe data).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Serialize an object containing Map/Set/Date/undefined and document every loss; roundtrip safely.
- **Common interview question:** "What breaks when you JSON.stringify a Map, Set, or Date?"

### 4.11 Date, Math, BigInt, Symbol
- **Java equivalent:** Partial — Math.random/parse notes; no java.time equivalent covered in JS terms.
- **Key difference:** `Date` is mutable-ish and timezone-hostile (use Temporal API or date-fns in real projects); `Math.random()` insecure (crypto.getRandomValues for secrets); `BigInt` for >2^53; `Symbol` for unique keys (links to iterators).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Format `income: ${salary}` money strings with Intl.NumberFormat for 2 locales.
- **Common interview question:** "Why do we need BigInt, and what are JavaScript's integer limits?"

---

# LEVEL 5 — Asynchronous JS (event loop, promises, async/await)

### 5.1 Event loop, call stack, microtasks vs macrotasks
- **Java equivalent:** No equivalent (closest concept: your JVM-thread & GC knowledge for scheduling intuition; TaskExecutor vibes).
- **Key difference:** **Single-threaded** — one call stack; async work is queued. Order: synchronous code → all microtasks (promises, queueMicrotask) → one macrotask (timers, I/O, UI events) → repeat. Rendering sits between macrotasks. This is the classic "predict console output" interview goldmine.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Predict output of mixed `setTimeout`/`Promise.then`/`sync` snippets — then verify in Node.
- **Common interview question:** "What is the event loop and what is the execution order between sync code, promises and setTimeout?"

### 5.2 Callbacks (old-school async)
- **Java equivalent:** No equivalent (you've seen `Runnable`/listeners, but callback-hell style async only exists in JS).
- **Key difference:** Callbacks flatten control flow (error-first convention `(err, result)`); nesting callbacks creates "callback hell" — the historical problem Promises solve.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Write `loadProfile(id, cb)` style code, refactor it into a Promise-returning version.
- **Common interview question:** "What is callback hell and how did Promises solve it?"

### 5.3 Promises
- **Java equivalent:** Partial analog — `CompletableFuture` mentioned in `micro 1/1-PHASE 1.md`; overall: new topic.
- **Key difference:** States pending → fulfilled/rejected (immutable once settled, like a SettableFuture); `then` returns a **new** Promise each time (chainable, like `.thenApply`/`.thenCompose`). Error propagation flows through the chain until a `catch` (like exception propagation). No checked-exceptions ceremony — everything is effectively unchecked.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Implement `retry(fn, attempts)` chaining promises recursively.
- **Common interview question:** "Explain the Promise lifecycle and how `.then().catch().finally()` chain errors."

### 5.4 async/await
- **Java equivalent:** Analog of sequential blocking-style code + CompletableFuture.get, but **non-blocking**.
- **Key difference:** `async` functions always return Promises; `await` suspends the function (not the thread — there's no thread block) and reschedules it as a microtask. Error handling: try/catch inside async fn; unhandled rejection = uncaught error (crash in Node ≥15). Loops: `await` in `for...of` is sequential — `forEach` does NOT await.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Rewrite a callback chain into async/await with proper try/catch; run 3 awaits both sequentially and via `Promise.all`, compare timing.
- **Common interview question:** "Does `await` block the thread? How does async/await relate to the event loop?"

### 5.5 Promise combinators: all / allSettled / race / any
- **Java equivalent:** Partial — CompletableFuture.allOf/anyOf from `micro 1` notes.
- **Key difference:** `all` fails fast (short-circuit like Streams), `allSettled` gathers results, `race` first settled wins, `any` first **fulfilled** wins. No `groupingBy`-like helper — build fan-out/fan-in yourself.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Fetch 3 URLs: (a) all-or-nothing, (b) all results incl. failures, (c) fastest response only.
- **Common interview question:** "When do you use Promise.all vs Promise.allSettled vs Promise.race?"

### 5.6 Async error handling & unhandled rejections
- **Java equivalent:** Partial analog — your checked/unchecked, `@ControllerAdvice` global handler notes (`39-`, `micro 1 PHASE 1 §5`).
- **Key difference:** Rejected promises without handlers can vanish silently in the browser / crash Node; try/catch works inside async but NOT across callbacks; error handling of `then` callbacks needs explicit `catch` — like Spring's global exception handler but manual.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Trigger an unhandled rejection in Node, observe behavior, then add `process.on('unhandledRejection')` and per-call `catch`.
- **Common interview question:** "How do you handle errors in async code — three patterns?"

### 5.7 Timers: setTimeout / setInterval, debounce & throttle (event scheduling)
- **Java equivalent:** No equivalent (ScheduledExecutorService is the fuzziest analog).
- **Key difference:** Timers are macrotask events scheduled ≥ delay (not exact); `setTimeout(fn,0)` still defers to after microtasks — great event-loop demo. `debounce` (fire after silence) and `throttle` (fire at most every X ms) via closures — practical UI pattern.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Implement `debounce` and `throttle` from scratch with closures; stress-test in a sandbox.
- **Common interview question:** "Implement debounce in an interview — explain the closure state you keep."

### 5.8 Web Workers / concurrency model
- **Java equivalent:** Analog of `Thread`/`ExecutorService` from `micro 1/1-PHASE 1.md` — but message-passing instead of shared memory.
- **Key difference:** No shared mutable state between workers (structured-clone messages; SharedArrayBuffer is an advanced exception) — no `synchronized`/`volatile` needed; main thread stays free. Cluster/process model mirrors Node's `worker_threads`/child processes.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Offload an Array-sum of 10M numbers to a Worker; compare UI responsiveness.
- **Common interview question:** "How can JavaScript do parallel work if it is single-threaded?"

---

# LEVEL 6 — Modern JS (ES6+, modules, error handling, functional programming)

### 6.1 Modern syntax tour: optional chaining, nullish coalescing, object shorthand
- **Java equivalent:** Partial analog — Optional API from your notes.
- **Key difference:** `a?.b?.[c]?.()`, `x ?? def`, `{ name }` shorthand, computed keys — all null-safe and terse (Optional ergonomics but baked into syntax). Java's `Optional` wore you out; here it's just `?.` and `??`.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Refactor a deeply-nested config access exercise from `if != null` pyramids into `?.` + `??`.
- **Common interview question:** "How does optional chaining behave when the method itself is missing?"

### 6.2 Modules: ESM vs CommonJS
- **Java equivalent:** Partial analog — `MAVEN` notes on JPMS / module-info.java (compile-time vs JS runtime modules).
- **Key difference:** Two competing systems: CommonJS `require/module.exports` (Node legacy, synchronous) vs ESM `import/export` (native, hoisted, async, tree-shakable). Node executes lazily per-require; ESM bindings are **live** (re-read current value). Cycles behave differently — biggest tooling headache.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Build a 3-file mini-project in both CommonJS and ESM; import one into the other in Node.
- **Common interview question:** "What are the differences between CommonJS and ES modules — hoisting, cycles, bundling?"

### 6.3 Error handling & custom Error classes
- **Java equivalent:** Covered — exception hierarchy, checked vs unchecked, custom exceptions, suppressed, finally traps (`12-`, `39-`, `84-`, `micro 1 §4`, `Finally Block.md`).
- **Key difference:** Only one `Error` root; `throw` anything but only Error is useful; **no checked exceptions ever**; `try/catch/finally` with no `throws` declarations; extensions: `class AppError extends Error { constructor(msg, code){ super(msg); this.code=code; } }`. `finally` runs even on `return` (you know these traps from Java — same semantics, no `throws`).
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Port your `ProductNotFoundException` + global handler idea into a JS `AppError` hierarchy and a `handle(fn)` wrapper.
- **Common interview question:** "When does `finally` run in JavaScript and what happens if you return inside it?"

### 6.4 FP in JS: first-class functions, higher-order, currying, compose
- **Java equivalent:** Covered — entire functional programming suite (`1-`, `3-`, `4-`, method references).
- **Key difference:** Everything your lavaava notes taught maps directly (functions as values, compose/pipe via manual code, no `andThen`/`compose` on plain functions). Closures are stronger here (mutable capture — Level 2). No generics-typed interfaces — just plain signatures + JSDoc.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Implement `pipe`, `curry`, `memoize`; build a 3-stage data-processing pipeline with them.
- **Common interview question:** "Explain currying and show a practical use case."

### 6.5 Immutability patterns & pure data flow (≈ records/streams style)
- **Java equivalent:** Covered — immutability from records, `List.of`, String pool, FP notes.
- **Key difference:** No language-level support: `Object.freeze` (shallow!), spread copies, `structuredClone` (deep, incl. Dates/Maps), or `JSON.parse(JSON.stringify())` (only JSON-safe). Immutability is a **convention**, not compiler-enforced → discipline required.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Write `updateUser(users, id, patch)` returning a new array + new object (no mutations).
- **Common interview question:** "How do you update an item inside a nested state object without mutating anything?"
  (adjacent: mentions of const vs mutable)

### 6.6 Lazy evaluation & pipelines (≈ Streams partition/groupBy alternatives)
- **Java equivalent:** Covered — stream laziness, Collectors (`Stream is lazy.md`).
- **Key difference:** Instant ad-hoc pipelines via chained array funcs; for real laziness/infinite data → generators (Level 4.7); no `groupingBy/partitioningBy` — either ES proposal `Object.groupBy` (origin trials soon-ish) or roll your own `reduce`.
- **Status:** `[x]` (related concepts in notes)
- **Practice task:** Implement `groupingBy` and `partitioningBy` yourself via reduce; compare with your Java Collector versions.
- **Common interview question:** "How would you implement the equivalent of Java Stream's Collectors.groupingBy in JS?"

### 6.7 RegExp details, Date/Intl deep-dive
- **Java equivalent:** Partial (regex only mentioned; nothing deep in notes).
- **Key difference:** JS regex literal syntax + named capture `(?<year>\d{4})`, lookarounds; `Intl.*` for locale formatting — document-adjacent to your Java `NumberFormat`/`SimpleDateFormat` world but modernized.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Parse logs `2026-01-05 ERROR ...` with named groups; format with Intl.DateTimeFormat.
- **Common interview question:** "How do you extract named capture groups from a regex match in JS?"

### 6.8 Proxy / Reflect (metaprogramming, optional advanced)
- **Java equivalent:** Partial analog — your Spring proxies/AOP notes (`18-Spring proxy.md`) conceptually.
- **Key difference:** `new Proxy(target, { get, set, ... })` — intercept a la aspects; powers reactivity (Vue 3) and validation at object boundary. Not needed day-1, big bonus in interviews.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Create a Proxy that logs every property get/set on an object (mini-AOP trace).
- **Common interview question:** "What is a Proxy in JS — mention a real-world library using it." (Vue 3 reactivity)

---

# LEVEL 7 — Browser & Node.js essentials (DOM, events, fetch, npm)

### 7.1 DOM: selecting, creating, updating nodes
- **Java equivalent:** No equivalent (closest: JSP/Thymeleaf server-rendering knowledge from Servlet notes `55-`).
- **Key difference:** `document.querySelector`, `createElement`, `append`, `classList`, dataset attrs; DOM is a live tree — every change can trigger reflow; batch changes for perf. Your DispatcherServlet mental model shifts: now the *client* owns rendering.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Build a todo-list widget with add/remove/complete using vanilla DOM APIs only.
- **Common interview question:** "What is the difference between `textContent` and `innerText`?" (rendering awareness)

### 7.2 Events: bubbling, capturing, delegation, preventDefault
- **Java equivalent:** No equivalent (Swing/JSF listeners vague; nothing in notes).
- **Key difference:** Events propagate capturing (top-down) → target → bubbling (bottom-up); `stopPropagation` cuts flow, `preventDefault` cancels default action (distinct!); event delegation = one listener on a parent checking `event.target` (your Observer-pattern knowledge, applied with real propagation semantics).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** One listener on `<ul>` handling clicks for 100 dynamic `<li>`s via delegation.
- **Common interview question:** "What is event delegation and why is it more performant?"

### 7.3 Storage: localStorage / sessionStorage / cookies
- **Java equivalent:** Partial analog — server-side session/JPA persistence knowledge; browser storage is client-side.
- **Key difference:** Same-origin, string-only keys/values (serialize JSON); `localStorage` persists forever, `sessionStorage` per-tab; cookies ride on every request → auth flows (relates JWT from your Spring Security notes: where does the token live?).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Persist a draft todo list across reloads with `localStorage` + JSON; handle corrupted data.
- **Common interview question:** "localStorage vs cookies — when would you use each, especially for auth tokens?"

### 7.4 fetch / HTTP from the browser + Node (axios-like)
- **Java equivalent:** Partial analog — `JDBC`/REST/consuming APIs from Spring notes (`48-JDBC`, REST in `micro 1/0-interview topics.md`), but HTTP client side is new territory.
- **Key difference:** `fetch` returns a Promise that resolves on **headers**; non-2xx responses do NOT reject → check `res.ok` manually (top gotcha); `await res.json()` for body; mirror of `WebClient`/`RestTemplate` in Java.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Fetch from a public API; handle 404, timeouts (AbortController), and network errors distinctly.
- **Common interview question:** "Does fetch reject on a 404? How do you handle non-2xx responses properly?"

### 7.5 Browser runtime model: window, BOM, same-origin policy
- **Java equivalent:** No equivalent.
- **Key difference:** Global object = `window` (browser) vs `global`/`module scope` (Node); `document`, `location`, `history`, timers are all globals; same-origin policy blocks cross-site reads (security mental model connects to your Spring Security CORS notes).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Log every global you didn't declare in a blank page; explain 5 of them.
- **Common interview question:** "What is the same-origin policy and why does it exist?"

### 7.6 npm, package.json, node_modules, scripts
- **Java equivalent:** Covered — Maven (`POM.xml.md`, `Explaination.md`, lifecycle).
- **Key difference:** `package.json` ≈ pom.xml; `dependencies` vs `devDependencies`; **lockfile is checked in** (package-lock.json ≈ mvn lock); `node_modules` is not committed (unlike checking jars); scripts replace Maven phases (`npm run dev/test/build`); semantic versioning `^`/`~`; multiple registries (npmjs).
- **Status:** `[x]` (Maven analog covered)
- **Practice task:** Init a project, add lodash as dep + jest as devDep, add a `start` script; explain each generated file.
- **Common interview question:** "What's the difference between dependencies, devDependencies, and why commit the lockfile?"

### 7.7 Node.js basics: runtime, fs, process, streams
- **Java equivalent:** Partial analog — JVM runtime model from your `.class`/JIT/GC notes; OS-file access similar to Java IO (subject not covered).
- **Key difference:** Same V8 engine outside the browser: no DOM/window, adds `fs`, `process.env` (like Spring's `@Value` env props), modules; async-by-default I/O; `node script.js` replaces `java Main`.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Write a CLI script that reads a `.md` file and prints word frequency (uses fs + reduce).
- **Common interview question:** "Is Node.js single-threaded? What about its file I/O then?"

### 7.8 Debugging: DevTools, breakpoints, sources panel
- **Java equivalent:** Partial analog — your logging/Debugging folder experience, IDE-debugger habits.
- **Key difference:** Chrome DevTools panel (Elements/Console/Sources/Network); `debugger;` statement; `console.log` levels / `table`; Node `--inspect` connects the same DevTools.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Fix a deliberately broken debounce implementation using only breakpoints (no console.log).
- **Common interview question:** "How would you debug an error thrown only in production?"

---

# LEVEL 8 — Interview preparation (common questions, coding challenges, pitfalls)

### 8.1 Output-prediction drills: hoisting / closures / this / prototypes
- **Java equivalent:** Analog of your variable-hiding, Integer-cache, method-hiding trick files (`103-`, `52-`, `49-`).
- **Key difference:** The JS versions of those same trap genres: `this` in callbacks, hoisting order, closure-in-loop, prototype shadowing.
- **Status:** `[x]` (same genre as your Java trick notes)
- **Practice task:** Take 20 classic JS quiz snippets (e.g., from "JS Questions" repos) — predict, run, log each mistake.
- **Common interview question:** "Predict the output: function declared after use + let variable + arrow `this`."

### 8.2 Coercion & equality deep-question set
- **Java equivalent:** Analog of `==/equals` and `Integer cache` trap notes.
- **Key difference:** JS adds abstract-equality table, `Object.is`, `[1] + [2] = "12"`-style nonsense.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Build flashcards for 20 `==`/`===`/`Object.is` results; explain each rule used.
- **Common interview question:** "`null == undefined`? `NaN === NaN`? `Object.is(NaN,NaN)`? Explain."

### 8.3 Event-loop & async ordering questions
- **Java equivalent:** No equivalent (concurrency lore in Java covers threads, not this model).
- **Key difference:** Classic "print numbers of setTimeout vs promise chain" ordering puzzles.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Write 5 ordering puzzles combining sync/microtask/macrotask; verify in Node then explain each line.
- **Common interview question:** "What is printed by this mixed setTimeout/Promise resolution snippet?"

### 8.4 Classic coding challenges (implement from scratch)
- **Java equivalent:** Partial analog — you did Big-O/collections implementations mentally in Java; different twsits here due to closures/async.
- **Key difference:** No `Collections.*` utility crutch — the interviewer wants closures/prototypes/async fluency.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Implement, in <20 lines each: `debounce`, `throttle`, `deepClone` (JSON-safe), `flatten` (recursive), `groupBy`, `memoize`, `once`, `promisify`, `chunks(arr,n)`, `unique(arr)`.
- **Common interview question:** "Implement a debounce function and walk through the closure state."

### 8.5 Live-coding patterns: async retry / promise pool / sequential fetch
- **Java equivalent:** Analog of ExecutorService/CompletableFuture patterns you studied (`micro 1`).
- **Key difference:** `retry`, `promisePool(working consumers over array)`, sequential vs parallel await.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Implement `runWithConcurrency(tasks, 3)` that runs tasks with max 3 concurrent promises.
- **Common interview question:** "Run 10 async tasks but only 3 at a time — implement it."

### 8.6 Behavioral / system-adjacent JS talking points
- **Java equivalent:** Partial analog — your interview-simulation scripts taught talking structure (Servlet vs Spring style answers).
- **Key difference:** Be ready to compare "JS vs Java memory model / multithreading / packaging" like you did Java-vs-Spring.
- **Status:** `[x]` (same talking-structure skill)
- **Practice task:** Record yourself answering 5 questions aloud in 90 seconds each (like your Interview_Simulation drills).
- **Common interview question:** "Compare JavaScript's concurrency model with Java's — when would you pick one over the other?"

---

# LEVEL 9 — Pre-framework checklist (TypeScript intro, tooling, readiness for React/Node/Express)

### 9.1 TypeScript core: types, interfaces, unions, generics
- **Java equivalent:** Your generics (`Generics in java/Methods.md`, `92-`, `T2.md`) map almost 1:1 conceptually.
- **Key difference:** Structural typing (shape-based, not nominal like Java!); unions `A|B`, `type` aliases, interface = shape contract; no runtime cost (erases like Java type erasure); discriminated unions replace sealed+pattern-matching (matches your `96-Sealed.md` notes).
- **Status:** `[x]` (related generics/sealed concepts)
- **Practice task:** Type a UserRepo interface + discriminated-union `Result<T, E>` replacing exceptions.
- **Common interview question:** "What is structural typing and how does it differ from Java's nominal typing?"

### 9.2 TS configuration & integration (tsconfig, strict mode)
- **Java equivalent:** Partial analog — your Maven profile/config knowledge.
- **Key difference:** `tsconfig.json` (strict, target, module); TS is a *layer on JS* — always know what JS it compiles to.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Set up strict TS for your Level 6 FP pipeline; fix every strict error and note what each revealed.
- **Common interview question:** "Why would you enable strict mode and what extra guarantees does it give?"

### 9.3 Tooling: bundlers (Vite), formatters/linters (Prettier/ESLint)
- **Java equivalent:** Partial analog — Maven plugins/archetypes from the `MAVEN/` folder.
- **Key difference:** Ecosystem-first tooling; bundling solves browser module loading; you configure less, choose more (npm scripts + Vite ≈ `exec-maven-plugin`-flexible world).
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Scaffold a Vite Vanilla-TS project; add ESLint+Prettier; make `npm run lint` fail on a demo file then pass.
- **Common interview question:** "What problem does a bundler solve, and what does Vite add over raw ES modules?"

### 9.4 Testing: Jest / Vitest basics
- **Java equivalent:** Close analog — JUnit-style mental model (you know assertions, Arrange-Act-Assert).
- **Key difference:** `describe/it/expect`; async tests return/await promises; mocks/spies with `jest.fn()` (vs Mockito equivalence); snapshot testing optional; test runner is npm-run based.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Write 10 Jest tests for your Level 8 utility functions (debounce tests with fake timers).
- **Common interview question:** "How would you test an async function that calls an API?" (mock + fake timers).

### 9.5 Framework readiness: JavaScript in React/Node/Express ecosystem
- **Java equivalent:** Partial analog — decisions like Spring Boot vs plain Spring (your notes cover that decision logic).
- **Key difference:** Choose track: **Frontend** (React — needs Level 7 DOM/event model + immutability habits + closures for hooks) or **Backend** (Node/Express/NestJS — needs Level 5 async + 7.6/7.7 + middleware thinking, DI similar to Spring). Knowing `HABSA` levels 1–8 makes either track a framework-syntax course instead of a language course.
- **Status:** `[ ]` **(ADDED)**
- **Practice task:** Pick your track; write a 1-page note: "3 JS concepts I must over-master for my chosen framework."
- **Common interview question:** "Why do React hooks rely on closures, and why does stale-closure happen?"

---

## ✅ Checklist summary

| Level | Topics | Covered-in-Java-notes[x] |
|---|---|---:|
| 1 | 7 | 4 |
| 2 | 7 | 3 |
| 3 | 10 | 8 |
| 4 | 11 | 7 |
| 5 | 8 | 2 |
| 6 | 8 | 5 |
| 7 | 8 | 3 |
| 8 | 6 | 3 |
| 9 | 5 | 3 |
| **Total** | **70** | **~38** |

≈ 32 topics are genuinely **new** for you — mostly the JS-runtime specifics (hoisting/`this`/prototype/event loop/async/DOM/npm/tooling).

---

## 📚 Study order (recommended)

1. **Week 1 — Levels 1 + 2** (spawn your Java mental model into JS core: `this`/closures/prototypes dominate)
2. **Week 2 — Level 3 + start Level 4** (OOP quirks then data structures with stream-knowledge advantage)
3. **Week 3 — Level 5 (async)** ← spend extra time here; it's the #1 interview differentiator for backend engineers
4. **Week 4 — Level 6 + Level 4 leftovers**
5. **Week 5 — Level 7 (browser/Node)**,avor 30-min DOM/HTTP drills nightly
6. **Week 6 — Level 8** (interview drills + classic challenges)
7. **Week 7 — Level 9** (TypeScript + tooling + testing)
8. **Week 8 — Buffer + 2 mock interviews** (record yourself; redo lvl-8 tasks from memory)

**Pacing:** ~6–9 focused hours/week → ~8 weeks total. **Accelerated (3–4 h/day):** ~3.5–4 weeks.

### Estimated time per level
| Level | Focus | Est. time |
|---|---|---|
| 1 | Fundamentals | 10–12 h |
| 2 | Core JS (this/prototype/closures) | 14–18 h ⭐ hardest
| 3 | OOP in JS | 8–10 h |
| 4 | Data structures & built-ins | 10–14 h |
| 5 | Async JS | 16–20 h ⭐ most valuable
| 6 | Modern JS | 8–10 h |
| 7 | Browser & Node essentials | 12–16 h |
| 8 | Interview prep | 12–14 h |
| 9 | Pre-framework (TS/tooling) | 10–12 h |
| **Total** |  | **~100–126 h** |

> Rule of thumb: for every hour of reading, schedule 30 min of **Node REPL / browser console drills** — it's the fastest path from "Java-brain → JS-fluent".

*Generated 2026-10-30 from full `JAVA CORE☕` analysis.*
