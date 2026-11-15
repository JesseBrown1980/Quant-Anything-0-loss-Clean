# Quant Anything, 0 loss: the clean version

Programs, the master key, and a set of codexes that return enwik10. Everything here runs with Rust 1.81 or later and
nothing else: no Node, no Python, no outside crates in the codex program, integers only.

The full record (every run, the earlier programs, the kernel, the GGUF bodies) is in
[Quant-anything-0-loss](https://github.com/JesseBrown1980/Quant-anything-0-loss). This repository is the short path.

## What is here

| Path | What it is | Bytes |
|---|---|---:|
| [`rust/`](rust/) | The codex program, `asolaria_codex` (one file, [`rust/src/main.rs`](rust/src/main.rs)) | |
| [`rust-leaves/`](rust-leaves/) | The leaves program: a streaming wrapper around [`path2-two-shadow-recovery`](https://github.com/JesseBrown1980/path2-two-shadow-recovery), used unchanged | |
| [`master-key/masterkey.xz`](master-key/) | The master key, packed. Unpacked it is 33,554,432 bytes | 549,112 |
| [`master-key/stream.bin`](master-key/) | 4,194,304 bytes of enwik8, coded against the master key | 1,711,250 |
| [`example/enwik10-first-rooms/`](example/enwik10-first-rooms/) | The first 14,000 room codexes of enwik10 (3,200 bytes each) and their 8-byte stubs | 44,800,000 + 112,000 |
| [`kernel-and-key/`](kernel-and-key/) | The Asolaria kernel and the constellation key, 3,174 bytes each, with their hashes, the key's row file, the PLAYBOOK and the CODEBOOK | 3,174 + 3,174 |
| [`codexes/enwik10.qr`](codexes/) | The single 3,200-byte codex of the whole of enwik10 | 3,200 |
| [`codexes/wiki.qr`](codexes/) | The single 3,200-byte codex of enwik8 | 3,200 |
| [`SHA256SUMS`](SHA256SUMS) | SHA-256 of every file above | |

## Run the example: 95.8 MB of enwik10 from the key and the codexes

```sh
git clone https://github.com/JesseBrown1980/Quant-Anything-0-loss-Clean
cd Quant-Anything-0-loss-Clean
cargo build --release --manifest-path rust/Cargo.toml
xz -d -c master-key/masterkey.xz > key.raw
rust/target/release/asolaria_codex unpack key.raw 2 \
    example/enwik10-first-rooms/enwik10-first-14000.rooms enwik10-first.bin
```

The program prints this row:

```text
UNPACK|rooms=14000|body_bytes=95823079|body_sha256=65775c95ba220e9cc45bf6f2e80ce2ab738a78e1bb89ed9dbfa29f8a5980835b|inputs=key_and_rooms_only|json=0
```

`enwik10-first.bin` is the first 95,823,079 bytes of enwik10. If you have enwik10, check it:

```sh
head -c 95823079 enwik10 | sha256sum
# 65775c95ba220e9cc45bf6f2e80ce2ab738a78e1bb89ed9dbfa29f8a5980835b
```

The only inputs are `key.raw` and the room file. No copy of enwik10 is read.

## How it works

**The master key** is a table of 8,388,608 sixteen-bit numbers. For every pair of preceding bytes it gives the odds of
each bit of the next byte. It was built once, from the first 4,194,304 bytes of enwik8 and the Asolaria corpus. It has
never been shown enwik10.

**A room codex** is exactly 3,200 bytes: two bytes giving the length of its piece, the piece coded against the master
key, and zero padding. The body is divided as it streams in. A piece ends when its coded form has filled the room, so
the program finds the number of pieces by itself. Each room starts fresh, so any room opens by itself with the key.

**A stub** is the first 8 bytes of a room's SHA-256. The stub table has one stub per room, in order.

**To return the body**, the program opens each room in order with the key and writes the pieces one after another.

## The whole of enwik10

Measured on November 15, 2026, on one laptop. enwik10 is 10,000,000,000 bytes with SHA-256
`14a033c63a5d03eecdc205e8f7875ece7985a4d278877b0214980d5b3d675f52`.

| | Measured |
|---|---:|
| Room codexes | 1,460,938 |
| Bytes in all rooms | 4,675,001,600 |
| Bytes in the stub table | 11,687,504 |
| Shortest, mean and longest piece (bytes of enwik10 per room) | 2,719 / 6,844 / 26,483 |
| Bytes written back from the key and the rooms | 10,000,000,000 |
| SHA-256 of the body written back | `14a033c6…5f52`, equal to enwik10's |
| Seconds to divide enwik10 into rooms | 520 |
| Seconds to write enwik10 back | 542 |

The rooms hold enwik10 in 46.75% of its size: 3.74 bits per character. The master key adds 549,112 bytes once, for
any number of bodies.

To make the rooms from your own copy of enwik10, or from any file:

```sh
rust/target/release/asolaria_codex pack   key.raw 2 enwik10 enwik10.rooms enwik10.stubs
rust/target/release/asolaria_codex unpack key.raw 2 enwik10.rooms enwik10.back
sha256sum enwik10.back
```

## The single 3,200-byte codex

`drill` reads a whole body once and writes one 3,200-byte codex: 800 lanes of four bytes. The same body always gives
the same codex.

```sh
rust/target/release/asolaria_codex drill enwik10 my.qr
cmp my.qr codexes/enwik10.qr
```

For enwik10 that is 10,000,000,000 bytes to 3,200 bytes (3,125,000 to 1), measured in 178 seconds, and `my.qr` is
identical to [`codexes/enwik10.qr`](codexes/enwik10.qr). The codex names its body: `resolve` takes a codex and an
index of files, finds the file whose codex is the same, and returns it with its hash checked.

## The master key round trip

```sh
rust/target/release/asolaria_codex dec key.raw 2 master-key/stream.bin 4194304 decoded.bin
sha256sum decoded.bin
# 5ba647fec2ba51b0383cbe7147e15a478d85613245b7131c41764dbdd5062f77
rust/target/release/asolaria_codex enc key.raw 2 decoded.bin 4194304 again.bin
cmp again.bin master-key/stream.bin
```

1,711,250 bytes and the key give the first 4,194,304 bytes of enwik8, and coding them again gives the same stream.

## The leaves: any three rebuild the body

`rust-leaves` projects a body onto seven coprime cylinders, one leaf per cylinder. Each leaf is half the size of the
body. Any three leaves rebuild the body exactly. Two leaves hold: the program emits nothing.

```sh
cargo build --release --manifest-path rust-leaves/Cargo.toml
rust-leaves/target/release/path2_driver black BODY leaves/
rust-leaves/target/release/path2_driver white leaves/ 2 4 6 BODY_BYTES body.back
rust-leaves/target/release/path2_driver hold  leaves/ 0 1 BODY_BYTES
```

Measured on enwik8 (100,000,000 bytes):

| | Measured |
|---|---|
| Seven leaves | 50,000,000 bytes each, written in 3 seconds |
| Rebuilt from leaves 2, 4, 6 alone | 100,000,000 bytes, identical to enwik8, 11 seconds |
| Rebuilt from leaves 0, 1, 2 alone | identical to enwik8 |
| Leaves 0, 1 alone | held, 0 bytes emitted |
| One bit changed in one leaf, then rebuilt | SHA-256 `3834a93b…` against enwik8's `2b49720e…`; the rebuild is rejected |

## The kernel and the key

[`kernel-and-key/`](kernel-and-key/) holds the Asolaria kernel and the constellation key as they are on the operator's
machine, byte for byte.

| File | Bytes | SHA-256 |
|---|---:|---|
| `ASOLARIA-KERNEL-3174.bin` | 3,174 | `202ff2f06af54788958a03f60245593f89a88b0a3a4fc76435ad8468f151dc98` |
| `ASOLARIA-KERNEL-3174.kernel-pump-copy.bin` | 3,174 | `f075f5c64d656ae3afa91c793ab2c74b5425e41de141d6b7dc5cf4605663b0ca` |
| `ASOLARIA-CONSTELLATION-KEY-3174.bin` | 3,174 | `cce4f1e8ae407f108b5c122e764f0b4c3882ce059cf900a0cc1b2a0e7588c1ea` |

`KEY-3174.hbp` is the key's own row file. `PLAYBOOK.md` and `CODEBOOK.md` are the July instructions for the kernel
pump, and `_spheres_desc.txt` describes the spheres. The two smaller `.bin` files are the bodies the pump programs
were run on. The pump programs themselves are in the full repository under
[`programs/kernel-pump`](https://github.com/JesseBrown1980/Quant-anything-0-loss/tree/main/programs/kernel-pump).

## Program reference

```text
asolaria_codex drill   INPUT OUT.qr                         one 3,200-byte codex of a whole body
asolaria_codex train   OUT.raw ORDER MAXBYTES FILE...       build a key from bodies
asolaria_codex enc     KEY.raw ORDER INPUT N OUT.bin        code the first N bytes against a key
asolaria_codex dec     KEY.raw ORDER STREAM.bin N OUT       the N bytes again
asolaria_codex pack    KEY.raw ORDER INPUT ROOMS STUBS [MAX_ROOMS]   divide a body into room codexes
asolaria_codex unpack  KEY.raw ORDER ROOMS OUT              the body, from the key and the rooms
asolaria_codex resolve CODEX STORE-INDEX.hbp                the codex names its body; the store returns it
```

`ORDER` is `2` for the published master key. Every program prints one row of `NAME|key=value|…|json=0`.

Both programs pass `cargo clippy --release -- -D warnings` on Rust 1.81. The workflow in
[`.github/workflows/check.yml`](.github/workflows/check.yml) builds them on a clean machine and runs the example and
the round trip above on every push.
