# SRC-721 NFT Collection Specification

## Introduction
Stamps are very expensive and there needs to be an inexpensive way for users to mint higher resolution composable NFTs (like 10k pfp projects).  SRC-721 specifies how to store all the art for a given collection as layers using the STAMPS protocol and have the user mint a small JSON file referencing data that was already stored on chain, in order to create an NFT composed from the aformentioned layers. By storing the layers individually it is possible to significantly reduce file size by using techniques like indexed color pallets for each layer.  By stacking the images up it is possible to create a high quality visually appealing final product.  By ensuring the user has to store less than 100 bytes on chain, the mint cost for this higher resolution NFT is significantly reduced.

Since the Stamps protocol is under active development and changes will occur causing reindexing of Stamp ID's this specification uses counterparty asset ID's to ensure that inevitable changes to the Stamps protocol do not impact asset references of deployed SRC-721 collections.  

## Operations
SRC-721 transactions must conform to these **required** fields or the transaction will not be considered a valid SRC-721 transaction. Fields labeled optional can be omitted when not used.

### DEPLOY
```
{
        "p": "src-721",
        "v": "1",
        "op": "deploy",
        "name": "Collection Name",      // The display name of the collection
        "symbol": "SYM",                // the symbol for the collection
        "description": "Description",
        "max": "2500",                    // maximum number of mints
        "lim": "1",                       // limit per mint [optional, default=1]
        "num": "1",                       // the maximum number of stamp with the same traits [optional, default=1]
        "wl": "1",                        // public(0)or whitelist(1) mint phase [optional, default=0]
        "mode": "1",                     //  traits allocation mode, random allocation(0) or authorized allocation(1) [optional, default=0]
        "operator": "1ABC...321",         // bitcoin address of the operator for whitelist mint [optional]
        "price": "200000",               // mint fee in satoshis [optional, default=0]
        "recipient": "1ABC...321",       // recipient address of mint fee. must exist and valid address if the price is not 0. 
        "type": "data:image/png;base64",// mime type of the images used in traits t0-tx
        "image-rendering":"pixelated",  // css property to ensure images are displayed properly [optional]
        "viewbox": "0 0 160 160",       // viewbox to properly see  traits t0-tx
        "icon": "A16308540544056654000",// CP asset for a collection icon [optional]
        // All t0-tx are optional if the reveal op is planned to be used
        "t0": ["A12430899936789156000", "A9676658320305385000"],    // up to x layers of stamp traits (references by CP asset#) containing
        "t1": ["A17140023175661332000", "A6689685157378600000"],    // transparency can be stacked on top of eachother to form a final image
        ...
        "tx": ["A12240402677681132000", "A4332886198473102000"]
}
```

### REVEAL
```
{
    "p": "src-721",
    "v": "1",
    "op": "reveal",
    "symbol": "SYM",    // symbol [optional]
    "c":"A123456789",   // a pointer to the deploy collection json cp asset
    "sig": "a1b2...e8d9",   // signed hash of data object containing references to traits [optional] only needed if the sender is not the collection owner
    "data":{
       "s": ["seed0", "seed1", ... "seedx"]                        // seed used for the deterministic generation of traits [optional]
        "t0": ["A12430899936789156000", "A9676658320305385000"],    // up to x layers of stamp traits (references by CP asset#) containing
        "c0": ["1000", "10000"],                                    // coefficients used to select t0 traits per tokenId [optional]    
        "t1": ["A17140023175661332000", "A6689685157378600000"],    // transparency can be stacked on top of eachother to form a final image
        "c1": ["4000", "10000"],                                    // coefficients used to select t1 traits per tokenId [optional]     
        ...
        "tx": ["A12240402677681132000", "A4332886198473102000"],
        "cx": ["7000", "10000"],                                    // coefficients used to select tx traits per tokenId [optional]     
    }
}
```

### MINT
```
{
    "p": "src-721",
    "op": "mint",
    "symbol": "SYM",      // symbol [optional]
    "c":"A123456789",     // a pointer to the deploy collection json cp asset
    "num": "0"            // The ordinal number of stamps with the same traits,  [optional, default=0]
    "amt": "1",           // amount to mint, value range is 1 to NUM [optional, default=1]
    "sig": "1234...abcd", // used for a permissioned mint signed(sha256(num+JSON.stingify(ts)+userAddress)) [optional]
                        // MAY WANT TO USE A truncate the signed hash to minimize mint op size
    "ts":[0,1,...,y]    // an array with x length wherein each item
                        // represents the index of the trait to use
                        // from the deploy mechanism
}
```

