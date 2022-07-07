Uage:
node did.js

Tutorial: How to create a DID on the ION network
Step by step guide on how to create a decentralised identifier on the ION network.

What we’re going to cover in this tutorial:

How to create a DID with ION
How to register (anchor) a DID with ION
How to resolve a DID
Versions used in this Tutorial:

npm: 7.24.0
node: v16.10.0
ion-tools: 0.1.0
Intro
In this tutorial we’re going to create a Decentralised Identity. So what is a Decentralised Identity? As the name already says, it’s an identity and it’s decentralised 😏.

An identity does identify something. This can be anything, a person, a company, a shipping container or a digital artwork. A DID can be used to identify any of those things, and much more.

Decentralised means that there is no central entity that controls your identity. Compare that to a Google email address, which is also an identifier. The difference is that Google fully controls your email account and is able to lock you out at any moment.

In this tutorial we’re going to create and register a Decentralised Identifier using the ION Network. The ION Network runs on top of Bitcoin and ensures that full ownership stays with the DID owners. It just recently moved out of beta status and is available for the general public.

So let’s get started.

Step 1
For this tutorial we’re creating a new directory and initialise a new NPM project. In your terminal type:

mkdir ion-did
cd ion-did
npm init
Step 2
Let’s add the ion-tools dependency:

npm install @decentralized-identity/ion-tools
Step 3
Create a NodeJs file:

touch did.js
This file is going to contain the logic to create and register a DID on ION for us.

Step 4
Let’s open the file did.js and add the ion-tools dependency:

const ION = require('@decentralized-identity/ion-tools')
const fs = require('fs').promises
const main = async () => {
    // all future code goes here
}
main()
Step 5
First we need to create a private/public key pair. The private key will proof our ownership of the DID, the public key will be attached to the DID.
Add this to the filedid.js:

// Create private/public key pair
const authnKeys = await ION.generateKeyPair('secp256k1')
console.log("Created private/public key pair")
console.log("Public key:", authnKeys.publicJwk)
// Write private and public key to files
await fs.writeFile(
  'publicKey.json', 
  JSON.stringify(authnKeys.publicJwk)
)
await fs.writeFile(
  'privateKey.json', 
  JSON.stringify(authnKeys.privateJwk)
)
console.log("Wrote public key to publicKey.json")
console.log("Wrote private key to privateKey.json")
Now you can run the file in the terminal with: node did.js
You should get an output like this:

Created private/public key pair
Public key: {
  kty: 'EC',
  crv: 'secp256k1',
  x: 'UMsfSylDngyuGMY4vlvyYRKynHpfTSPYqvoJLOmJuTc',
  y: 'YT2iT_GBlRlH2LvJ1WqyXnvPtFskjQRjnNVF6mSoL-w'
}
Wrote public key to publicKey.json
Wrote private key to privateKey.json
You should now have two files:

publicKey.json: contains your public key. This will be used to create your DID. It will also be attached to the DID. This allows other users to verify your signatures.
privateKey.json: contains your private key. The private key is proof of your ownership of your DID. Therefore the private key needs to be kept secret.
Step 6
Now we are going to create the DID. For that we will need to specify the DID document. This document will contain information about what this DID can be used for. For this tutorial, the public key is also added to the DID for the purpose of authentication e.g. for allowing the owner to log-in to 3rd party services. Additionally an Identity Hub service is added, which allows the DID owner to manage sensitive personal information.

Add this to your code:

// Create a DID
const did = new ION.DID({
  content: {
    // Register the public key for authentication
    publicKeys: [
      {
        id: 'auth-key',
        type: 'EcdsaSecp256k1VerificationKey2019',
        publicKeyJwk: authnKeys.publicJwk,
        purposes: [ 'authentication' ]
      }
    ],
    // Register an IdentityHub as a service
    services: [
      {
        id: "IdentityHub",
        type: "IdentityHub",
        serviceEndpoint: {
          "@context": "schema.identity.foundation/hub",
          "@type": "UserServiceEndpoint",
          instance: [
            "did:test:hub.id",
          ]
        }
      }
    ]
  }
})
const didUri = await did.getURI('short');
console.log("Generated DID:", didUri)
If you run the file you’ll get the DID Uri:

Generated DID: did:ion:EiCjHFpU1Fm6Qnq7XIj_Gt2QGCpnwQrrnUWoVrM4H9we1A
Step 7
We now successfully created a DID. However it is not yet registered anywhere. In order to do that we need to “anchor” the DID. This means the DID is registered on the ION network, which itself runs on top of Bitcoin.

To anchor the DID we need to create and submit an anchor request. Add this to your file:

const anchorRequestBody = await did.generateRequest()
const anchorRequest = new ION.AnchorRequest(anchorRequestBody)
const anchorResponse = await anchorRequest.submit()
console.log(JSON.stringify(anchorResponse))
If you run the file you will get an output similar to:

