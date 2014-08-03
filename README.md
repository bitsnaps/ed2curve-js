ed2curve.js
===========

Convert Ed25519 signing key pair into Curve25519 key pair suitable for
Diffie-Hellman key exchange. This means that by exchanging only 32-byte
Ed25519 public keys users can both sign and encrypt with NaCl.

Note that there's currently [no proof](http://crypto.stackexchange.com/a/3311/291)
that this is safe to do. It is safer to share both Ed25519 and Curve25519
public keys (their concatenation is 64 bytes long).

Written by Dmitry Chestnykh in 2014. Public domain. No warranty.

Thanks to [@CodesInChaos](https://github.com/CodesInChaos) and
[@nightcracker](https://github.com/nightcracker) for showing how to
convert Edwards coordinates to Montgomery coordinates.

[![Build Status](https://travis-ci.org/dchest/ed2curve-js.svg?branch=master)
](https://travis-ci.org/dchest/ed2curve-js)


Installation
------------

Via NPM:

    $ npm install ed2curve

Via Bower:

    $ bower install ed2curve


or just download `ed2curve.js` or `ed2curve.min.js` and include it after
[TweetNaCl.js](https://github.com/dchest/tweetnacl-js):

```html
<script src="nacl.min.js"></script>
<script src="ed2curve.min.js"></script>
```

Usage
-----

### ed2curve.convertKeyPair(keyPair)

Converts the given key pair as generated by
[TweetNaCl.js](https://github.com/dchest/tweetnacl-js)'s `nacl.sign.keyPair`
into a key pair suitable for operations which accept key pairs generated by
`nacl.box.keyPair`. This function is a combination of `convertPublicKey`
and `convertSecretKey`.

### ed2curve.convertPublicKey(edPublicKey)

Converts a 32-byte Ed25519 public key into a 32-byte Curve25519 public key
and returns it.

### ed2curve.convertSecretKey(edSecretKey)

Converts a 64-byte Ed25519 secret key (or just the first 32-byte part of it,
which is the secret value) into a 32-byte Curve25519 secret key and returns it.


Example
-------

```javascript
// Generate new sign key pair.
var myKeyPair = nacl.sign.keyPair();

// Share public key with a peer.
console.log(myKeyPair.publicKey);

// Receive peer's public key.
var theirPublicKey = // ... receive

// Sign a message.
var message = nacl.util.decodeUTF8('Hello!');
var signedMessage = nacl.sign(message, myKeyPair.secretKey);

// Send message to peer. They can now verify it using
// the previously shared public key (myKeyPair.publicKey).
// ...

// Receive a signed message from peer and verify it using their public key.
var theirSignedMessage = // ... receive
var theirMessage = nacl.sign.open(theirSignedMessage, theirPublicKey);
if (theirMessage) {
  // ... we got the message ...
}

// Encrypt a message to their public key.
// But first, we need to convert our secret key and their public key
// from Ed25519 into the format accepted by Curve25519.
//
// Note that peers are not involved in this conversion -- all they need
// to know is the signing public key that we already shared with them.

var theirDHPublicKey = ed2curve.convertPublicKey(theirPublicKey);
var myDHSecretKey = ed2curve.convertSecretKey(myKeyPair.secretKey);

var anotherMessage = nacl.util.decodeUTF8('Keep silence');
var encryptedMessage = nacl.box(anotherMessage, theirDHPublicKey, myDHSecretKey);

// When we receive encrypted messages from peers,
// we need to use converted keys to open them.

var theirEncryptedMessage = // ... receive
var decryptedMessage = nacl.box.open(theirEncryptedMessage, theirDHPublicKey, myDHSecretKey);
```

Requirements
------------

* Requires [TweetNaCl.js](https://github.com/dchest/tweetnacl-js)
* Works in the same enviroments as it.
