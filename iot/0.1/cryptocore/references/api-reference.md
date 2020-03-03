# CryptoCore API

**The CryptoCore exposes an UART API, which you can call over a UART connection. For example, you could send API calls through a device that's connected to the CryptoCore over USB. This API reference also shows you how to use the REST API server, that we started in [this guide](../introduction/get-started.md).**

<!--## REST API header

All REST API requests must include an HTTP header.

| **Header**       | **Value** | **Required or Optional** |
|:---------------|:--------|:--------|
| X-IOTA-API-Version | 1 | Required |
| Content-Type | application/json | Required |
-->

## setFlags

Sets the given flags.

The following flags are available:

- **keepSeedInRAM:** Caches the seed in RAM after reading it from the secure element. (Caching the seed saves 1-2 seconds for each request of the seed.) This flag is set to `true` by default.

- **debugRS232Output:** Enables debugging output on the SWD line or RS232

:::info:
Before you can set the `keepSeedInRAM` flag, you must initalize the secure
element by calling the [`initSecureElement`](#initsecureelement) command.
:::

### Parameters

|**Parameter**    |**Type**      |                   **Description**|
|:----------- |:-------- |:--------------------------------------------------------|
| `flags`     |object   |The name/value pairs of the flags that you want to set|

### Example request

```json
{
    "command":"setFlags",
    "flags":{
        "keepSeedInRAM":true
    }
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "command":"setFlags",
    "duration": 250
}
```
---
### 400
```json
{
    "code": 400,
    "command":"setFlags",
    "error": "Error message"
}
```
--------------------


## testHardwareAcceleration

Tests the hardware acceleration on the ICCFPGA module.

The following functions are tested and compared with the results of
unaccelerated functions:

- Type conversions (bytes/trytes and bytes/trits)

- Hashing (proof of work, Curl, Keccak384, and Troika)

### Example request

```json
{
    "command":"testHardwareAcceleration"
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "command":"testHardwareAcceleration"
    "bytesToTritsSingle":"pass",
    "tritsToBytesSingle":"pass",
    "pow":"pass",
    "keccak384":"pass",
    "bytesToTritsRandomRepeated":"pass",
    "tritsToBytesRandomRepeated":"pass",
    "trytesToBigintRandomRepeated":"pass",
    "bigintToTrytesRandomRepeated":"pass",
    "troika":"pass",
    "curl":"pass",
    "duration":1867
}
```
---
### 400

```json
{
    "code": 400,
    "command":"testHardwareAcceleration",
    "error": "Error message"
}
```
--------------------

## generateRandomSeed

Generates a random seed and stores it in one of eight available memory
addresses in the secure element.

:::info:
Before you can call this command, you must initalize the secure
element by calling the [`initSecureElement`](#initsecureelement) command.
:::

### Parameters

|**Parameter** |   **Type**  |              **Description**|
|:----------- |:---------|:-------------------------
|`slot`   |   integer |  An integer between 0 and 7, which specifies the memory address in which to save the seed|

### Example request

```json
{
    "command":"generateRandomSeed",
    "slot": 0
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "command":"generateRandomSeed",
    "duration":1800
}
```
---
### 400

```json
{
    "code": 400,
    "command":"generateRandomSeed",
    "error": "Error message"
}
```
--------------------

## generateAddress

Generates addresses for the seed in given slot of the secure element.

:::info:
Before you can call this command, you must initalize the secure
element by calling the [`initSecureElement`](#initsecureelement) command.
:::

### Parameters

|**Parameter**  |   **Type**  |     **Description**|
|:------------ |:---------|-------------------------|
`slot`       |integer   |The memory address of the seed from which you want to derive the address
`firstIndex`  | integer    |      The address index from which to start generating addresses
`number` |    integer  |    The number of addresses to generate, starting from the first index
`security`   | integer    |     The security level of the address that you want to generate

### Example request

```json
{
    "command":"generateAddress",
    "slot": 0,
    "firstIndex": 0,
    "number": 10,
    "security": 2
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "command":"generateAddress",
    "trytes": ["....","...."],
    "duration": 1800
}
```
---
### 400

```json
{
    "code": 400,
    "command":"generateAddress",
    "error": "Error message"
}
```
--------------------

## version

Gets the current API version

### Example request

```json
{
    "command":"version"
}
```

### Response examples

--------------------
### 200
```json
{
    "version":"0.11rv",
    "command":"version",
    "duration":0,
    "code":200
}
```
---
### 400

```json
{
    "code": 400,
    "command":"version",
    "error": "Error message"
}
```
--------------------

## attachToTangle

Chains the given transaction trytes into a bundle and does proof of work on all of them, using the given minimum weight magnitude.

This command can do proof of work for a bundle that contains up to eight
transactions.

### Parameters

|**Parameter** |             **Type**            |**Description**|
|:-------------------- |:------------------ |:---------------------------|
|`trunkTransaction`  |       string   |       Trunk transaction hash to use to attach the bundle to the Tangle|
|`branchTransaction`    |     string   |       Branch transaction hash to use to attach the bundle to the Tangle|
|`minWeightMagnitude`   |    integer      |        The minimum weight magnitude to use during proof of work|
|`timestamp`       |     integer  |      A Unix epoch timestamp in milliseconds to add to the transaction's 'attachmentTimestamp' field|
|`trytes`      |   array of strings    |    Transaction trytes of up to eight transactions in a bundle|

### Example request

```json
{
    "command":"attachToTangle",
    "trunkTransaction": "...",
    "branchTransaction": "...",
    "minWeightMagnitude": 14,
    "timestamp": 1552571227826,
    "trytes": ["....", "...."]
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "command":"attachToTangle",
    "trytes": ["....","....",...],
    "duration": 1800
}
```
---
### 400

```json
{
    "code": 400,
    "command":"attachToTangle",
    "error": "Error message"
}
```
--------------------

## doPow

Does proof of work on a bundle's transaction trytes.

This command can do proof of work for up to eight transactions at once.

### Parameters

|**Parameter**              |**Type**    |                           **Description**|
|:-------------------- |:------------------ |:----------------------------------------------------------
|minWeightMagnitude   |    integer      |  The minimum weight magnitude to use during proof of work
|trytes     |    array of strings     |       Transaction trytes

### Example request

```json
{
    "command":"doPow",
    "minWeightMagnitude": 14,
    "trytes": ["....", "....", ...]
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "command":"doPow",
    "trytes": ["....","....",...],
    "duration": 1800
}
```
---
### 400

```json
{
    "code": 400,
    "command":"doPow",
    "error": "Error message"
}
```
--------------------

## signTransaction

Signs a single input transaction, using the seed in the given slot of the secure element.

:::info:
Before you can call this command, you must initalize the secure
element by calling the [`initSecureElement`](#initsecureelement) command, then do the following calculation and add the result to the `auth` parameter:

```
keccak384(slot+addressIndex+bundleHash+apiKey)
```
:::

### Parameters

|**Parameter**|      **Type** |                                  **Description**
|:--------------- |:--------- |:--------------------------------------------------------------------------
|`slot`    |    string        |      The memory address of the seed that owns the address
|`addressIndex`  |  string       |           The index of the input transaction's address
|`bundleHash`   |  integer     |        The bundle hash in the transaction's `bundle` field
|`securityLevel` |  integer   |         The security level of the input transaction's address
|`auth`       | string  |  The Keccak384 hash of the `slot`, `addressIndex`, `bundleHash`, and the API key

### Example Request

```json
{
    "command":"signTransaction",
    "slot": 0,
    "addressIndex": 0,
    "bundleHash": "...",
    "securityLevel": 2,
    "auth": "..."
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "command":"signTransaction",
    "trytes": ["....","....",...],
    "duration": 1800
}
```
---
### 400

```json
{
    "code": 400,
    "command":"signTransaction",
    "error": "Error message"
}
```
--------------------

## jsonDataTX

Creates a zero-value transaction that contains the given JSON data in
the `signatureMessageFragment` field.

This command returns the transaction trytes (including proof of work) of
the zero-value transaction.

These trytes are ready for sending to a node.

### Parameters

|**Parameter**|      **Type** |                                  **Description**
|:--------------- |:--------- |:--------------------------------------------------------------------------
|`trunkTransaction`  |       string   |       Trunk transaction hash to add to the transaction's `trunkTransaction` field|
|`branchTransaction`    |     string   |       Branch transaction hash to add to the transaction's `branchTransaction` field|
|`minWeightMagnitude`   |    integer      |        The minimum weight magnitude to use during proof of work|
|`tag`       |     string  |      27 trytes to add to the transaction's `tag` field|
|`address`      |   string |    81 tryte address to add to the transaction's `address` field|
|`timestamp`      |   integer    |    A Unix epoch timestamp in **milliseconds** to add to the transaction's `attachmentTimestamp` field|
|`data`      |   JSON    |    JSON data to add to the transaction's `signatureMessageFragment` field|

:::warning:
Nodes check that the `attachmentTimestamp` is no older than their oldest recorded milestone. Therefore, if the `timestamp` field is not valid, the nodes will return an `invalid trytes` error.
:::

### Example Request

```json
{
    "command": "jsonDataTX",
    "trunkTransaction":"...",
    "branchTransaction":"...",
    "minWeightMagnitude": 14,
    "tag":"...",
    "address":"...",
    "timestamp":1566907523000,
    "data":{"test":"myFirstCryptoCoreTransaction"}
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "command": "jsonDataTX",
    "hash": "...9999",
    "trytes": ["....","....",...],
    "duration": 1800
}
```
---
### 400

```json
{
    "code": 400,
    "command": "jsonDataTX",
    "error": "Error message"
}
```
--------------------

## initSecureElement

Initializes the secure element so that the API can access it.

Before the RISC-V firmware shares the secret key with the
secure element, you must call this command to prove that you know the
key.

This command is a security measure that prevents attackers from removing the secure element, replacing it with another and reading the secret key
from the RISC-V firmware.

This command needs to be called only once.

### Parameters

|**Parameter**|      **Type** |                                  **Description**
|:--------------- |:--------- |:--------------------------------------------------------------------------
|`key`  |       string   |       The secret key |

### Example Request

```json
{
    "command": "initSecureElement",
    "key": "3780e63d4968ade5d822c013fcc323845d1b569fe705b60006feec145a0db1e3"
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "command": "initSecureElement",
    "duration": 1800
}
```
---
### 400

```json
{
    "code": 400,
    "command": "initSecureElement",
    "error": "Error message"
}
```
--------------------

## readFlashPage

Reads 4 kB pages from the QSPI flash memory. The output data is in base64 format.

### Example request

```json
{
    "command": "readFlashPage",
    "page": 0    
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "page":0,
    "command": "readFlashPage",
    "duration": 12,
    "data":"..base64.." 
}
```
---
### 400

```json
{
    "code": 400,
    "command": "readFlashPage",
    "error": "Error message"
}
```
--------------------

## writeFlashPage

Writes one or more 4 kB pages to the QSPI flash memory. This way, the soft CPU can update the entire system by writing new bitstreams into flash.

:::info:
Before you can call this command, do the following
calculation and add the result to the `auth` parameter:

```
base64(keccak384(page+data+apiKey))
```
:::

### Parameters

|**Parameter**    |**Type** |   **Description**
|:----------- |:--------- |:-----------------------------------------------------------
|`page`      |integer|  page number in QSPI flash. Valid values are between 0 and 4095.
|`data`     | string  | 4 kB data in base64 encoding
|`auth`  |    string |  Checksum and authentication
                        

```json
{
    "command": "writeFlashPage",
    "page": 0    
    "data": "..b64..",
    "auth": "....",
}
```

### Response examples

--------------------
### 200
```json
{
    "code": 200,
    "command": "writeFlashPage",
    "duration": 100
}
```
---
### 400

```json
{
    "code": 400,
    "command": "writeFlashPage",
    "error": "Error message"
}
```
--------------------