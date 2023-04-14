# Hello Substrate

`pallets/hello-substrate`
<a target="_blank" href="https://playground.substrate.dev/?deploy=recipes&files=%2Fhome%2Fsubstrate%2Fworkspace%2Fpallets%2Fhello-substrate%2Fsrc%2Flib.rs">
<img src="https://img.shields.io/badge/Playground-Try%20it!-brightgreen?logo=Parity%20Substrate" alt ="Try on playground"/>
</a>
<a target="_blank" href="https://github.com/substrate-developer-hub/recipes/tree/master/pallets/hello-substrate/src/lib.rs">
<img src="https://img.shields.io/badge/Github-View%20Code-brightgreen?logo=github" alt ="View on GitHub"/>
</a>

This pallet has one dispatchable call that prints a message to the node's output. Printing to the
node log is not common for runtimes, but can be quite useful when debugging and as a "hello world"
example. Because this is the first pallet in the recipes, we'll also take a look at the general
structure of a pallet.

## No Std

The very first line of code tells the rust compiler that this crate should not use rust's standard
library except when explicitly told to. This is useful because Substrate runtimes compile to Web
Assembly where the standard library is not available.

```rust, ignore
#![cfg_attr(not(feature = "std"), no_std)]
```

Next usage is to ensure `unused_unit` warnings are not emitted. This is a common warning that can be
removed by adding the following line.

```rust, ignore
#[allow(clippy::unused_unit)]
```

## Imports

Just one `import` which includes everything:

```rust, ignore
pub use pallet::*;
```

## Tests

Next we see a reference to the tests module. This pallet, as with most recipes pallets, has tests
written in a separate file called `tests.rs`.

## Module

