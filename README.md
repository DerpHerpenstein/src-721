# SRC-721 Composable NFT Specification

Stamps are very expensive and there needs to be an inexpensive way for users to mint good resolution composable NFTs (like 10k pfp projects).  SRC-721 specifies how to store all the art for a given collection as layers using the STAMPS protocol and have the user mint a small JSON file referencing data that was already stored on chain, in order to create an NFT composed from the aformentioned layers. By storing the layers individually it is possible to significantly reduce file size by using techniques like indexed color pallets for each layer.  By stacking the images up it is possible to create a high quality visually appealing final product.

## Introduction

SRC-721 transactions must conform to these **required** fields or the transaction will not be considered a valid SRC-721 transaction. Fields labeled optional can be omitted when not used.

### DEPLOY
```
{
        "p": "src-721",
        "v": 1,
        "op": "deploy",
        "name": "Collection Name",      // The display name of the collection
        "symbol": "SYM",                // the symbol for the collection
        "description": "Description",
        "unique": true,                 // determines if a set of traits must be unique to be valid
        "root": "a1b2...e8d9"           // merkle root for a permissioned mint [optional]
        "type": "data:image/png;base64",// mime type of the images used in traits t0-tx
        "image-rendering":"pixelated",  // css property to ensure images are displayed properly [optional]
        "viewbox": "0 0 160 160",       // viewbox to properly see  traits t0-tx
        "max": 2500,                    // maximum number of mints
        "lim": 1,                       // limit per mint
        "icon": "A16308540544056654000",// CP asset for a collection icon 
        // All t0-tx are optional if the reveal op is planned to be used
        "pubkey": "a1b2...e8d9"         // pubkey for future ops such as reveal [optional]
        "t0": ["A12430899936789156000", "A9676658320305385000"],    // up to x layers of stamp traits (references by CP asset#) containing
        "t1": ["A17140023175661332000", "A6689685157378600000"],    // transparency can be stacked on top of eachother to form a final image
        ...
        "tx": ["A12240402677681132000", "A4332886198473102000"]
}
```

### Reveal
```
{
    "p": "src-721",
    "v": 1,
    "op": "reveal",
    "symbol": "SYM",
    "sig": "a1b2...e8d9",   // signed hash of data object containing references to traits
    "data":{
        "t0": ["A12430899936789156000", "A9676658320305385000"],    // up to x layers of stamp traits (references by CP asset#) containing
        "t1": ["A17140023175661332000", "A6689685157378600000"],    // transparency can be stacked on top of eachother to form a final image
        ...
        "tx": ["A12240402677681132000", "A4332886198473102000"]
    }
}
```


### MINT
```
{
    "p": "src-721",
    "op": "mint",
    "symbol": "SYM",
    "proof": "",        // merkle proof, used for a permissioned mint [optional]
    "ts":[0,1,...,y]    // an array with x length wherein each item
                        // represents the index of the trait to use
                        // from the deploy mechanism
}
```

### MINT a single item (not part of a collection)
```
{
    "p": "src-721",
    "op": "mint",
    "symbol": "",
    "ts":["A12430899936789156000", .., "A17140023175661332000"]    
                        // an array with x length wherein each item
                        // represents the CP asset ID 
}
```
### TRANSFER and USE

SRC-721 transactions are valid counterparty assets and can be use as such.

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
    - CP Asset for deploy muse be value 1, nondivisible
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
