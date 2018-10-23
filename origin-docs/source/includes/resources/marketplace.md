# Marketplace

The marketplace manages listings from creation to sale as well as disputes between users.

## getListings

This will return information about the listing, combining information from IPFS and the blockchain. In the future, fields returned may differ based on the listing's schema.

> Example: getListings

```javascript

> origin.marketplace.getListings({ idsOnly: true })

//returns

['0-34-2-34', '000-234']

> origin.marketplace.getListings({ listingsFor: '0x627306090abab3a6e1400e9345bc60c78a8bef57' })

//returns all listings for '0x627306090abab3a6e1400e9345bc60c78a8bef57'

[{
  id: "99-0023",
  title: "Kettlebell For Sale",
  media: [],
  schemaId: "09398482-2834",
  unitsTotal: 1,
  type: "unit",
  category: "Health and Beauty",
  subCategory: "daily exercise",
  language: "English",
  description: "32kg gorilla kettlebell",
  price: { currency: 'ETH', amount: '0.5' },
  commission: { currency: 'OGN', amount: '1' },
  ipfsHash: "QmWZDcDq4aYGx9XmkPcx4mnKaGW2jCxf5tknrCtbfpJJFf",
  seller: "0x627306090abab3a6e1400e9345bc60c78a8bef57",
  depositManager: '0x02394099fu9dfse0920394u329u4024',
  status: 'active',
  offers: [], //is this supposed to be an array or an object?
  expiry: '1529674159',
  events:[{ id: '20949-345', event: 'ListingCreated' }]
}]
```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**idsOnly** |boolean|optional||
|**listingsFor** | string |optional|user's address|
|**purchasesFor** | string |optional|user's address|

## createListing

When you create a listing, the API will create both the IPFS data and the Listing contract on the blockchain.
When a listing is successfully created, the `createListing` method takes a callback with two arguments:

`confirmationCount` - the number of successfully created listings

`transactionReceipt` - an object with ipfs information about the newly created listing.

> Example: createListing

```javascript

const listing = {
  category: "schema.forSale",
  commission: { amount: "0", currency: "OGN" },
  description: "ojwoifj weofijfoijfewoi qfoiqejqoidjq oidwjqdo",
  language: "en-US",
  listingType: "unit",
  media: [{…}],
  price: { amount: "0.01", currency: "ETH" },
  schemaId: "http://schema.originprotocol.com/listing_v1.0.0",
  subCategory: "schema.forSale.appliances",
  title: "I am a great appliance listing",
  unitsTotal: 1
}

const callback = (confirmationCount, transactionReceipt) => {
  //manage response
}

> origin.marketplace.createListing({ listing }, callback)
```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**listing** |object|required|
|**callback** | function |optional|provides args `confirmationCount` and `transactionReceipt`|

## withdrawListing

Withdrawing a transaction will set the `unitsAvailable` to zero. This will stop any further purchases of that Listing.
When a listing is successfully created, the `withdrawListing` method takes a callback with two arguments:

`confirmationCount` - the number of successfully created listings

`transactionReceipt` - an object with ipfs information about the newly created listing.

> Example: withdrawListing

```javascript

const listingId = "927-832"
const data = {}
const callback = (confirmationCount, transactionReceipt) => {
  //manage response
}

> origin.marketplace.withdrawListing(listingId, data, callback)
```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**listingId** |string|required|id of the listing to be withdrawn|
|**data**|object|optional|default to `{}` if not needed|
|**callback** | function |optional|provides args `confirmationCount` and `transactionReceipt`|

## getListingReviews

Reviews are created by the buyer in the receipt stage and by the seller in the payout stage of the purchase process. A review consists of a required 1-5 rating and an optional reviewText text field.

> Example: withdrawListing

```javascript

const listingId = "927-832"

> origin.marketplace.getListingReviews(listingId)

//returns

[{
  "schemaId": "http://schema.originprotocol.com/review_v1.0.0",
  "rating": 4,
  "text": "Solid Listing"
  "reviewer": "0x29884972398479234792"
]}
```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**listingId** |string|required|id of the listing|

## getNotifications

Each Notification corresponds to a state change of a Purchase. Notifications are currently generated for each of the following purchase stages:

- seller_listing_purchased
- seller_review_received
- buyer_listing_shipped