Here the pallet is enabled as dev mode by annotating the module with
[`#[frame_support::pallet]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#dev-mode-palletdev_mode)
attribute with `dev_mode` like this:

```rust, ignore
#[frame_support::pallet(dev_mode)]
pub mod pallet {
	// --snip--
}
```

> `#[frame_support::pallet(dev_mode)]` attribute shouldn't be used on a pallet to be deployed on
> production. Remove this before deployment.

Inside this module, we define the pallet's traits, dispatchable calls, and storage items.

---

Next, you'll find imports inside that come from various parts of the Substrate framework. All
pallets will import from a few common crates including
[`frame-support`](https://paritytech.github.io/substrate/master/frame_support/index.html), and
[`frame-system`](https://paritytech.github.io/substrate/master/frame_system/index.html). Complex
pallets will have many imports. The `hello-substrate` pallet uses these imports.

```rust, ignore
use frame_support::{dispatch::DispatchResultWithPostInfo, pallet_prelude::*};
use frame_system::pallet_prelude::*;
use sp_runtime::print;
```

where,

- [`frame_support::dispatch::DispatchResultWithPostInfo`](https://paritytech.github.io/substrate/master/frame_support/dispatch/type.DispatchResultWithPostInfo.html)
  is a type alias for `Result<(), DispatchError>` with an additional `PostDispatchInfo` field. This
  is the return type of a dispatchable call.
- [`frame_support::pallet_prelude::*`](https://paritytech.github.io/substrate/master/frame_support/pallet_prelude/index.html)
  is a set of commonly used types and traits that are useful when writing pallets.
- `sp_runtime::print` is for accessing the `print` function that prints to the node log.

### Configuration Trait

Next, each pallet has a configuration trait which is called
[`Config`](https://substrate.dev/rustdocs/v3.0.0/frame_system/pallet/trait.Config.html). The
configuration trait can be used to access features from other pallets, or
[constants](./constants.md) that affect the pallet's behavior. This pallet is simple enough that our
configuration trait can remain empty, although it must still exist.

```rust, ignore
pub trait Config: frame_system::Config {}
```

### Pallet Struct

Need to have a placeholder
[`#[pallet::pallet]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#pallet-struct-placeholder-palletpallet-mandatory)
to specify the pallet information.
[`[pallet::generate_store($vis trait Store)]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#palletgenerate_storevis-trait-store)
is used to generate a `Store` trait associating all storage types. Hence, altogether the `Pallet`
struct is annotated like this:

```rust, ignore
#[pallet::pallet]
#[pallet::generate_store(pub(super) trait Store)]
pub struct Pallet<T>(PhantomData<T>);
```

> Here, `PhantomData` doesn't do anything. It's just a way to communicate to the compiler that you
> are using data in a certain way and it's up to you to express that information accurately. Read
> [more](https://doc.rust-lang.org/nomicon/phantom-data.html).

### Pallet Hooks

[`#[pallet::hooks]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#hooks-pallethooks-optional)
is a optional attribute that can be used to define pallet hooks. Hooks are functions that are called
at specific points in the block execution process. Hooks are used to execute code that depends on
the state of the block. For example, a hook can be used to execute code that depends on the block
number, or the block weight. Hooks are defined in the `Hooks` associated type of the pallet's
configuration trait. The `hello-substrate` pallet does not use any hooks, so we can leave the
`Hooks` type empty.

```rust, ignore
#[pallet::hooks]
impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {}
```

### Dispatchable Calls

A Dispatchable call is a function that a blockchain user can call as part of an Extrinsic.
"Extrinsic" is Substrate jargon meaning a call from outside of the chain. Most of the time they are
transactions, and for now it is fine to think of them as transactions. Dispatchable calls are
defined inside as method. The `hello-substrate` pallet has a single dispatchable call called
`say_hello`.

As you can see, the `say_hello` function is decorated with the `#[pallet::call]`
attribute. #[pallet::call] is a macro that can be used to define a pallet's dispatchable calls. The
macro takes care of the boilerplate code that is required to make a dispatchable call work. The
macro also takes care of the encoding and decoding of the call arguments.

Also

```rust, ignore
#[pallet::call]
impl<T: Config> Pallet<T> {
	/// Increase the value associated with a particular key
	#[pallet::weight(10_000)]
	pub fn say_hello(origin: OriginFor<T>) -> DispatchResultWithPostInfo {
		// --snip--
	}

	// More dispatchable calls could go here
}
```

As you can see, our `hello-substrate` pallet has a dispatchable call that takes a single argument,
called `origin`. The `origin` argument is a
[`OriginFor<T>`](https://paritytech.github.io/substrate/master/frame_system/enum.Origin.html)
[`DispatchResultWithPostInfo`](https://paritytech.github.io/substrate/master/frame_support/dispatch/index.html#types).
The call returns a which can be either `Ok(().into())` indicating that the call succeeded, or an
[`DispatchErrorWithPostInfo`](https://paritytech.github.io/substrate/master/frame_support/dispatch/type.DispatchErrorWithPostInfo.html)
which is demonstrated in most other recipes pallets.

> **Note:** The `DispatchResultWithPostInfo` type is a subset of `DispatchResult`. It is used to
> override the default return type `PostDispatchInfo` for dispatchable calls. The `PostDispatchInfo`
> is a struct that contains information about the execution of a dispatchable call. It is used to
> calculate the transaction fee. The `PostDispatchInfo` is returned by the `#[pallet::weight]`
> macro. For more information, see the [Weights](./weights.md) section.

```rust, ignore
struct PostDispatchInfo {
	/// Actual weight consumed by the call.
	pub actual_weight: Option<Weight>,
	/// The fee paid for the call.
	pub pays_fee: Pays,
}
```

So, the output is more about the weight of the call and the fee paid for the call. And hence, the
dispatchable call returns a `DispatchResultWithPostInfo` type. This says to return the function
result along with transaction details.

### Weight Annotations

Right before the `hello-substrate` function, we see the line `#[weight = 10_000]`. This line
attaches a default weight to the call. Ultimately weights affect the fees a user will have to pay to
call the function. Weights are a very interesting aspect of developing with Substrate, but they too
shall be covered later in the section on [Weights](./weights.md). For now, and for many of the
recipes pallets, we will simply use the default weight as we have done here.

## Inside a Dispatchable Call

Let's take a closer look at our dispatchable call.

```rust, ignore
pub fn say_hello(origin: OriginFor<T>) -> DispatchResultWithPostInfo {
	// Ensure that the caller is a regular keypair account
	let caller = ensure_signed(origin)?;

	// Print a message
	print("Hello World");
	// Inspecting a variable as well
	debug::info!("Request sent by: {:?}", caller);

	// Indicate that this call succeeded
	Ok(().into())
}
```

This function essentially does three things. First, it uses the
[`ensure_signed` function](https://substrate.dev/rustdocs/v3.0.0/frame_system/fn.ensure_signed.html)
to ensure that the caller of the function was a regular user who owns a private key. This function
also returns who that caller was. We store the caller's identity in the `caller` variable.

Second, it prints a message and logs the caller. Notice that we aren't using Rust's normal
`println!` macro, but rather a special
[`print` function](https://substrate.dev/rustdocs/v3.0.0/sp_runtime/fn.print.html) and
[`debug::info!` macro](https://substrate.dev/rustdocs/v3.0.0/frame_support/debug/macro.info.html).
The reason for this is explained in the next section.

Finally, the call returns `Ok(())` to indicate that the call has succeeded. At a glance it seems
that there is no way for this call to fail, but this is not quite true. The `ensure_signed`
function, used at the beginning, can return an error if the call was not from a signed origin. This
is the first time we're seeing the important paradigm "**Verify first, write last**". In Substrate
development, it is important that you always ensure preconditions are met and return errors at the
beginning. After these checks have completed, then you may begin the function's computation.

## Printing from the Runtime

Printing to the terminal from a Rust program is typically very simple using the `println!` macro.
However, Substrate runtimes are compiled to both Web Assembly and a regular native binary, and do
not have access to rust's standard library. That means we cannot use the regular `println!`. I
encourage you to modify the code to try using `println!` and confirm that it will not compile.
Nonetheless, printing a message from the runtime is useful both for logging information, and also
for debugging.

![Substrate Architecture Diagram](./img/substrate-architecture.png)

At the top of our pallet, we imported `sp_runtime`'s
[`print` function](https://substrate.dev/rustdocs/v3.0.0/sp_runtime/fn.print.html). This special
function allows the runtime to pass a message for printing to the outer part of the node which is
not compiled to Wasm and does have access to the standard library and can perform regular IO. This
function is only able to print items that implement the
[`Printable` trait](https://substrate.dev/rustdocs/v3.0.0/sp_runtime/traits/trait.Printable.html).
Luckily all the primitive types already implement this trait, and you can implement the trait for
your own datatypes too.

**Print function note:** To actually see the printed messages, we need to use the flag
`-lruntime=debug` when running the kitchen node. So, for the kitchen node, the command would become
`./target/release/kitchen-node --dev -lruntime=debug`.

The next line demonstrates using `debug::info!` macro to log to the screen and also inspecting the
variable's content. The syntax inside the macro is very similar to what regular rust macro
`println!` takes.

You can specify the logger target with

```rust, ignore
debug::debug!(target: "mytarget", "called by {:?}", sender);
```

Now you can filter logs with

```
kitchen-node --dev -lmytarget=debug
```

If you do not specify the logger target, it will be set to the crate's name (not to `runtime`!).

**Runtime logger note:** When we execute the runtime in native, `debug::info!` messages are printed.
However, if we execute the runtime in Wasm, then an additional step to initialise
[RuntimeLogger](https://substrate.dev/rustdocs/v3.0.0/frame_support/debug/struct.RuntimeLogger.html)
is required:

```
debug::RuntimeLogger::init();
```

You'll need to call this inside every pallet dispatchable call before logging.
