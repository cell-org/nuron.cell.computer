![nuron.png](nuron.png)

# Nuron

> NFT Management System


Nuron is an NFT management system that lets you create and manage NFTs. It is made up of:

1. **Programmable Wallet:** automatically sign messages by POSTing messages to sign over HTTP
2. **NFT File System:** store and manage IPFS files both locally and publish when ready
3. **NFT Database:** store signed tokens locally and publish when ready

> If you're looking for a way to download nuron, check out the Tutorial here: https://tutorial.cell.computer/#/?id=_2-download-nuron

## Usage

Once installed, you can interact with your Nuron using HTTP requests:

1. Use the HTTP API directly: You can use whatever programming language you want (python, javascript, ruby, etc.)
2. Use a JavaScript library: [nuron.js](https://nuronjs.cell.computer) is just a wrapper library that makes it easy to interact with a Nuron using simple JavaScript code.

## Making requests to Nuron

You don't have to use **nuron.js**. You can build your own Nuron client, or just directly program networking code to interact with Nuron, using any programming language. It's simple because Nuron is just a local HTTP server.

Check out the API reference section below for all available endpoints. For example, for `GET /wallet/accounts`, you can do this:

```javascript
let accounts = await fetch("http://localhost:42000/wallet/accounts").then((res) => {
  return res.json()
})
```

---

# API

## wallet

### GET /wallet/accounts

Get all local account names (The names you save seed phrases under:)

```
GET /wallets/accounts
```

returns:

```json
{
  "response": [
    <account name>,
    <account name>,
    ...
  ]
}
```

### GET /wallet/session

```
GET /wallet/session?path=<bip44 derivation path>
```

returns the corresponding account for the bip44 derivation path:

```json
{
  "response": {
    "username": <account name>,
    "address": <address>
  }
}
```

- if logged in
  - if the `bip44 derivation path` was passed, returns both the `username` and the `address` of the wallet address at the derivation path.
  - if the `bip44 derivation path` was not passed, only returns the `username` of the current session.
- if not logged in, returns the response with a `username` of `null`


### GET /wallet/chains/:chainId

Get first 100 addresses derived from the currently logged-in wallet for the specified chainId

```
GET /wallet/chains/:chainId
```

returns

```json
{
  "response": {
    "name": <logged in account name>,
    "chain": <the blockchain symbol>,
    "chainId": <the requested blockchain chainId>,
    "accounts": [{
      "address": <address 0>,
      "path": <bip44 derivation path 0>,
    }, {
      "address": <address 1>,
      "path": <bip44 derivation path 1>,
    }, {
      ...
    }, {
      "address": <address 99>,
      "path:" <bip44 derivation path 99>,
    }]
  }
}
```


### POST /wallet/connect

```
POST /wallet/connect

{
  "username": <username>,
  "password": <key encryption password>
}
```

returns:

```json
{
  "success": true
}
```

or if there's an error:

```json
{
  "error": <error message>
}
```

### POST /wallet/disconnect

Sign out of the current session

```
POST /wallet/disconnect
```

returns:

```json
{
  "success": true
}
```

or if there's an error:

```json
{
  "error": <error message>
}
```

### POST /wallet/delete

delete the wallet for the current session. Will only delete successfully if the encryption password is correct

```
POST /wallet/delete

{
  "password": <key encryption password>
}
```

returns:

```json
{
  "success": true
}
```

or if there's an error:

```json
{
  "error": <error message>
}
```

### POST /wallet/export

Export a wallet 

```
POST /wallet/export

{
  "key": <bip44 derivation path>,
  "password": <key encryption password>
}
```

returns:

```json
{
  "response": <exported value (seed phrase or private key)>
}
```

- if the `key` attribute is included in the payload
  - the private key (hex format) at the bip44 derivation path will be exported
- if the `key` attribute is NOT included in the payload
  - the seed phrase is exported


### POST /wallet/import

Import a `seed` phrase, encrypt it with `password`, and store it under `username` account on Nuron.

```
POST /wallet/import

{
  "response": {
    "seed": <seed phrase>,
    "username": <username to assign to the new wallet>,
    "password": <key encryption password>
  }
}
```

returns:

```json
{
  "response": {
    "username": <the username for the imported wallet>,
    "seed": <the imported seed phrase>,
    "encrypted": <the encrypted seed phrase>
  }
}
```

or if there's an error:

```json
{
  "error": <error message>
}
```


### POST /wallet/generate

generate a random seed phrase, encrypt it with the `password` and store it under the `username` account on Nuron.

```
POST /wallet/generate

{
  "username": <username>,
  "password": <key encryption password>
}
```

returns:

```json
{
  "response": {
    "username": <the username for the generated wallet>,
    "seed": <the generated seed phrase>,
    "encrypted": <the encrypted seed phrase>
  }
}
```

or if there's an error:

```json
{
  "error": <error message>
}
```


## fs

### GET /fs/raw/:path

Get the files at `:path` within the Nuron file system.

For example,

```
GET /fs/raw/cube/index.html
```

returns the `index.html` file under the `cube` workspace folder



### GET /fs/config

Get the current file system configuration

```
GET /fs/config
```

returns

```json
{
  "ipfs": {
    "key": <nft.storage APK key>
  },
  "workspace": {
    "home": <the folder path where the nuron stores everything>
  }
}
```

### POST /fs/configure

```
POST /fs/configure

{
  "ipfs": {
    "key": <nft.storage API key>
  }
}
```

if successful, returns:

```json
{
  "success": true
}
```

if failed, returns the error message:

```json
{
  "error": <error message>
}
```

### POST /fs/binary

Ask Nuron to save a Buffer object to the file system. Here's an example JavaScript code to make the POST request:


```javascript
let save = (data) => {
  let type = data.constructor.name;
  let blob;
  if (type === 'ArrayBuffer') {
    // If ArrayBuffer, transform into Blob
    blob = new Blob([data])
  } else if (type === "File" || type === "Blob" || type === "Buffer") {
    // Otherwise if they're File, Blob, or Buffer, use the data as is
    blob = data
  }

  // Create a formdata object
  let fd = new FormData()
  // Attach the blob under the "file" field
  fd.append('file', blob, { filename: "filename" })
  // Attach the workspace folder path to store the file under
  fd.append('workspace', workspace_folder_path)
  // Make the POST request
  let r = await fetch("/fs/binary", {
    method: "POST",
    body: fd
  }).then((res) => {
    return res.json()
  })
}
```

### POST /fs/json

save a JSON to the file system. Here's an example JavaScript code for posting to the `/fs/object` endpoint:

```javascript
let save = (json) => {
  let r = await fetch("/fs/json", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(json)
  }).then((res) => {
    return res.json()
  })
}
```

### POST /fs/rm

remove cids from the `fs` folder

```
POST /fs/rm

{
  "workspace": <the nuron workspace path>,
  "cids": <an array of IPFS cids to remove from the fs folder>
}
```

The `cids` can be:

- **an array of IPFS CIDs:** remove all files with the CIDs
- **a single IPFS CID:** remove a single file
- `*`: remove all files under the `fs` folder in the specified workspace

if successful, returns

```json
{
  "success": true
}
```

if failed, returns:

```json
{
  "error": <error message>
}
```


### POST /fs/pin

```
POST /fs/pin

{
  "workspace": <nuron workspace folder path>,
  "cid": <The IPFS cid of the file to pin>
}
```

where:

- `cid` can be one of the following:
  - **<cid>:** The IPFS CID of the file to pin
  - **"*": ** Pin the entire `fs` folder

returns

```json
{
  "response": <pinned cid>
}
```

Lf you want to pin the entire `fs` folder within the Nuron project folder `canvas`, you can make a POST request without the `cid` attribute:

```json
POST /fs/pin

{
  "workspace": "canvas",
}
```

If you want to pin a specific file inside the `fs` folder, you can do:

```json
POST /fs/pin

{
  "workspace": "canvas",
  "cid": "bafkreihly2rt63htnzhxqjgixknywcty53zhcxmyskj3l2gzwdchr6adle""
}
```


## token

### POST /token/build

builds an unsigned token according to the payload description:

```
POST /token/build

{
  "payload": {
    "body": <token body description>,
    "domain": <token domain description>
  }
}
```

returns the generated **unsigned token**:

```json
{
  "body": <generated token body (NO signature)>,
  "domain": <generated token domain>
}
```


### POST /token/sign

signs an unsigned token to create a signed token:

```
POST /token/sign

{
  "key": <bip44 derivation path>,
  "payload": <unsigned token (constructed with build)>
}
```

signs the unsigned token and attaches the signature and returns a **signed token**, which has the exact same schema as an unsigned token, except that the signature is attached at `body.signature` path of the JSON:

```json
{
  "body": <token body with "signature" attached>,
  "domain": <generated token domain>
}
```


### POST /token/create

```
POST /token/create

{
  "key": <bip44 derivation path>,
  "payload": {
    "body": <token body description>,
    "domain": <token domain description>
  }
}
```

returns the generated token:

```json
{
  "body": <generated token body, with signature>,
  "domain": <token domain>
}
```

## db

The Nuron DB is powered by [Mixtape](https://mixtape.papercorp.org).


### POST /db/write


save objects to the database at `mixtape.db`.

```
POST /db/write

{
  "workspace": <nuron workspace folder path>,
  "table": <table name>,
  "payload": <signed or unsigned token, or a metadata object>
}
```

the `table` attribute can be one of the following:

1. `token`: A table that can be used to store all the signned and unsigned tokens
2. `metadata`: A table that can be used to store all the NFT metadata

returns the generated IPFS CID for the token:

if successful, returns:

```json
{
  "success": true
}
```

or if there's an error:

```json
{
  "error": <error message>
}
```

### POST /db/read


read the sqlite database stored inside the file `mixtape.db` at a specified path:

```
POST /db/read

{
  "workspace": <nuron workspace folder path>,
  "table": <table name>,
  "payload": <query language>
}
```

the `table` attribute can be one of the following:

1. `token`: A table that can be used to store all the signned and unsigned tokens
2. `metadata`: A table that can be used to store all the NFT metadata


If successful, returns:

```json
{
  "response": <the result array of the Mixtape read() method>
}
```

If failed, returns:

```json
{
  "error": <error message>
}
```

### POST /db/readOne


read the sqlite database stored inside the file `mixtape.db` at a specified path:

```
POST /db/readOne

{
  "workspace": <nuron workspace folder path>,
  "table": <table name>,
  "payload": <query language>
}
```

the `table` attribute can be one of the following:

1. `token`: A table that can be used to store all the signned and unsigned tokens
2. `metadata`: A table that can be used to store all the NFT metadata


If successful, returns:

```json
{
  "response": <the result object of the Mixtape readOne() method>
}
```

If failed, returns:

```json
{
  "error": <error message>
}
```


### POST /db/rm

Deletes the rows that match the query

```
POST /db/rm

{
  "workspace": <nuron workspace folder path>,
  "table": <table name>,
  "payload": <query language>
}
```

the `table` attribute can be one of the following:

1. `token`: A table that can be used to store all the signned and unsigned tokens
2. `metadata`: A table that can be used to store all the NFT metadata


If successful, returns:

```json
{
  "success": true
}
```

If failed, returns:

```json
{
  "error": <error message>
}
```


## web

Build a static site for the workspace at path `workspace` for the tokens of `domain`. Builds the following web pages, which you can publish immediately to the web and let people mint:

- `index.html`: the home page that dieplays all tokens
- `token.html`: individual token minting page

```
POST /web/build

{
  "workspace": <nuron workspace folder path>,
  "domain": <token domain>,
  "templates": <templates array (optional)>
}
```

Where `<templates array>` is an array of template mapping objects, each of which has the following attributes:

- `src`: the ejs template location (HTTP URL or file path)
- `dest`: the file names to create render from the `src` template files.

Here's an example:


```json
{
  "workspace": <nuron workspace folder path>,
  "domain": <token domain>,
  "templates": [{
    "src": "https://myserver.com/index.ejs",
    "dest": "index.html"
  }, {
    "src": "https://myserver.com/token.ejs",
    "dest": "token.html"
  }]
}
```


if successful, returns:

```json
{
  "success": true
}
```

or if there's an error:

```json
{
  "error": <error message>
}
```
