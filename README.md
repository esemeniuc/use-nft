<p align=center><img src=https://user-images.githubusercontent.com/36158/112562506-52184100-8dcf-11eb-95ae-88d5dfb06f4a.png>

[![npm version](https://badgen.net/npm/v/use-nft)](https://www.npmjs.com/package/use-nft) [![bundle size](https://badgen.net/bundlephobia/minzip/use-nft)](https://bundlephobia.com/result?p=use-nft) [![License](https://badgen.net/github/license/spectrexyz/use-nft)](https://github.com/spectrexyz/use-nft/blob/main/LICENSE)

useNft() allows to access the metadata of any NFT ([EIP 721](https://eips.ethereum.org/EIPS/eip-721), [EIP 1155](https://eips.ethereum.org/EIPS/eip-1155) and [more](https://www.larvalabs.com/cryptopunks)) on the Ethereum blockchain.

## Install

```console
yarn add use-nft
```

## Usage

useNft() uses a concept of “fetchers”, in order to provide different ways to retrieve data from Ethereum. If you use the [Ethers](https://github.com/ethers-io/ethers.js) in your app, using `ethersFetcher()` is recommended. Otherwise you can use `ethereumFetcher()`, which only requires a [standard Ethereum provider](https://eips.ethereum.org/EIPS/eip-1193), like the one provided by MetaMask.

```jsx
import ethers from "ethers"
import { NftProvider, useNft } from "use-nft"

// We are using the "ethers" fetcher here.
const fetcher = ["ethers", { ethers, provider: ethers.getDefaultProvider() }]

// Alternatively, you can use the "ethereum" fetcher. Note
// that we are using window.ethereum here (injected by wallets
// like MetaMask), but any standard Ethereum provider would work.
// const fetcher = ["ethereum", { ethereum }]

// Wrap your app with <NftProvider />.
function App() {
  return (
    <NftProvider fetcher={fetcher}>
      <Nft />
    </NftProvider>
  )
}

// useNft() is now ready to be used in your app. Pass
// the NFT contract and token ID to fetch the metadata.
function Nft() {
  const { loading, error, nft } = useNft(
    "0xd07dc4262bcdbf85190c01c996b4c06a461d2430",
    "90473"
  )

  // nft.loading is true during load.
  if (loading) return "Loading…"

  // nft.error is an Error instance in case of error.
  if (error) return "Error."

  // You can now display the NFT metadata.
  return (
    <section>
      <h1>{nft.name}</h1>
      <img src={nft.image} alt="" />
      <p>{nft.description}</p>
      <p>Owner: {nft.owner}</p>
      <p>Metadata URL: {nft.metadataUrl}</p>
    </section>
  )
}
```

## API

### `useNft(contract, tokenId)`

The useNft() hook requires two arguments: the NFT `contract` address, and its token ID.

The returned value is an object containing information about the loading state:

```tsx
const result = useNft("0xd07dc4262bcdbf85190c01c996b4c06a461d2430", "90473")

// one of "error", "loading" and "done"
result.status

// same as status === "loading"
result.loading

// undefined or Error instance when status === "error"
result.error

// call this function to retry in case of error
result.reload

// nft is undefined when status !== "done"
result.nft

// name of the NFT (or empty string)
result.nft.name

// description of the NFT (or empty string)
result.nft.description

// image / media URL of the NFT (or empty string)
result.nft.image

// current owner of the NFT (or empty string)
result.nft.owner

// url of the json containing the NFT's metadata
result.nft.metadataUrl
```

As TypeScript type:

```tsx
type NftResult = {
  status: "error" | "loading" | "done"
  loading: boolean
  reload: () => void
  error?: Error
  nft?: {
    description: string
    image: string
    name: string
    owner: string
    metadataUrl?: string
  }
}
```

### `<NftProvider />`

NftProvider requires a prop to be passed: `fetcher`. It can take a declaration for the embedded fetchers, or you can alternatively pass a custom fetcher.

#### `fetcher`

##### With Ethers.js

```tsx
<NftProvider fetcher={["ethers", { ethers, provider }]} />
```

- `ethers` is the default import of the Ethers library (note that only `{ Contract }` is needed, so you can pass this instead).
- `provider` is a [provider](https://docs.ethers.io/v5/api/providers/) from the Ethers library (not to be mistaken with [standard Ethereum providers](https://eips.ethereum.org/EIPS/eip-1193)).

##### With an Ethereum provider

```tsx
<NftProvider fetcher={["ethereum", { ethereum }]} />
```

`ethereum` is a [standard Ethereum providers](https://eips.ethereum.org/EIPS/eip-1193).

##### Custom fetcher

A fetcher is an object implementing the `Fetcher` type:

```tsx
type Fetcher<Config> = {
  config: Config
  fetchNft: (contractAddress: string, tokenId: string) => Promise<NftMetadata>
}
type NftMetadata = {
  name: string
  description: string
  image: string
}
```

See the implementation of the [Ethers](https://github.com/spectrexyz/use-nft/blob/38bd803f20e778b9bb684d682c194a812a94a05c/src/fetchers/ethers/index.tsx#L12-L42) and [Ethereum](https://github.com/spectrexyz/use-nft/blob/38bd803f20e778b9bb684d682c194a812a94a05c/src/fetchers/ethereum/index.tsx#L12-L42) fetchers for more details.

#### `ipfsUrl`

A function that allows to define the IPFS gateway (defaults to `https://ipfs.io/`).

Default value:

```js
function ipfsUrl(cid, path = "") {
  return `https://ipfs.io/ipfs/${cid}${path}`
}
```

#### `imageProxy`

Allows to proxy the image URL. This is useful to optimize (compress / resize) the raw NFT images by passing the URL to a service.

Default value:

```js
function imageProxy(url) {
  return url
}
```

#### `jsonProxy`

Allows to proxy the JSON URL. This is useful to get around the CORS limitations of certain NFT services.

Default value:

```js
function jsonProxy(url) {
  return url
}
```

### FetchWrapper

`FetchWrapper` is a class that allows to use the library with other frontend libraries than React, or with NodeJS. Unlike the `useNft()` hook, `FetchWrapper#fetchNft()` does not retry, cache, or do anything else than attempting to fetch the NFT data once.

```js
import { FetchWrapper } from "use-nft"
```

Pass the fetcher declaration to the `FetchWrapper` and call the `fetchNft` function to retreive the NFT data.

```js
// See the documentation for <NftProvider /> fetcher prop
const fetcher = ["ethers", { ethers, provider: ethers.getDefaultProvider() }]

const fetchWrapper = new FetchWrapper(fetcher)

// You can also pass options to the constructor (same as the <NftProvider /> props):
// const fetchWrapper = new FetchWrapper(fetcher, {
//   ipfsUrl: (cid, path) => `…`,
//   imageProxy: (url) => `…`,
//   jsonProxy: (url) => `…`,
// })

const result = await fetchWrapper.fetchNft(
  "0xd07dc4262bcdbf85190c01c996b4c06a461d2430",
  "90473"
)
```

The `fetchNft()` function returns a promise which resolves to an `NftMetadata` object.

## Supported NFT formats

Any standard NFT ([EIP 721](https://eips.ethereum.org/EIPS/eip-721) or [EIP 1155](https://eips.ethereum.org/EIPS/eip-1155)) is, in theory supported by useNft(). In practice, some adjustments are needed to support some NFT formats, either because their implementation doesn’t follow the specification or because some parts of the specifications can be interpreted in different ways.

This table keeps track of the NFT minting services that have been tested with useNft() and the adaptations needed.

| NFT minting service                                  | Supported | Specific adaptations done by useNft()                                                  |
| ---------------------------------------------------- | --------- | -------------------------------------------------------------------------------------- |
| [AITO](https://www.thisisaito.xyz/)                  | Yes       |                                                                                        |
| [Async Art](https://async.art/)                      | Yes       |                                                                                        |
| [Clovers](https://clovers.network/)                  | Yes       |                                                                                        |
| [CryptoKitties](https://www.cryptokitties.co/)       | Yes       | Non standard NFT, dedicated mechanism.                                                 |
| [CryptoPunks](https://www.larvalabs.com/cryptopunks) | Yes       | Non standard NFT, dedicated mechanism.                                                 |
| [Cryptovoxels](https://www.cryptovoxels.com/)        | Yes       |                                                                                        |
| [Decentraland ](https://decentraland.org/)           | Yes       | Estate and parcels are fetched from The Graph. Wearables are fetched as standard NFTs. |
| [Foundation](https://foundation.app/)                | Yes       |                                                                                        |
| [JOYWORLD](https://www.joy.world/joys)               | Yes       |                                                                                        |
| [KnownOrigin](https://knownorigin.io/)               | Yes       |                                                                                        |
| [MakersPlace](https://makersplace.com/)              | Yes       | Incorrect JSON format (uses `imageUrl` instead of `image`).                            |
| [Meebits](https://meebits.larvalabs.com/)            | Yes       | CORS restricted, requires a JSON proxy to be set (see `jsonProxy`).                    |
| [MoonCats](https://mooncatrescue.com/)               | Yes       | Non standard NFT, dedicated mechanism.                                                 |
| [Nifty Gateway](https://niftygateway.com/)           | Yes       | Incorrect metadata URL.                                                                |
| [OpenSea](https://opensea.io/)                       | Yes       | Incorrect metadata URL.                                                                |
| [Portion.io](https://app.portion.io/)                | Yes       | Non-standard JSON format.                                                              |
| [Rarible](https://rarible.com/)                      | Yes       |                                                                                        |
| [SuperRare](https://superrare.co/)                   | Yes       |                                                                                        |
| [Uniswap V3](https://uniswap.org/)                   | Yes       |                                                                                        |
| [Zora](https://zora.co/)                             | Yes       |                                                                                        |

## License

[MIT](LICENSE)
