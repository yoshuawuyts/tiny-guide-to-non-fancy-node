# Tiny Guide to Non Fancy Node
A tiny guide to non fancy, high-value Node.js things. Because we're folks that
want to build fun things, rather than spending time fighting our tools.

## Introduction
Node.js is a truly neat piece of software. It has one of the largest
programming communities, and somehow both a very low barrier to entry and a
high skill cap. Picking Node as the hammer in your programming toolbox is not a
bad choice.

Node _can_ be a little overwhelming though. Because of the low barrier to entry
for both writing and publishing software, it can sometimes become hard to find
exactly what you need. That's why this tiny guide exists: a little set of
opinions to help guide you in the vast world of Node.

__Obviously other people will have different opinions, and that's cool. This is
what I've found to work for me, over the years. It's fine if we disagree.
But there might be a chance you'll find this works for you too.__

_Disclaimer: If you disagree enough to get angry at this guide, I urge you to
take a breath, step back and reconsider. Feel free to write your own guide;
hell - having more perspectives and resources is a great thing! Remember that
kindness matters_

## Programming style
[There's style, and there's
syntax](https://gist.github.com/dominictarr/2401787). Generally discussions
about punctuation and other linting things are boring, so use
[standard](https://github.com/feross/standard) and worry about solving actual
problems instead. `standard --fix` will even take care of formatting code for
you; it's been around for a while and generally gets the job done.

### language version
ES3 is generally all you need. Really? Yup, really. The Array methods in ES5
are nice, and template strings are useful - but that's it. `var` is performant,
and flexible. `callbacks` are the unifying interface for asynchronous
programming, and `for-loops` are about as fast as you'll get. Using a tiny
subset of the language makes your code stable, readable and predictable.
Exactly the way we like it.

### require
Don't bother with `import`, [it's
unstable](https://twitter.com/jasnell/status/819618164535214080). `require()`
has worked since the dawn of Node, and works reliably for any scenario you can
come up with. And no need to worry about "tree shaking" / "dead code
elimination" - unless you've got extremely complicated applications [it won't
matter](https://nolanlawson.com/2016/08/15/the-cost-of-small-modules/). At that
point you'll probably have much larger problems to worry about.

### functions
We don't need [13 different types of
functions](https://github.com/feross/standard/issues/414#issuecomment-182685548).
Using `function foo () {}` will cover all your bases. Yup, even arrow functions
aren't worth it - we aim for boring; and removing `arguments` and `this` from
arrow functions makes arrow functions weird and hard to debug.

## Async
Callbacks are your ticket. They're fast, easy to reason about and easy to
debug. If you ever find yourself struggling with them, [consider naming and
splitting them](http://callbackhell.com/). Other paradigms build on top of
callbacks and will probably make things more complicated. For example by
ignoring error handling in examples, swallowing errors all together or far
worse. You want errors to be predictable.

Speaking of errors: use `throw` to crash and burn your program (e.g. programmer
errors, for example a wrong type) and use callbacks for expected errors (e.g.
oops, wrong input - try again).

The [feross/run-*](https://github.com/feross/run-series) family of packages is
truly wonderful. They'll give you virtually all you need for working with
callbacks efficiently. There's also
[map-limit](https://github.com/hughsk/map-limit) which is my go-to hammer when
needing to run an array of things, in series, parallel or something in between.

If you ever need composable `async` in Node, consider using streams. The
`streams2` API isn't pretty, but it's worth learning. Streams can be passed
around synchronously and will send data from one place to another. They're
ideal for creating composable, reusable APIs. Check out
[mississippi](https://github.com/maxogden/mississippi) on GitHub to learn more
about good streams packages.

## Testing
The [Test Anything Protocol (TAP)](https://testanything.org/) has existed for years, and works in almost
every language. [tape](https://github.com/substack/tape) is the way to use it
in Node. Don't worry about fancy (Promise based) parallel test runners; if they
provide any measurable performance improvement then your code is probably too
complicated in the first place. Instead consider removing all IO from your
tests for example.

If you want pretty printing for `TAP`, use
[tap](https://github.com/tapjs/node-tap). If you want code coverage reporting,
use [nyc](https://github.com/istanbuljs/nyc).

Use [dependency-check](https://github.com/maxogden/dependency-check) to make
sure you didn't miss any deps. [greenkeeper](http://greenkeeper.io/) is neat if
you maintain applications.

## Functional utility toolbelts
Grab bag packages often lead to code creep; don't use 'em. If there's 60
functions exposed in any given package, chances are that given enough time
the code base will asymptotically start using a higher percentage of these
functions. And that's not good; there's a thing as being _too_ dependent on
external code.

## Databases
[level](https://github.com/Level/level) is the only database. Use
[membdb](https://github.com/juliangruber/memdb) when testing and developing.
Use [subleveldown](https://github.com/mafintosh/subleveldown) to split your
database. [Check out the wiki for more
modules](https://github.com/Level/levelup/wiki/Modules); there's one for
virtually anything you could need (replication, caching, LRU, etc.)

I'm not a fan of other databases as they're usually peer dependencies. If
you're building something, you probably want to optimize for iteration - not so
much other things. And level excels at that. Unless you're building a bank; I
wouldn't trust myself to build a boring money transaction system in Node. Take
advice from other people if you're planning to do stuff like that.

## Devops
Use [lil-pids](https://github.com/mafintosh/lil-pids) for process management
and [taco-nginx](https://github.com/mafintosh/taco-nginx) to expose processes
to the outside world. `scp(1)` or `npm` are generally enough to deploy your
stuff with. Use `npm install / npm start` for your apps to be predictable. You
probably don't need `Docker` - remember that Node is a cross platform virtual
machine already; if you get started Node is probably all you need. You can
always get fancy later.

## Distributed systems
You probably don't need high performance distributed systems stuff when writing
applications. Ask yourself: do you really need to run this on more than one
machine? This is some of the stuff we're working on during my day job though;
often it just turns a problem into a distributed problem.

If you do need to run something on multiple machines though (e.g. live video
streaming!) consider using [hypercore](https://github.com/mafintosh/hypercore)
(distributed streams) or [hyperdrive](https://github.com/mafintosh/hyperdrive)
(distributed filesystem). They're neat, secure modules that make writing
distributed system stuff relatively painless.

## APIs
I've recently been working on [merry](https://github.com/yoshuawuyts/merry/)
which uses streams, fast logging and has loads of conventions. It's all of my
opinions around API design turned to code.

## Static typing
It's generally a good idea to validate assumptions you make. This helps catch
mistakes early and makes fixing them easier later on. I like using Node's
built-in `assert()` package for this. Got an input type you want to validate?
Assert it. Expect certain config values to exist? Assert it. Because I use it
in all my packages, having an extra compile check for static types doesn't make
sense for me. When deploying production code
[unassertify](https://github.com/unassert-js/unassertify) removes the extra
checks.

When building APIs [json-schema](https://github.com/mafintosh/is-my-json-valid)
is a great way to validate assumptions made about incoming JSON. When building
custom wire protocols
[protocol-buffers](https://github.com/mafintosh/protocol-buffers) and similar
are great.

## Frontend
I've recently been working on [choo](https://choo.io)
which uses pure HTML, JS and has a clean architecture. It's all of my
opinions around frontend applications turned to code.

To serve frontend code I [use bankai](https://github.com/yoshuawuyts/bankai).
It's based on [browserify](https://github.com/substack/node-browserify) which
is a super stable tool to build frontend stuff. Boring and stable is the name
of the game. [budo](https://github.com/mattdesl/budo) is similar to `bankai`,
maybe you'll like it too.

For css there's [sheetify](https://github.com/stackcss/sheetify) and
[tachyons](http://tachyons.io/). `sheetify` allows you to require css from
`node_modules`. It's built right into `bankai` so it's easy to work with.
`tachyons` is a sane CSS system. It might feel awkward at first, but once you
get past the bump you'll be building maintainable websites faster than ever
before.

## Accounts, authentication and the like
We're currently working on easy to setup account stuff through
[township](https://github.com/township/township) but it's still a work in
progress. If there was anything that scratched our itches already we'd be using
it.

## Native development
[electron](https://github.com/electron/electron) is cool. I've never built a
mobile app, so I can't comment on that - probably building a website would be
where I'd start though.

## Wrapping up
Yup, and that's it. These are my boring tools for writing software. Obviously
there's much more tools to be discovered, but npm should be your friend. I hope
this little guide can at least help you out a little when getting started in
the wonderous world of Node.

_PS._ The fancy name for the tech described in this guide is [The Lebron
Stack](http://lebron.technology/). We don't kid. Happy coding!

## License
[MIT](https://tldrlegal.com/license/mit-license)