### UPDATE - updates mutable properties of deploy
```
{
"p": "src-721",
"op": "update",
"operator": "1ABC...321", // the bitcoin address of the new operator [optional]
"price":"10000", // the price for the mint in satoshis [optional]
"recipient": "1ABC...321", // mint fee recipient address [optional]
"wl": "1",           // public(0) or whitelist(1) mint phase [optional]
"mode": "1",         //  traits allocation mode, random allocation(0) or authorized allocation(1) [optional]
}
```

### MINT a single item (not part of a collection)
It may be desireable to mint a single NFT (that is not part of a collection) based on various on chain assets.  This can be done by omitting the symbol field and using counterparty asset IDs instead of index's
```
{
    "p": "src-721",
    "op": "mint",
    "ts":["A12430899936789156000", .., "A17140023175661332000"]    
                        // an array with x length wherein each item
                        // represents the CP asset ID 
}
```

### TRANSFER and USE

SRC-721 transactions are valid counterparty assets and can be use as such.


## Design Philosophy

### Target

1. Uniform Indexer: Develop a comprehensive, standardized SRC721 indexing algorithm.

2. Flexible Mint Mode: Offer an array of versatile minting modes that are both combinable and interchangeable.

3. Real-time Validation: Ascertain the validity of a mint transaction immediately upon its confirmation.

### Roles

1. Owner: The address that holds the collection deployment stamp. The owner changes as the stamp is transferred.

2. Operator: The address executes delegated mint through providing signature or directly mint. The primary function is for the owner to delegate the mint service to handle project issuance. 

### Authority Operations

1. Change Parameters: The owner has the ability to initiate a protocol operation to modify the operator, mint price, recipient, mint phase, and mint mode.

2. Generate Reveal Signature: The owner signs the hash(traits data objects) to obtain a signature. If the first input of the transaction is the owner, no signature is required.

3. Generate Mint Signature: The operator signs the sha256(num+JSON.stingify(ts)+userAddress) to obtain a signature. sha256 should be a byte array not hex, to reduce footprint

Note: ****May want to truncate the mint signature to a fraction of the total size to reduce on-chain footprint

### Sale Mode

1. Public Sale: Any address can participate in minting.

2. Whitelist Sale: Only authorized addresses are allowed to mint.

### Traits Allocation Mode

There are two main mint modes for traits allocation,
1. Authorized allocation: A mint transaction, which is either initiated by the operator or containing the operator's signature, incorporates the relevant traits. ts must be present, 3 or more UTXOs

2. Random allocation: the reveal function outlines the traits distribution. ts is ignoted, 2 or more UTXOs

### Additional Notes

1. Token ID: If a Token ID is required, it is assigned in the order of valid minting. However, if a Token ID is added in the mint JSON, it would be difficult to effectively implement the combination and switching of mint modes.

2. NUM: The maximum number of stamp with the same traits. In order to allow for the existence of Images with the same traits within a collection and prevent multiple mints with the same signature from a single address, we have introduced a variable called "num" to represent the maximum quantity of Images with the same traits. Additionally, the signature data needs to include an indication of which numbered Image with the same traits it corresponds to. If the value of "num" in the deploy JSON is N, then the range of "num" in the signature data is [0, N-1].

## Mint Modes Combination

Based on the Sale Mode and Traits Allocation Mode, 4 minting modes can be combined, and these 4 modes can be switched interchangeably.

### Whitelist Sale with Authorized Traits Allocation
```
{
    "p": "src-721",
    "op": "mint",
    "c": "A123456789",   
    "num": "0"           // [optional, default=0]
    "amt": "1",         // [optional, default=1]
    "ts": [0,1,...,y],
    "sig": "1234...abcd" // signed(sha256(num+JSON.stingify(ts)+userAddress)). The signature field is not required if the sender is the operator. 
} 
```   
MultiSig UTXO Amount: at least 3

### Whitelist Sale with Random Traits Allocation
```
{
    "p": "src-721",
    "op": "mint",
    "c": "A123456789",   
    "num": "0"          // [optional, default=0]
    "amt": "1",         // [optional, default=1]
    "sig": "1234...abcd" // signed(sha256(userAddress)). The signature field is not required if the sender is the operator
}  
```  
MultiSig UTXO Amount: at least 2


