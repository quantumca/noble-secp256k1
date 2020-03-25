# noble-secp256k1

[secp256k1](https://www.secg.org/sec2-v2.pdf), an elliptic curve that could be used for assymetric encryption, ECDH key agreement protocol and deterministic ECDSA signature scheme from RFC 6939.

Algorithmically resistant to timing attacks. Tested against thousands of vectors from tiny-secp256k1.

### This library belongs to *noble* crypto

> **noble-crypto** — high-security, easily auditable set of contained cryptographic libraries and tools.

- No dependencies, one small file
- Easily auditable TypeScript/JS code
- Uses es2019 bigint. Supported in Chrome, Firefox, node 10+
- All releases are signed and trusted
- Check out all libraries:
  [secp256k1](https://github.com/paulmillr/noble-secp256k1),
  [ed25519](https://github.com/paulmillr/noble-ed25519),
  [bls12-381](https://github.com/paulmillr/noble-bls12-381),
  [ripemd160](https://github.com/paulmillr/noble-ripemd160),
  [secretbox-aes-gcm](https://github.com/paulmillr/noble-secretbox-aes-gcm)

## Usage

> npm install noble-secp256k1

```js
import * as secp256k1 from "noble-secp256k1";

// You can also pass Uint8Array and BigInt.
const privateKey = "00000000000000000000000000000000000000a665a45920422f9d417e4867ef";
const messageHash = "9c1185a5c5e9fc54612808977ee8f548b2258d31";
const publicKey = secp256k1.getPublicKey(privateKey);
const signature = secp256k1.sign(messageHash, privateKey);
const isMessageSigned = secp256k1.verify(signature, messageHash, publicKey);

// Or:
// const privateKey = Uint8Array.from([
//   0xa6, 0x65, 0xa4, 0x59, 0x20, 0x42, 0x2f,
//   0x9d, 0x41, 0x7e, 0x48, 0x67, 0xef
// ]);
// const privateKey = 0xa665a45920422f9d417e4867efn;
```

## API

- [`getPublicKey(privateKey)`](#getpublickeyprivatekey)
- [`getSharedSecret(privateKeyA, publicKeyB)`](#getsharedsecretprivatekeya-publickeyb)
- [`sign(hash, privateKey)`](#signhash-privatekey)
- [`verify(signature, hash)`](#verifysignature-hash)
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

##### `verify(signature, hash)`
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

`utils.precompute(W = 4, point = BASE_POINT)`

This is done by default, no need to run it unless you want to
disable precomputation or change window size.

We're doing scalar multiplication (used in getPublicKey etc) with
precomputed BASE_POINT values.

This slows down first getPublicKey() by milliseconds (see Speed section),
but allows to speed-up subsequent getPublicKey() calls up to 20x.

The precomputation window is variable. For example, we increase W to 8
for tests, to speed-up tests 2x.

You may want to precompute values for your own point.

```typescript
// 𝔽p
secp256k1.P // 2 ** 256 - 2 ** 32 - 977

// Prime order
secp256k1.PRIME_ORDER // 2 ** 256 - 432420386565659656852420866394968145599

// Base point
secp256k1.BASE_POINT // new secp256k1.Point(x, y) where
// x = 55066263022277343669578718895168534326250603453777594175500187360389116729240n
// y = 32670510020758816978083085130507043184471273380659243275938904335757337482424n;

// Elliptic curve point
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
  toHex(): string;
  add(other: Point): Point;
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

## Speed

Measured with 2.9Ghz Coffee Lake.

- `getPrivateKey()`: 3.5ms
- `sign()`: 30ms
- `getSharedSecret()`: 27ms
- precomputation: first `getPrivateKey()` or `utils.precompute()`: 65ms
- `getPrivateKey()` with `utils.precompute(8)` instead of `4`: 1.75ms

## Security

Noble is production-ready & secure. Our goal is to have it audited by a good security expert.

We're using built-in JS `BigInt`, which is "unsuitable for use in cryptography" as [per official spec](https://github.com/tc39/proposal-bigint#cryptography). This means that the lib is potentially vulnerable to [timing attacks](https://en.wikipedia.org/wiki/Timing_attack). But:

1. JIT-compiler and Garbage Collector make "constant time" extremely hard to achieve in a scripting language.
2. Which means *any other JS library doesn't use constant-time bigints*. Including bn.js or anything else. Even statically typed Rust, a language without GC, [makes it harder to achieve constant-time](https://www.chosenplaintext.ca/open-source/rust-timing-shield/security) for some cases.
3. If your goal is absolute security, don't use any JS lib — including bindings to native ones. Use low-level libraries & languages.
4. We however consider infrastructure attacks like rogue NPM modules very important; that's why it's crucial to minimize the amount of 3rd-party dependencies & native bindings. If your app uses 500 dependencies, any dep could get hacked and you'll be downloading rootkits with every `npm install`. Our goal is to minimize this attack vector.
5. We've hardened implementation of koblitz curve multiplication to be algorithmically timing-resistant.

## Contributing

1. Clone the repository.
2. `npm install` to install build dependencies like TypeScript
3. `npm run compile` to compile TypeScript code
4. `npm run test` to run jest on `test/index.ts`

## License

MIT (c) Paul Miller [(https://paulmillr.com)](https://paulmillr.com), see LICENSE file.
