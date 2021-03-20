# Signing Payloads

Signing a payload is a way of verifying its **authenticity**, when placed on the CIA triad (authorization, authentication, and avialability) - payload signing is falls under *authentication* because we are ensuring that the sender was able to sign the payload using a pre-shared key.

What payload singing is NOT is encryption, we are not encrypting the payload and it can still be seen and read by anyone. The layer we are adding is a verification hash field of the payloads contents between two endpoints, the server hashes the payload -> the client reads the payloads and hashes it, then compares the hashes to ensure that the payload was not tampered with.

## Signing with SHA1

Lets split it up into two components.

### Encrypting the payload

```js
const payload = {
    hello: "world"
};

const signPayload = (payload) => {
    const crypto = require("crypto");

    // pre-shared secret
    const secret = "secret_password";

    // hash the payload
    const payloadHash = crypto
                            .createHmac("sha1", secret)
                            .update(JSON.stringify(payload))
                            .digest("hex");

    // create the signature with the hashing type
    const sig = `sha1=${payloadHash}`;

    // return the hashed payload string
    return sig
}
```

Next we want to send the payload to the client.

```js
// get the payload hash
const payload = {
        hello: "world"
};

const payloadHash = signPayload(payload)

// Create some payload endpoint to post to
const url = "client.com/endpoint"

// Create a fetch options object
const options = {
    method: "POST",
    const headers = {
        "x-payload-signature": sig,
    },
    body: JSON.stringify(payload)
}

// Do the fetch!
fetch(url, options)
```

### Validating the payload

Now we need to accept the payload on the client, and compare hashes.
To do this, we recycle the `signPayload` function (so signPayload exists on the sender and receiver) and pass it the parsed JSON from the received payload.

To extract the hashed payload from the payload you can use something like this.

```js
// Using express we can get the information we need from the req object
const payloadHash = req.headers["x-payload-signature"];
const payload = req.body;
validatePayload(payload, payloadHash)
```

```js
const validatePayload = (senderPayload, senderHash) => {
    // Run a check on the received payload
    const ourHash = signPayload(JSON.parse(senderPayload));

    // check if they match
    if (senderHash === ourHash) {
        return true;
    } else {
        return false;
    }
}
```

## Signing with JWT

Coming soon...