Notifications do not exist on the blockchain nor are they read from a database. They are derived from the blockchain transaction logs of purchases at the time of the API request. Because of this, there is no central record of a notification's status as "read" or "unread". When a client first interacts with the notifications API, Origin.js will record a timestamp in local storage. All notifications resulting from blockchain events that happen prior to this timestamp will be considered to be "read". This ensures that when the same user interacts with the notifications API from a different client for the first time, they will not receive a large number of "unread" notifications that they have previously read from their original client.

> Example: getNotifications

```javascript

> origin.marketplace.getNotifications()

//returns all notifications for the user

[{
  "id": "2984803-23433",
  "type": "buyer_listing_shipped",
  "status": "unread",
  "event": {},
  "resources": { listingId: "927-832", offerId: "183", listing: { title: "Whirlpool Microwave" } }
]}
```

## setNotification

Since notification objects do not live on the blockchain or in a database, this method only records an update to the client's local storage. It accepts a single parameter, which should be an object containing the id of the notification and a status value of either read or unread. Any other properties included in this object will be ignored.

> Example: setNotification

```javascript

const id = "2984803-23433"
const status = "read"

> origin.marketplace.setNotification({ id, status })

```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**id** |string|required|id of the notification|
|**status** |string|required|`"read"` or `"unread"`|


## getOffer

This will turn a specific offer sent by a buyer for a listing purchase.


> Example: getOffer

```javascript

const offerId = "2403-234"

> origin.marketplace.getOffer(offerId)

//returns the specified offer

{
  id: "2403-234",
  listingId: "999-000-0",
  status: "accepted",
  createdAt: 1539991086,
  buyer: "0x627306090274fwfiou97h0c78a8BEf57",
  events: [...],
  refund: "0",
  schemaId: "http://schema.originprotocol.com/offer_v1.0.0",
  listingType: "unit",
  unitsPurchased: 1,
  totalPrice: { currency: "ETH", amount: "0.033" },
  ipfs: {...}
}
```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**offerId** |string|required|`offer.id`|

## getOffers

This will turn all offers related to a specific listing.


> Example: getOffers

```javascript

const listingId = "9903-75"
const options = {
  for: "0x627306090274fwfiou97h0c78a8BEf57"
}

> origin.marketplace.getOffers(listingId, options)

//returns all offers for the specified listing

[{
  id: "999-000-0-0",
  listingId: "9903-75",
  status: "created",
  createdAt: 1539991086,
  buyer: "0x627306090274fwfiou97h0c78a8BEf57",
  events: [...],
  refund: "0",
  schemaId: "http://schema.originprotocol.com/offer_v1.0.0",
  listingType: "unit",
  unitsPurchased: 1,
  totalPrice: { currency: "ETH", amount: "0.033" },
  ipfs: {...}
},
{
  id: "82838-247-3-3",
  listingId: "9903-75",
  status: "created",
  buyer: "0x938828348ske02heo92394hwf",
  ...
}]
```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**listingId** |string|required|`listing.id`|
|**options** |object|optional|`const options = { for: "0x627..." }`|

## makeOffer

The `makeOffer` method takes a callback with two arguments:

`confirmationCount` - the number of successfully created listings

`transactionReceipt` - an object with ipfs information about the newly created listing.

> Example: makeOffer

```javascript

const listingId = "9903-75"
const offer = {
  id: "82838-247-3-3",
  listingId: "9903-75",
  status: "created",
  buyer: "0x938828348ske02heo92394hwf",
  ...
}
const callback = (confirmationCount, transactionReceipt) => {
  //manage response
}

> origin.marketplace.makeOffer(listingId, offer, callback)

```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**listingId** |string|required|`listing.id`|
|**offer** |object|required|args see Protocol Schemas: Offer Schema|
|**callback** | function |optional|provides args `confirmationCount` and `transactionReceipt`|

## acceptOffer

The `acceptOffer` method takes a callback with two arguments:

`confirmationCount` - the number of successfully created listings

`transactionReceipt` - an object with ipfs information about the newly created listing.

> Example: acceptOffer

```javascript

const offerId = "543-0099"
const data = {}
const callback = (confirmationCount, transactionReceipt) => {
  //manage response
}

> origin.marketplace.acceptOffer(offerId, data, callback)

```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**offerId** |string|required|`offer.id`|
|**data**|object|optional|default to `{}` if not needed|
|**callback** | function |optional|provides args `confirmationCount` and `transactionReceipt`|

