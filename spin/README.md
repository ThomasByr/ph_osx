# spin-rs

Spin-based synchronization primitives.

This crate provides [spin-based](https://en.wikipedia.org/wiki/Spinlock)
versions of the primitives in `std::sync`. Because synchronization is done
through spinning, the primitives are suitable for use in `no_std` environments.

## Features

- `Mutex`, `RwLock`, `Once`, `Lazy` and `Barrier` equivalents
- Support for `no_std` environments
- [`lock_api`](https://crates.io/crates/lock_api) compatibility
- Upgradeable `RwLock` guards
- Guards can be sent and shared between threads
- Guard leaking
- Ticket locks
- Different strategies for dealing with contention

## Usage

Include the following under the `[dependencies]` section in your `Cargo.toml` file.

```toml
spin = "x.y"
```

## Example

When calling `lock` on a `Mutex` you will get a guard value that provides access
to the data. When this guard is dropped, the mutex will become available again.

```rust
extern crate spin;
use std::{sync::Arc, thread};

fn main() {
    let counter = Arc::new(spin::Mutex::new(0));

    let thread = thread::spawn({
        let counter = counter.clone();
        move || {
            for _ in 0..100 {
                *counter.lock() += 1;
            }
        }
    });

    for _ in 0..100 {
        *counter.lock() += 1;
    }

    thread.join().unwrap();

    assert_eq!(*counter.lock(), 200);
}
```

## Feature flags

The crate comes with a few feature flags that you may wish to use.

- `mutex` enables the `Mutex` type.
- `spin_mutex` enables the `SpinMutex` type.
- `ticket_mutex` enables the `TicketMutex` type.
- `use_ticket_mutex` switches to a ticket lock for the implementation of `Mutex`. This
  is recommended only on targets for which ordinary spinning locks perform very badly
  because it will change the implementation used by other crates that depend on `spin`.
- `rwlock` enables the `RwLock` type.
- `once` enables the `Once` type.
- `lazy` enables the `Lazy` type.
- `barrier` enables the `Barrier` type.
- `lock_api` enables support for [`lock_api`](https://crates.io/crates/lock_api)
- `std` enables support for thread yielding instead of spinning.
- `portable_atomic` enables usage of the `portable-atomic` crate
  to support platforms without native atomic operations (Cortex-M0, etc.).
  The `portable_atomic_unsafe_assume_single_core` cfg flag
  must also be set by the final binary crate.
  This can be done by adapting the following snippet to the `.cargo/config` file:

  ```toml
  [target.<target>]
  rustflags = [ "--cfg", "portable_atomic_unsafe_assume_single_core" ]
  ```

  Note that this cfg is unsafe by nature, and enabling it for multicore systems is unsound.

## Remarks

It is often desirable to have a lock shared between threads. Wrapping the lock in an
`std::sync::Arc` is route through which this might be achieved.

Locks provide zero-overhead access to their data when accessed through a mutable
reference by using their `get_mut` methods.

The behaviour of these lock is similar to their namesakes in `std::sync`. they
differ on the following:

- Locks will not be poisoned in case of failure.
- Threads will not yield to the OS scheduler when encounter a lock that cannot be
  accessed. Instead, they will 'spin' in a busy loop until the lock becomes available.

Many of the feature flags listed above are enabled by default. If you're writing a
library, we recommend disabling those that you don't use to avoid increasing compilation
time for your crate's users. You can do this like so:

```toml
[dependencies]
spin = { version = "x.y", default-features = false, features = [...] }
```