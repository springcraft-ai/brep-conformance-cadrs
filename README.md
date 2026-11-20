# brep-conformance-cadrs

The [brep-conformance](https://github.com/springcraft-ai/brep-conformance) adapter for
[cadrs](https://github.com/rvdende/cadrs): it puts cadrs's B-rep kernel, `cadrs_kernel`, behind the
suite's protocol, so that the suite can score it.

## Who maintains this

brep-conformance scores a B-rep kernel through an adapter: a small program that answers the suite's
requests with the kernel's own calls. Springcraft maintains the suite and this adapter. cadrs has
its own maintainers, at [rvdende/cadrs](https://github.com/rvdende/cadrs).

- **It adds nothing to the kernel.** The adapter is pass-through, as the suite's
  [contract](https://github.com/springcraft-ai/brep-conformance/blob/main/doc/contract.md#what-an-adapter-may-do)
  asks: one call of the kernel for each function, and anything the kernel has no one call for is
  listed as unsupported. What is scored is cadrs, not this code.
- **The suite decides what is scored.** An official run uses only a commit of this adapter that the
  suite's maintainers have reviewed as pass-through.

## Overview

The adapter answers the requests of the suite's
[api.md](https://github.com/springcraft-ai/brep-conformance/blob/main/doc/api.md#calling-a-kernel)
on stdin and stdout, through `serve` in the `brep-protocol` crate. [`src/cadrs.rs`](src/cadrs.rs)
answers each function with one call of a `cadrs_kernel` session (`DefaultKernel`), as
[the contract](https://github.com/springcraft-ai/brep-conformance/blob/main/doc/contract.md#what-an-adapter-may-do)
asks, and `supports` lists the functions it answers: since cadrs 0.2.9, all of them.

What the adapter adds is translation:

- The suite's profiles, frames and paths become cadrs's sketches, planes and curves.
- A point becomes the edge or face it lies on through the kernel's distance query, within `10·ε`.
- cadrs's `EmptyResult` from a boolean is the empty solid.
- A STEP file with several solids is gathered into one body by `compound`.
- A split's pieces are sorted by side.
- Where the protocol's convention is one of several cadrs offers, the adapter names it: a torus
  whose tube crosses its axis is refused (`Spindle::Refuse`), and distances are to solids, insides
  included (`DistanceTarget::Solid`).

cadrs_kernel is not on crates.io. [`Cargo.toml`](Cargo.toml) takes it from git at a fixed `rev`.

## Setup

The Rust toolchain [`rust-toolchain.toml`](rust-toolchain.toml) pins.

## Usage

The suite runs the adapter as a separate process. Build the adapter here, then run the suite from a
checkout of [brep-conformance](https://github.com/springcraft-ai/brep-conformance) beside this one:

```bash
cargo build --release
cd ../brep-conformance
cargo build --release -p brep-conformance
target/release/brep-conformance --adapter ../brep-conformance-cadrs/target/release/cadrs-adapter \
    --out cadrs.json
```

## Licences

The adapter is under the MIT licence: see [LICENSE](LICENSE). cadrs is under the MIT or the
Apache-2.0 licence: see cadrs's own repository.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
