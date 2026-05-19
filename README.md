<div align="center">

```
 ████████╗ ██████╗ ██╗  ██╗███████╗
    ██╔══╝██╔═══██╗██║ ██╔╝██╔════╝
    ██║   ██║   ██║█████╔╝ █████╗  
    ██║   ██║   ██║██╔═██╗ ██╔══╝  
    ██║   ╚██████╔╝██║  ██╗███████╗
    ╚═╝    ╚═════╝ ╚═╝  ╚═╝╚══════╝
```

**Token-oriented human-readable serialization**

[![Version](https://img.shields.io/badge/version-0.1.0-6C3EF4?style=for-the-badge&labelColor=0a0a0f)](https://github.com/NotKisoMomo/Toke/releases)
[![Luau](https://img.shields.io/badge/luau-typed-6C3EF4?style=for-the-badge&labelColor=0a0a0f)](https://luau-lang.org)
[![License](https://img.shields.io/badge/license-MIT-6C3EF4?style=for-the-badge&labelColor=0a0a0f)](LICENSE)
[![Built by Plinko Labs](https://img.shields.io/badge/built_by-Plinko_Labs-6C3EF4?style=for-the-badge&labelColor=0a0a0f)](https://github.com/NotKisoMomo)

</div>

---

Toke is a custom serialization format designed for **human readability and Roblox network transport**. Unlike binary-only formats that sacrifice debuggability, or plain JSON that sacrifices efficiency, Toke sits in between -- a notation you can read, write, and reason about, that still encodes repetitive data more compactly than raw strings.

The format is built around two ideas: a **global character table** built on first-encounter order, and a **token system** that registers whole words for reuse. Both can be referenced by index rather than repeated inline.

```
"Hello Hello World"  →  tf1eup1h1e2l1ote ws tp1e ws up1w1o1r2l1d
```

---

## Highlights

- Human-readable notation -- debuggable over the wire without tooling
- Two-level reference system -- individual chars and whole word tokens
- Run-length encoding on literals -- `3l` instead of `lll`
- Uppercase handling -- `up` for single char, `ou...e` for whole blocks
- Range references -- `rib1x3e` constructs new strings from indexed chars
- Token indexing -- `tp1i4e` extracts a single char from a registered token
- Fully typed Luau -- strict mode compatible
- Zero dependencies

---

## Files

| File | Role |
|---|---|
| `Toke.lua` | Public API -- `encode`, `decode`, `inspect`, `roundtrip` |
| `Lexer.lua` | Parses Toke notation into a typed token stream |
| `Util.lua` | Shared helpers -- char/token table management, number reading |
| `Types.lua` | Exported type definitions |

---

## Quick Start

```lua
local Toke = require(path.to.Toke)

local encoded = Toke.encode("Hello Hello World")
-- tf1eup1h1e2l1ote ws tp1e ws up1w1o1r2l1d

local decoded = Toke.decode(encoded)
-- "Hello Hello World"

local ok = Toke.roundtrip("Hello Hello World")
-- true
```

Toke is most effective on **repetitive string data** -- repeated words, shared substrings, and structured identifiers like Roblox instance or event names.

```lua
-- Roblox event names -- high repetition, good compression
local input = "PlayerAdded PlayerRemoving PlayerAdded CharacterAdded CharacterAdded"
local encoded = Toke.encode(input)

-- Words appearing 2+ times are auto-registered as tokens
-- Second+ occurrences emit tp<id>e instead of the full word
```

---

## Notation Reference

### Literals

| Notation | Meaning | Example |
|---|---|---|
| `<count><char>` | Repeat char N times | `3l` → `lll` |
| `ws` | Space character | `ws` → ` ` |
| `up<char>` | Single uppercase char | `upH` → `H` |
| `ou<content>e` | Uppercase entire block | `ouheoe` → `HEO` |

### Global Char Table

Characters are indexed in first-encounter order as the stream is decoded. Every new lowercase character seen gets the next available index.

| Notation | Meaning |
|---|---|
| `ri<index>e` | Emit char at index |
| `rib<a>x<b>e` | Emit new string from chars at indices a through b |

### Token System

Words can be registered as tokens and referenced later. Tokens appearing 2+ times in input are auto-registered by the encoder. Manual registration uses explicit ids.

| Notation | Meaning |
|---|---|
| `tf<id>e ... te` | Register token with explicit id |
| `tfe ... te` | Register token with auto-incremented id |
| `tp<id>e` | Emit full token |
| `tp<id>i<index>e` | Emit single char from token at position |

Token bodies can reference other tokens up to **2 levels deep**.

---

## Encoding Rules

The encoder runs a two-pass strategy:

1. **Frequency scan** -- count word occurrences across the full input
2. **Emit pass** -- words seen 2+ times are registered as tokens on first encounter, referenced via `tp<id>e` on all subsequent occurrences

Character indexing is automatic -- every new lowercase character encountered is appended to the global char table. The decoder rebuilds this table in lockstep as it reads the stream.

---

## When Toke Wins

Toke's compression ratio improves with **repetition**. The more a word or character recurs, the more it benefits from token and index references.

| Input type | Expected ratio |
|---|---|
| Repeated words (2+ times) | < 1.0 -- net gain |
| Mixed content, low repetition | ~1.0 -- neutral |
| Fully unique text | > 1.0 -- net loss |
| Roblox instance/event names | < 1.0 -- net gain |

For single short strings, raw transmission is always smaller. Toke pays off on **streams** of structured data sent repeatedly over a network session.

---

## API

### `Toke.encode(input: string): string`

Encodes a plain string into Toke notation. Automatically registers tokens for words appearing 2 or more times.

### `Toke.decode(encoded: string): string`

Decodes a Toke notation string back to plain text. Raises an error on malformed input.

### `Toke.inspect(encoded: string): { TokeToken }`

Returns the raw parsed token stream without decoding. Useful for debugging and building tooling on top of Toke.

### `Toke.roundtrip(input: string): boolean`

Encodes then decodes `input` and returns whether the result matches the original. Use in tests.

---

## Wiki

Full documentation lives in the [GitHub Wiki](https://github.com/NotKisoMomo/Toke/wiki).

| Page | Contents |
|---|---|
| [Home](https://github.com/NotKisoMomo/Toke/wiki) | Overview and links |
| [Getting Started](https://github.com/NotKisoMomo/Toke/wiki/Getting-Started) | Installation, first encode/decode |
| [Notation Guide](https://github.com/NotKisoMomo/Toke/wiki/Notation-Guide) | Full syntax reference with examples |
| [Encoder Internals](https://github.com/NotKisoMomo/Toke/wiki/Encoder-Internals) | Two-pass strategy, token threshold, char table |
| [Decoder Internals](https://github.com/NotKisoMomo/Toke/wiki/Decoder-Internals) | Stream walking, state reconstruction |
| [Token Depth](https://github.com/NotKisoMomo/Toke/wiki/Token-Depth) | Nested token references, depth limit |
| [Performance Guide](https://github.com/NotKisoMomo/Toke/wiki/Performance-Guide) | When to use Toke, benchmarks, tradeoffs |
| [API Reference](https://github.com/NotKisoMomo/Toke/wiki/API-Reference) | Full typed API docs |

---

<div align="center">

Built by [Plinko Labs](https://github.com/NotKisoMomo)

</div>