### Public Sale with Authorized Traits Allocation
```
{
    "p": "src-721",
    "op": "mint",
    "c": "A123456789",   
    "num": "0"           // [optional, default=0]
    "amt": "1",         // [optional, default=1]
    "ts": [0,1,...,y],
    "sig": "1234...abcd" // signed(sha256(num+JSON.stingify(ts)+userAddress)). The signature field is not required if the sender is the operator
} 
```   
MultiSig UTXO Amount: at least 3

### Public Sale with Random Traits Allocation
```
{
    "p": "src-721",
    "op": "mint",
    "c": "A123456789",
    "num": "0"           //[optional, default=0]
    "amt": "1"          // [optional, default=1]
} 
```   
MultiSig UTXO Amount: at least 2

## Indexer

### How indexer validates a valid mint?

1. The "c" field must point to a valid SRC721 Collection.

2. The collection is not fully minted yet.

3. If the price is not 0, the mint transaction should include an output whose amount is equal to or greater than the price to the recipient.

4. For images with the same traits, the number can't exceed NUM. Count(sha256(JSON.stringify(ts))) <= NUM, except in the case of Random Traits Allocation mode.

5. If mode=1 or wl=1(Whitelist Sale or Authorized Traits Allocation), the signature (sig) in the mint json must be unique.

6. If mode=1(Authorized Traits Allocation), then the sender needs to either be the operator or the mint json has a correctly signature signed(sha256(num+JSON.stringify(ts)+userAddress)).

7. If mode=0 and wl=1(Whitelist Sale with Random Traits Allocation), then the sender needs to either be the operator or the mint json has a correctly signature signed(sha256(userAddress)).


## SRC-721 Token Requirements

1. Tokens must be 1-5 characters in length.

2. Allowed characters:
   a. Any word character (alphanumeric characters and underscores)
   b. Special characters: ~!@#$%^&*()_+=<>?
   c. Most printable emojis in U+1F300 to U+1F5FF

3. Disallowed characters:
   a. Non-printable Unicode characters
   b. Quotation marks: " ` '
   c. Special characters not present in 2c

4. Only numeric values are allowed in the "max", "lim" fields

5. Other Qualifications:
    - CP Asset must be locked, and multisig dust assigned to qualified burn address For more details on "KeyBurn" see: https://github.com/mikeinspace/stamps/blob/main/Key-Burn.md
    - CP Asset for deploy must be value 1, nondivisible
    - CP Asset value for mint must be less than or equal to lim
    - not case sensitive DOGE=doge
    - max lim amount: uint64_max 18,446,744,073,709,551,615 (**commaas not allowed**, here for readability only)
    - json strings are not order sensitive
    - json strings are not case sensitive


**INVALID** tokens will not be created in the Bitcoin Stamps Protocol index or API, and the transaction will not be considered a valid SRC-721 transaction. Any further modifications to the standard must be designed around backwards compatibility.


## Allowed Unicode Chars

Emoji_Presentation: This property includes all characters that are defined as emojis and have a distinct emoji-style appearance. These characters are intended to be displayed as colorful pictographs, rather than black-and-white text symbols. Examples include face emojis (😀, 😂, 😊), objects (🚗, 🌍, 🍕), and symbols (❤️, 🚫, ⏰).

Emoji_Modifier_Base: This property consists of characters that can be modified by emoji modifiers, such as skin tone modifiers. These characters usually represent human-like figures (e.g., 👩, 👨, 🤳) and can be combined with emoji modifiers to represent variations in skin tone or other attributes.

Emoji_Modifier: This property contains characters that can be used to modify the appearance of other emojis, particularly the ones classified as Emoji_Modifier_Base. The most common example is the skin tone modifiers (🏻, 🏼, 🏽, 🏾, 🏿) that can be applied to human-like emojis to represent different skin tones.


## Excluded Unicode Chars

These chars are excluded from the allowed chars list because they are not printable, and are not allowed in any field. Tokens with these chars will not be created in Bitcoin Stamps Protocol index or API, and the transaction will not be considered a valid SRC-721 transaction.

Emoji_Component: Characters that are used to create more complex emojis, such as skin tone modifiers and hair components. These characters are not emojis on their own but can be used with other emojis.

Extended_Pictographic: This includes additional pictographic characters not covered by Emoji_Presentation but can still be considered emojis.
