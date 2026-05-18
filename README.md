# Toke Wiki

Toke is a token-oriented human-readable serialization format for Luau. This wiki covers everything from initial setup to internals.

---

## Pages

### Foundations
- [Getting Started](Getting-Started) -- installation and first use
- [Notation Guide](Notation-Guide) -- complete syntax reference

### Internals
- [Encoder Internals](Encoder-Internals) -- how encoding works
- [Decoder Internals](Decoder-Internals) -- how decoding works
- [Token Depth](Token-Depth) -- nested token references and the depth limit

### Reference
- [Performance Guide](Performance-Guide) -- when Toke wins, when it loses
- [API Reference](API-Reference) -- full typed API

---

## At a Glance

```
Input:    "PlayerAdded PlayerAdded PlayerRemoving PlayerAdded"
Encoded:  tf1e up1p1l1a1y1e1r up1a1d1d1e1d te ws tp1e ws ...
```

The encoder makes one pass to count word frequency, then a second pass to emit -- auto-registering any word that appears 2 or more times as a token. The decoder rebuilds the char and token tables in lockstep as it reads the stream, requiring no header or dictionary preamble.
