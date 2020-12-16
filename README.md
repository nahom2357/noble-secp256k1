# noble-secp256k1 ![Node CI](https://github.com/paulmillr/noble-secp256k1/workflows/Node%20CI/badge.svg) [![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat-square)](https://github.com/prettier/prettier)

[Fastest](#speed) JS implementattion of [secp256k1](https://www.secg.org/sec2-v2.pdf),
an elliptic curve that could be used for asymmetric encryption,
ECDH key agreement protocol and deterministic ECDSA signature scheme from RFC6979. Supports Schnorr signatures.

Algorithmically resistant to timing attacks. Tested against thousands of vectors from tiny-secp256k1.

Check out a blog post about this library: [Learning fast elliptic-curve cryptography in JS](https://paulmillr.com/posts/noble-secp256k1-fast-ecc/).

### This library belongs to *noble* crypto

> **noble-crypto** — high-security, easily auditable set of contained cryptographic libraries and tools.

- No dependencies, one small file
- Easily auditable TypeScript/JS code
- Uses es2020 bigint. Supported in Chrome, Firefox, Safari, node 10+
- All releases are signed and trusted
- Check out all libraries:
  [secp256k1](https://github.com/paulmillr/noble-secp256k1),
  [ed25519](https://github.com/paulmillr/noble-ed25519),
  [bls12-381](https://github.com/paulmillr/noble-bls12-381),
  [ripemd160](https://github.com/paulmillr/noble-ripemd160)

## Usage

> npm install noble-secp256k1

```js
import * as secp from "noble-secp256k1";

(async () => {
  // You can also pass Uint8Array and BigInt.
  const privateKey = "6b911fd37cdf5c81d4c0adb1ab7fa822ed253ab0ad9aa18d77257c88b29b718e";
  const messageHash = "9c1185a5c5e9fc54612808977ee8f548b2258d31";
  const publicKey = secp.getPublicKey(privateKey);
  const signature = await secp.sign(messageHash, privateKey);
  const isSigned = secp.verify(signature, messageHash, publicKey);

  // Supports Schnorr signatures.
  const signature2 = await secp.schnorr.sign(messageHash, privateKey);
})();
```

Deno:

```typescript
import * as secp from "https://deno.land/x/secp256k1/mod.ts";
const publicKey = secp.getPublicKey("6b911fd37cdf5c81d4c0adb1ab7fa822ed253ab0ad9aa18d77257c88b29b718e");
```

## API

- [`getPublicKey(privateKey)`](#getpublickeyprivatekey)
- [`getSharedSecret(privateKeyA, publicKeyB)`](#getsharedsecretprivatekeya-publickeyb)
- [`sign(hash, privateKey)`](#signhash-privatekey)
- [`verify(signature, hash, publicKey)`](#verifysignature-hash-publickey)
- [`recoverPublicKey(hash, signature, recovery)`](#recoverpublickeyhash-signature-recovery)
- [Helpers](#helpers)

##### `getPublicKey(privateKey)`
```typescript
function getPublicKey(privateKey: Uint8Array, isCompressed?: false): Uint8Array;
function getPublicKey(privateKey: string, isCompressed?: false): string;
function getPublicKey(privateKey: bigint): Uint8Array;
```
`privateKey` will be used to generate public key.
  Public key is generated by doing scalar multiplication of a base Point(x, y) by a fixed
  integer. The result is another `Point(x, y)` which we will by default encode to hex Uint8Array.
`isCompressed` (default is `false`) determines whether the output should contain `y` coordinate of the point.

To get Point instance, use `Point.fromPrivateKey(privateKey)`.

##### `getSharedSecret(privateKeyA, publicKeyB)`
```typescript
function getSharedSecret(privateKeyA: Uint8Array, publicKeyB: Uint8Array): Uint8Array;
function getSharedSecret(privateKeyA: string, publicKeyB: string): string;
function getSharedSecret(privateKeyA: bigint, publicKeyB: Point): Uint8Array;
```

Computes ECDH (Elliptic Curve Diffie-Hellman) shared secret between a private key and a different public key.

To get Point instance, use `Point.fromHex(publicKeyB).multiply(privateKeyA)`.

To speed-up the function massively by precomputing EC multiplications,
use `getSharedSecret(privateKeyA, secp.utils.precompute(8, publicKeyB))`


##### `sign(hash, privateKey)`
```typescript
function sign(msgHash: Uint8Array, privateKey: Uint8Array, opts?: Options): Promise<Uint8Array>;
function sign(msgHash: string, privateKey: string, opts?: Options): Promise<string>;
function sign(msgHash: Uint8Array, privateKey: Uint8Array, opts?: Options): Promise<[Uint8Array | string, number]>;
```

Generates deterministic ECDSA signature as per RFC6979. Asynchronous, so use `await`.

- `msgHash: Uint8Array | string` - message hash which would be signed
- `privateKey: Uint8Array | string | bigint` - private key which will sign the hash
- `options?: Options` - *optional* object related to signature value and format
- `options?.recovered: boolean = false` - determines whether the recovered bit should be included in the result. In this case, the result would be an array of two items.
- `options?.canonical: boolean = false` - determines whether a signature `s` should be no more than 1/2 prime order
- Returns DER encoded ECDSA signature, as hex uint8a / string and recovered bit if `options.recovered == true`.

##### `verify(signature, hash, publicKey)`
```typescript
function verify(signature: Uint8Array, msgHash: Uint8Array, publicKey: Uint8Array): boolean
function verify(signature: string, msgHash: string, publicKey: string): boolean
```
- `signature: Uint8Array | string | { r: bigint, s: bigint }` - object returned by the `sign` function
- `msgHash: Uint8Array | string` - message hash that needs to be verified
- `publicKey: Uint8Array | string | Point` - e.g. that was generated from `privateKey` by `getPublicKey`
- Returns `boolean`: `true` if `signature == hash`; otherwise `false`

##### `recoverPublicKey(hash, signature, recovery)`
```typescript
export declare function recoverPublicKey(msgHash: string, signature: string, recovery: number): string | undefined;
export declare function recoverPublicKey(msgHash: Uint8Array, signature: Uint8Array, recovery: number): Uint8Array | undefined;
```
- `msgHash: Uint8Array | string` - message hash which would be signed
- `signature: Uint8Array | string | { r: bigint, s: bigint }` - object returned by the `sign` function
- `recovery: number` - recovery bit returned by `sign` with `recovered` option
  Public key is generated by doing scalar multiplication of a base Point(x, y) by a fixed
  integer. The result is another `Point(x, y)` which we will by default encode to hex Uint8Array.
  If signature is invalid - function will return `undefined` as result.

To get Point instance, use `Point.fromSignature(hash, signature, recovery)`.

#### Point methods

##### Helpers

###### `utils.generateRandomPrivateKey(): Uint8Array`

Returns `Uint8Array` of 32 cryptographically secure random bytes. You can use it as private key.

###### `utils.precompute(W = 8, point = BASE_POINT): Point`

Returns cached point which you can use to pass to `getSharedSecret` or to `#multiply` by it.

This is done by default, no need to run it unless you want to
disable precomputation or change window size.

We're doing scalar multiplication (used in getPublicKey etc) with
precomputed BASE_POINT values.

This slows down first getPublicKey() by milliseconds (see Speed section),
but allows to speed-up subsequent getPublicKey() calls up to 20x.

You may want to precompute values for your own point.

```typescript
secp256k1.CURVE.P // 2 ** 256 - 2 ** 32 - 977
secp256k1.CURVE.n // 2 ** 256 - 432420386565659656852420866394968145599
secp256k1.Point.BASE // new secp256k1.Point(Gx, Gy) where
// Gx = 55066263022277343669578718895168534326250603453777594175500187360389116729240n
// Gy = 32670510020758816978083085130507043184471273380659243275938904335757337482424n;

// Elliptic curve point in Affine (x, y) coordinates.
secp256k1.Point {
  constructor(x: bigint, y: bigint);
  // Supports compressed and non-compressed hex
  static fromHex(hex: Uint8Array | string);
  static fromPrivateKey(privateKey: Uint8Array | string | number | bigint);
  static fromSignature(
    msgHash: Hex,
    signature: Signature,
    recovery: number | bigint
  ): Point | undefined {
  toRawBytes(isCompressed = false): Uint8Array;
  toHex(isCompressed = false): string;
  equals(other: Point): boolean;
  negate(): Point;
  add(other: Point): Point;
  subtract(other: Point): Point;
  // Constant-time scalar multiplication.
  multiply(scalar: bigint | Uint8Array): Point;
}
secp256k1.SignResult {
  constructor(r: bigint, s: bigint);
  // DER encoded ECDSA signature
  static fromHex(hex: Uint8Array | string);
  toHex(): string;
}
```

## Security

Noble is production-ready & secure. Our goal is to have it audited by a good security expert.

We're using built-in JS `BigInt`, which is "unsuitable for use in cryptography" as [per official spec](https://github.com/tc39/proposal-bigint#cryptography). This means that the lib is potentially vulnerable to [timing attacks](https://en.wikipedia.org/wiki/Timing_attack). But:

1. JIT-compiler and Garbage Collector make "constant time" extremely hard to achieve in a scripting language.
2. Which means *any other JS library doesn't use constant-time bigints*. Including bn.js or anything else. Even statically typed Rust, a language without GC, [makes it harder to achieve constant-time](https://www.chosenplaintext.ca/open-source/rust-timing-shield/security) for some cases.
3. If your goal is absolute security, don't use any JS lib — including bindings to native ones. Use low-level libraries & languages.
4. We however consider infrastructure attacks like rogue NPM modules very important; that's why it's crucial to minimize the amount of 3rd-party dependencies & native bindings. If your app uses 500 dependencies, any dep could get hacked and you'll be downloading rootkits with every `npm install`. Our goal is to minimize this attack vector.
5. Nonetheless we've hardened implementation of koblitz curve multiplication to be algorithmically constant time.

## Speed

Benchmarks measured with 2.9Ghz i9-8950HK.

    getPublicKey(utils.randomPrivateKey()) x 4,017 ops/sec @ 248μs/op
    sign x 2,620 ops/sec @ 381μs/op
    verify x 558 ops/sec @ 1ms/op
    recoverPublicKey x 301 ops/sec @ 3ms/op
    getSharedSecret aka ecdh x 435 ops/sec @ 2ms/op
    getSharedSecret (precomputed) x 4,079 ops/sec @ 245μs/op
    schnorr.sign x 1,643 ops/sec @ 608μs/op
    schnorr.verify x 316 ops/sec @ 3ms/op

Compare to other libraries:

    elliptic#sign x 1,326 ops/sec
    sjcl#sign x 185 ops/sec
    openssl#sign x 1,926 ops/sec
    ecdsa#sign x 69.32 ops/sec

    elliptic#verify x 575 ops/sec
    sjcl#verify x 155 ops/sec
    openssl#verify x 2,392 ops/sec
    ecdsa#verify x 45.64 ops/sec

    (gen is getPublicKey)
    elliptic#gen x 1,434 ops/sec
    sjcl#gen x 194 ops/sec

    elliptic#ecdh x 704 ops/sec

## Contributing

Check out a blog post about this library: [Learning fast elliptic-curve cryptography in JS](https://paulmillr.com/posts/noble-secp256k1-fast-ecc/).

1. Clone the repository.
2. `npm install` to install build dependencies like TypeScript
3. `npm run compile` to compile TypeScript code
4. `npm run test` to run jest on `test/index.ts`

Special thanks to [Roman Koblov](https://github.com/romankoblov), who have helped to improve scalar multiplication speed.

## License

MIT (c) Paul Miller [(https://paulmillr.com)](https://paulmillr.com), see LICENSE file.