## finalizeOffer

The `finalizeOffer` method takes a callback with two arguments:

`confirmationCount` - the number of successfully created listings

`transactionReceipt` - an object with ipfs information about the newly created listing.

> Example: finalizeOffer

```javascript

const offerId = "9903-75"
const buyerReview = {
  rating: 5
  schemaId: "http://schema.originprotocol.com/review_v1.0.0"
  text: "Great response times. Professional."
}
const callback = (confirmationCount, transactionReceipt) => {
  //manage response
}

> origin.marketplace.finalizeOffer(offerId, buyerReview, callback)

```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**offerId** |string|required|`offer.id`|
|**buyerReview** |object|required|args `rating`, `schemaId`, `text`|
|**callback** | function |optional|provides args `confirmationCount` and `transactionReceipt`|

## withdrawOffer

The `withdrawOffer` method takes a callback with two arguments:

`confirmationCount` - the number of successfully created listings

`transactionReceipt` - an object with ipfs information about the newly created listing.

> Example: withdrawOffer

```javascript

const offerId = "543-0099"
const data = {}
const callback = (confirmationCount, transactionReceipt) => {
  //manage response
}

> origin.marketplace.withdrawOffer(offerId, data, callback)

// returns the offer with a OfferWithdrawn event

{
  offerId: "543-0099",
  events: { OfferWithdrawn: {...} }
  ...
}
```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**offerId** |string|required|`offer.id`|
|**data**|object|optional|default to `{}` if not needed|
|**callback** | function |optional|provides args `confirmationCount` and `transactionReceipt`|

## addData

The `addData` method takes a callback with two arguments:

`confirmationCount` - the number of successfully created listings

`transactionReceipt` - an object with ipfs information about the newly created listing.

> Example: addData

```javascript

const offerId = "543-0099"
const listingId = "9900-234"
const sellerReview = {
  rating: 4
  schemaId: "http://schema.originprotocol.com/review_v1.0.0"
  text: ""
}
const callback = (confirmationCount, transactionReceipt) => {
  //manage response
}

> origin.marketplace.addData(listingId, offerId, sellerReview, callback)

// returns a timestamp and transaction receipt

{
  timestamp: 1540221215
  events: { OfferWithdrawn: {...} }
  ...
}
```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**listingId** |string|optional|can be `null`|
|**offerId** |string|optional|can be `null`|
|**sellerReview**|object|optional|default to `{}` if not needed|
|**callback** | function |optional|provides args `confirmationCount` and `transactionReceipt`|

## initiateDispute

The `initiateDispute` method takes a callback with two arguments:

`confirmationCount` - the number of successfully created listings

`transactionReceipt` - an object with ipfs information about the newly created listing.

> Example: initiateDispute

```javascript

const offerId = "543-0099"
const data = {}
const callback = (confirmationCount, transactionReceipt) => {
  //manage response
}

> origin.marketplace.initiateDispute(offerId, data, callback)

// returns a timestamp and transaction receipt

{
  timestamp: 1540221215
  events: { OfferDisputed: {...} }
  ...
}

```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**offerId** |string|required||
|**data**|object|optional|default to `{}` if not needed|
|**callback** | function |optional|provides args `confirmationCount` and `transactionReceipt`|

## resolveDispute

The `resolveDispute` method takes a callback with two arguments:

`confirmationCount` - the number of successfully created listings

`transactionReceipt` - an object with ipfs information about the newly created listing.

> Example: resolveDispute

```javascript

const offerId = "543-0099"
const data = {}
const ruling = 1
const refund = 33000000000000000
const callback = (confirmationCount, transactionReceipt) => {
  //manage response
}

> origin.marketplace.resolveDispute(offerId, data, ruling, refund, callback)

// returns a timestamp and transaction receipt

{
  timestamp: 1540221215
  events: { OfferRuling: {...} }
  ...
}

```

### Arguments:

|Name|Type|Required|Description|
|----|-----|-----|-----|
|**offerId** |string|required||
|**data**|object|required|default to `{}` if not needed|
|**ruling**|number|required|`0` or `1`|
|**refund**|object|required|price in `wei`|
|**callback** | function |optional|provides args `confirmationCount` and `transactionReceipt`|