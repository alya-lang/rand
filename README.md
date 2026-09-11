# rand

[![CI](https://github.com/alya-lang/rand/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/rand/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/rand?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Frand%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Frand%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

Modern, fast pseudo-random number generator (PRNG), statistical distributions, UUID v4/v7, ULID, and sampling toolkit for Alya.

---

## 🌟 Features

- ⚡ **High Performance Engines**: Built-in global PRNG (>330M ops/s) alongside dedicated PRNG engines: **SplitMix64**, **Xorshift64**, **PCG32**, and **LCG**.
- 🎲 **Statistical Distributions**: Uniform ranges (integers and floats), Normal/Gaussian (Box-Muller transform), Exponential, Bernoulli, and Binomial trials.
- 🎯 **Sampling & Shuffling**: Non-destructive `sample` (without replacement), `choices` (with replacement), `choice`, in-place `shuffle`, `shuffled` copies, and cumulative roulette `weighted_choice`.
- 🆔 **Modern Unique Identifiers**: RFC 4122 **UUID v4**, RFC 9562 timestamp-ordered **UUID v7**, Crockford Base32 **ULID**, and URL-safe **NanoID**.
- 🔤 **Random Strings**: Customizable alphanumeric, numeric PIN/OTP, hex tokens, and custom alphabet strings.
- 🔄 **100% Backward Compatible**: Drop-in aliases for all legacy `std/rand` functions (`rand_int`, `rand_float`, `rand_choice`, `rand_shuffle`, `rand_new`, etc.).

---

## 📁 Project Architecture

```
rand/
├── alya.toml                  # Package manifest
├── src/
│   ├── lib.alya               # Public API facade & std/rand aliases
│   ├── types.alya             # Rng struct and algorithm constants
│   └── core/
│       ├── algorithms.alya    # PRNG engines: SplitMix64, Xorshift64, PCG32, LCG
│       ├── distributions.alya # Normal, Exponential, Bernoulli, Binomial, Uniform
│       ├── sampling.alya      # Choice, sample, choices, shuffle, weighted_choice
│       ├── strings.alya       # Alphanumeric, digits, hex, ascii generators
│       └── identifiers.alya   # UUID v4, UUID v7, ULID, NanoID
├── examples/
│   └── demo.alya              # Comprehensive usage showcase
├── tests/
│   └── test_basic.alya        # Automated test suite (39 tests)
└── benches/
    └── bench_basic.alya       # Micro-benchmarks
```

---

## 📦 Installation

Add `rand` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
rand = { git = "https://github.com/alya-lang/rand", branch = "main" }
```

Or install it directly using the Alya package CLI:

```bash
alyac add rand --git https://github.com/alya-lang/rand --branch main
alyac install
```

---

## 🚀 Quick Start

```alya
import "rand"

function main()
    # 1. Basic Generation
    let n = rand::int(1, 100)          # Integer in [1, 100]
    let f = rand::float_range(0.0, 1.0) # Float in [0.0, 1.0)
    let ok = rand::chance(75)          # 75% probability (returns 1 or 0)

    # 2. Collections & Sampling
    let items = ["apple", "banana", "cherry", "date"]
    let pick = rand::choice(items)
    let subset = rand::sample(items, 2) # Without replacement

    # 3. Unique Identifiers
    let id_v4 = rand::uuid_v4()         # RFC 4122 UUID v4
    let id_v7 = rand::uuid_v7()         # RFC 9562 UUID v7 (time-sortable)
    let id_ulid = rand::ulid()          # 26-char Crockford Base32 ULID
    let token = rand::nanoid(21)        # 21-char NanoID

    # 4. Deterministic RNG with Seed
    let rng = rand::new(42, rand::ALG_SPLITMIX64)
    let val = rand::rng_int(rng, 1, 10)
end

main()
```

---

## 📖 API Reference

### Global Convenience Facade

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `next()` | - | `int` | Raw pseudo-random non-negative 31-bit integer. |
| `int(min, max)` | `min: int, max: int` | `int` | Random integer in inclusive range `[min, max]`. |
| `rand_float()` | - | `float` | Random float in half-open range `[0.0, 1.0)`. |
| `float_range(min, max)` | `min: float, max: float` | `float` | Random float in half-open range `[min, max)`. |
| `bool()` | - | `int` | Returns `1` or `0` with 50% probability. |
| `chance(pct)` | `pct: int` | `int` | Returns `1` with `pct`% probability (0..100). |
| `die(sides = 6)` | `sides: int` | `int` | Simulates rolling a die with `sides` faces `[1, sides]`. |
| `bernoulli(p)` | `p: float` | `int` | Bernoulli trial with success probability `p` (`0.0 <= p <= 1.0`). |
| `binomial(n, p)` | `n: int, p: float` | `int` | Binomial experiment with `n` trials and probability `p`. |
| `normal(mean, stddev)` | `mean: float, stddev: float` | `float` | Normally distributed float via Box-Muller transform. |
| `exponential(rate)` | `rate: float` | `float` | Exponentially distributed float with given rate parameter. |

### Sampling & Collections

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `choice(arr)` | `arr: array` | `any` | Returns a single randomly selected element from `arr`. |
| `choices(arr, count)` | `arr: array, count: int` | `array` | Returns `count` elements sampled with replacement. |
| `sample(arr, count)` | `arr: array, count: int` | `array` | Returns `count` distinct elements sampled without replacement. |
| `shuffle(arr)` | `arr: array` | `array` | In-place Fisher-Yates shuffle of `arr`. |
| `shuffled(arr)` | `arr: array` | `array` | Returns a new shuffled copy of `arr`. |
| `weighted_choice(items, weights)` | `items: array, weights: array` | `any` | Selects an item based on relative integer weights using cumulative roulette. |

### Strings & Identifiers

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `string(len, charset)` | `len: int, charset: string` | `string` | Generates a string of length `len` from custom alphabet. |
| `alphanumeric(len = 16)` | `len: int` | `string` | Generates alphanumeric string `[a-zA-Z0-9]`. |
| `digits(len = 6)` | `len: int` | `string` | Generates numeric string (ideal for OTP / PIN codes). |
| `hex(len = 16)` | `len: int` | `string` | Generates lowercase hexadecimal string. |
| `hex_upper(len = 16)` | `len: int` | `string` | Generates uppercase hexadecimal string. |
| `uuid_v4()` | - | `string` | RFC 4122 compliant UUID v4 (`xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`). |
| `uuid_v4_simple()` | - | `string` | 32-character hexadecimal UUID v4 without hyphens. |
| `uuid_v7()` | - | `string` | RFC 9562 compliant timestamp-ordered UUID v7. |
| `uuid_v7_simple()` | - | `string` | 32-character hexadecimal UUID v7 without hyphens. |
| `ulid()` | - | `string` | 26-character Crockford Base32 Universally Unique Lexicographically Sortable Identifier. |
| `nanoid(size = 21)` | `size: int` | `string` | Compact, URL-friendly unique identifier. |

### Deterministic PRNG Engines

| Algorithm Constant | Value | Description |
|---|---|---|
| `ALG_XORSHIFT64` | `1` | Ultra-fast 64-bit shift-register generator (Marsaglia). |
| `ALG_SPLITMIX64` | `2` | High-quality 64-bit generator with excellent state avalanche (Default). |
| `ALG_PCG32` | `3` | Permuted Congruential Generator (O'Neill). |
| `ALG_LCG` | `4` | Linear Congruential Generator. |

Use `new(seed, algorithm)` to instantiate an independent RNG:

```alya
let rng = rand::new(12345, rand::ALG_XORSHIFT64)
let r_int = rand::rng_int(rng, 1, 100)
let r_flt = rand::rng_float(rng)
```

---

## 🧪 Running Tests & Benchmarks

Run the test suite using `alyac`:

```bash
alyac run tests/test_basic.alya
```

Run the benchmark suite:

```bash
alyac run benches/bench_basic.alya
```

Run the example demo:

```bash
alyac run examples/demo.alya
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository and clone it locally
2. Install dependencies:
   ```bash
   alyac install
   ```
3. Create your feature branch (`git checkout -b feature/my-feature`)
4. Verify tests and formatting before opening a PR:
   ```bash
   alyac test
   alyac fmt . --check
   ```
5. Commit your changes (`git commit -m "feat: add feature"`) and open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.