Getting challenge from: https://beta.ion.msidentity.com/api/v1.0/proof-of-work-challenge
{
  challengeNonce: '...',
  validDurationInMinutes: 10,
  largestAllowedHash: '0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff'
}
Solving for body:
{...}
9fc2188b8c75aea0fb0c4256075f1e4a328cbfb0f7078f70882d4f9bf84f8360
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
e31c1896afd20fd105b4f18ef19ddc0e03586717d214fc587b474ae7539aaad9
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
5953405ea04cbde4203c083951c1d5c7babe869d54c08c490c7bb8236f6b70c6
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
8f07ef2c5d977514a7e841ff746e19f851d92c9a2e831f8ec7b59a5365a83047
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
e58a53e82a44c96c4fdbcd428bdb27a547d32ce7098061f7e2db32684034a0f1
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
fb91d4b873bcb90325adca0becb7c681708b9e679537489503502ac46d06e595
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
5b928168542f66d5ec461a729fd329899c4ba1e499aa72afcab286c83b850541
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
6007335138a9695407111a6a721f90ae96672d5d0ca40bc261a2f08e642e1bea
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
95aed107ef50b0a66289f518291e7e2cd1570c6c015fbb810610b8ac68b299ca
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
426e14e497abbddd7e149c66e5806864583cac274ed54a999a9bd9b2fd620031
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
48f8ded2adf1b910e88437c35f35d78114cd814bad0b88e1edfaca62893775b2
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
2556f5e0f097c014054335fc402122a692adb841081167aba70344e8310584e6
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
0d8058cdbc3a81706e70db54ffef2f2c916e8600e48e9e790787fc02aebfe63e
0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
3
Successful registration
{
  "@context": "https://w3id.org/did-resolution/v1",
  "didDocument": {
    "id": "did:ion:EiCjHFpU1Fm6Qnq7XIj_Gt2QGCpnwQrrnUWoVrM4H9we1A",
    "@context": [
      "https://www.w3.org/ns/did/v1",
      {
        "@base": "did:ion:EiCjHFpU1Fm6Qnq7XIj_Gt2QGCpnwQrrnUWoVrM4H9we1A"
      }
    ],
    "service": [
      {
        "id": "#IdentityHub",
        "type": "IdentityHub",
        "serviceEndpoint": {
          "@context": "schema.identity.foundation/hub",
          "@type": "UserServiceEndpoint",
          "instance": [
            "did:test:hub.id"
          ]
        }
      }
    ],
    "verificationMethod": [
      {
        "id": "#auth-key",
        "controller": "",
        "type": "EcdsaSecp256k1VerificationKey2019",
        "publicKeyJwk": {
          "kty": "EC",
          "crv": "secp256k1",
          "x": "53hsUK6agrBUYX7rxN-LZR60s8_XcfAF2iqug4Kv-UE",
          "y": "hSWYRwXHiNm4oJPyakWoqBAtIx8Zxwzh52U2Oytzjfk"
        }
      }
    ],
    "authentication": [
      "#auth-key"
    ]
  },
  "didDocumentMetadata": {
    "method": {
      "published": false,
      "recoveryCommitment": "EiAXhWTi_9eezd-GZRvOgBSRGhFTJd-jROr5YyylAgYLNA",
      "updateCommitment": "EiDFhyN3--vu481IfuOTO2JtMrVYzwK_dWLMMX1DWceXeg"
    },
    "canonicalId": "did:ion:EiCjHFpU1Fm6Qnq7XIj_Gt2QGCpnwQrrnUWoVrM4H9we1A"
  }
}
When anchoring, we’re sending the DID registration request to an ION node. The ION node sends back a challenge that needs to be solved by the DID owner (all of this is automated by the ion-tools package).

After successfully solving the challenge the DID is registered. In the logs you can see final DID document with the service and verification method that were registered.

Step 8 ☕
The DID was now anchored, but it is not published yet. In the meta-data of the DID document returned in the previous step you can see the property "published": false.

It can still take a while until the DID is published. For that to happen we need to wait until the ION network “anchors” a list of DIDs (including yours) on the Bitcoin blockchain.

You can use the ION Explorer to check online if your DID was already published: https://identity.foundation/ion/explorer/

There you can search for your DID. Once your DID was published, you will see the DID document like this:


You can also try to resolve your DID using the Universal Resolver: https://dev.uniresolver.io/

You can also “resolve” the DID document programmatically using ion-tools. To do that simply run this code:

const resolvedDid = await ION.resolve(
  'did:ion:EiCjHFpU1Fm6Qnq7XIj_Gt2QGCpnwQrrnUWoVrM4H9we1A'
)
console.log(JSON.stringify(response))
Outlook
Now that you created a new DID and published it, you can use this DID to sign arbitrary data, for example to create verifiable credentials.

Using ion-tools you can can sign and verify arbitrary data like this:

const privateKey = JSON.parse(await fs.readFile('privateKey.json'))
const myData = 'This message is signed and cannot be tampered with'
const signature = await ION.signJws({
  payload: myData,
  privateJwk: privateKey
});
console.log("Signed JWS:", signature)
const randomKeyPair = await ION.generateKeyPair('secp256k1')
let verifiedJws = await ION.verifyJws({
  jws: signature,
  publicJwk: randomKeyPair.publicJwk
})
console.log("Verify with random new key:", verifiedJws)
const publicKey = JSON.parse(await fs.readFile('publicKey.json'))
verifiedJws = await ION.verifyJws({
  jws: signature,
  publicJwk: publicKey
})
console.log("Verify with my public key:", verifiedJws)
First we create a signature for our data: “This message is signed and cannot be tampered with”. At this point we could use any data to sign, for example a stringified JSON object.

As you can see in the example, the signature can the only be verified with the correct public key: our DID public key in this case.