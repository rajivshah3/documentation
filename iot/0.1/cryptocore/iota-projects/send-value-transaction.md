# Sign a bundle with the CryptoCore

**In this guide, you use the CryptoCore UART API to sign a bundle, do proof of work, and send attach the transactions to the Tangle.**

This guide walks you through the process of writing the following scripts:

- **`create-unsigned-bundle.js`:** This script creates an unsigned bundle
- **`generate-auth.js`:** This script generates a hexadecimal-encoded hash to authenticate the `signBundleHash` command on the CryptoCore
- **`send_value_tx.sh`:** This scripts uses the CryptoCore to sign the bundle and send attach the transactions to th Tangle
- **`add-signature-to-bundle.js`:** This script adds the signature to the bundle

:::info:
These code samples are also hosted on [GitHub](https://github.com/JakeSCahill/cryptocore-scripts).
:::

:::warning:
For your convenience, all sample code uses the default secret key and the default API key.

If you want to use the CryptoCore in a production environment, you should follow the instructions in the 'Securing the FPGA' section of the [manual](https://gitlab.com/iccfpga-rv/iccfpga-manual/-/blob/master/iccfpga.pdf).
:::

## Prerequisites

To complete this guide, you need to have completed the [CryptoCore getting started guide](../introduction/get-started.md).

You also need Node.js installed on the Raspberry Pi. See [this article](https://github.com/nodesource/distributions/blob/master/README.md#debinstall) and follow the Ubuntu installation instructions.

## Packages

To complete this guide, you need to install the following Node.js packages:

--------------------
### npm
```bash
npm install @iota/core @iota/transaction-converter @iota/transaction @iota/bundle serialport cryptojs
```
---
### Yarn
```bash
yarn add @iota/core @iota/transaction-converter @iota/transaction @iota/bundle serialport cryptojs
```
--------------------

You will also need to install the following Linux package:

```bash
sudo apt-get install jq
```

## Step 1. Create an unsigned bundle

In this step, you write a script that uses the Javascript client library to create an unsigned bundle.

1. Create a file called `create-unsigned-bundle.js`

    ```bash
    sudo nano create-unsigned-bundle.js
    ```

2. Get the arguments that are passed to the script

    ```js
    // Get the first argument that was passed to this script
    // This should be a minimum weight magnitude (14 or 9)
    const network = process.argv[2];

    // Get the second argument that was passed to this script
    // This should be an 81 tryte address from which to withdraw IOTA tokens
    const inputAddress = process.argv[3];
    const inputAddressTrits = Converter.trytesToTrits(inputAddress);

    // Get the third argument that was passed to this script
    // This should be an 81 tryte address in which to deposit the IOTA tokens from the input address
    const outputAddressTrits = Converter.trytesToTrits(process.argv[4]);

    // Get the fourth argument that was passed to this script
    // This should be a security level between 1 and 3
    const securityLevel = parseInt(process.argv[5]);

2. Use the first argument to connect to a node on either the Devnet or the Mainnet

    ```js
    // Define a node for each IOTA network
    const nodes = {
            devnet: 'https://nodes.devnet.thetangle.org:443',
            mainnet: `https://nodes.iota.org:443`
    }

    // Connect to the correct IOTA network, depending on the user's
    // selection in the main script
    if (network === 14) {
        iota = Iota.composeAPI({
            provider: nodes.mainnet
            });
    } else {
        iota = Iota.composeAPI({
            provider: nodes.devnet
            });
    }

3. Define a function to create the unsigned bundle

    ```js
    async function createUnsignedBundle({ outputAddress, inputAddress, securityLevel, value }) {

    let bundle = new Int8Array();

    const issuanceTimestamp = Converter.valueToTrits(Math.floor(Date.now() / 1000));

    // Add a transaction to the bundle to deposit the total balance of the input address to the output address
    bundle = Bundle.addEntry(bundle, {
    address: outputAddress,
    value: Converter.valueToTrits(value),
    issuanceTimestamp
    });

    // For every security level, create a new zero-value transaction to which you can later add the extra signature fragments
    for (let i = 0; i < securityLevel; i++) {
    bundle = Bundle.addEntry(bundle, {
        address: inputAddress,
        value: Converter.valueToTrits(i == 0 ? -value : 0),
        issuanceTimestamp
    });
    }

    // Calculate the bundle hash
    const result = await Bundle.finalizeBundle(bundle);

    // Save the bundle array to a binary file
    // that you can later read to add the signature
    fs.writeFileSync('bundle', result, (error) => {
    if(!error) {
        console.log('Bundle details saved to file');
    } else{
        console.log(`Error writing file: ${error}`);
    }});

    // Get the bundle hash and print it to the console
    const bundleHash = Converter.tritsToTrytes(Transaction.bundle(result));
    console.log(bundleHash);
    }
    ```

4. Make sure that the input address has a non-zero balance of IOTA tokens before creating the unsigned bundle

    ```js
    // Set the default transaction value
    let value = 0;

    // Check the balance of the input address
    iota.getBalances([inputAddress], 100)
    .then(({ balances }) => {
        
        if (balances[0] === 0) {
        console.log("The CryptoCore address has no IOTA tokens");
        console.log("Send IOTA tokens to the address to continue");
        return;
        } else {
        const parameters = {
        outputAddress: outputAddressTrits,
        inputAddress: inputAddressTrits,
        securityLevel: securityLevel,
        value: balances[0]
        }

        // Create an unsigned bundle and save it to a binary file
        createUnsignedBundle(parameters);
        }
    })
    .catch(error => {
        console.log(error);
    });
    ```

## Step 2. Authenticate the `signBundleHash` command

In this step, you write a script that generates the `auth` parameter for the [`signBundleHash`](../references/api-reference.md#signbundlehash) command.

1. Install the packages

    ```bash
    cd ~/cryptocore-scripts/node-scripts
    sudo npm i crypto-js
    ```

2. Create a file called `generate-auth.js`

    ```bash
    sudo nano generate-auth.js
    ```

3. Get the arguments that are passed to the script

    ```js
    // Get the first argument that was passed to this script
    // This should be a slot number between 0 and 7
    const slot = process.argv[2];

    // Get the second argument that was passed to this script
    // This should be a keyIndex
    const keyIndex = process.argv[3];

    // Get the third argument that was passed to this script
    // This should be an unsigned bundle hash
    const bundleHash = process.argv[4];

4. Define a function to convert the integer arguments to bytes

    ```js
    // Converts an integer to bytes in the little endian format
    function toBytesInt32 (number) {
        return new Uint8Array([(number & 0x000000ff), (number & 0x0000ff00) >> 8, (number & 0x00ff0000) >> 16, (number & 0xff000000) >> 24]);
    }
    ```

    :::info:
    The CryptoJS library that will generate the hash requires all arguments to be converted to a Word Array from bytes.
    :::
    
5. Define a function to convert bytes to a Word Array

    ```js
    // Converts bytes to a CryptoJS Word Array
    function byteArrayToWordArray(byteArray) {
        var wordArray = [];
        for (i = 0; i < byteArray.length; i++) {
                wordArray[(i / 4) | 0] |= byteArray[i] << (24 - 8 * i);
        }

        return CryptoJS.lib.WordArray.create(wordArray, byteArray.length);
    }
    ```

6. Convert the arguments to bytes

    ```js
    let bundleHashChars = new Buffer.from(bundleHash, "ascii");
    let bundleHashBytes = Uint8Array.from(bundleHashChars);
    
    var buffer = [];
    
    // Add the arguments as bytes to the buffer array
    buffer.push(toBytesInt32(slot));
    buffer.push(toBytesInt32(keyIndex));
    buffer.push(bundleHashBytes);
    buffer.push(apiKey);
    ```

7. Convert the arguments to a Word Array, then use the CryptoJS library to hash the arguments and print the hexadecimal-encoded string to the console

    ```js
    k = CryptoJS.algo.SHA3.create()
    k.init({
    outputLength: BIT_HASH_LENGTH,
    })
    
    // Convert the bytes in the buffer array to a Word Array
    for (let b of buffer) {
            k.update(byteArrayToWordArray(b))
    }

    hash = k.finalize();

    // Print the hexadecimal-encoded hash
    console.log(hash.toString(CryptoJS.enc.Hex))
    ```

    This code uses the SHA-3 method of the [CryptoJS library](https://cryptojs.gitbook.io/docs/) to hash the arguments and convert the hash to a hexadecimal-encoded string.

## Step 3. Sign the bundle hash

In this step, you write a bash script that signs the bundle hash.

1. Create a new file called `send_value_tx.sh`

    ```bash
    sudo nano send_value_tx.sh
    ```

2. Ask whether the user wants to create a transaction for the Devnet or the Mainnet, and store the answer in the `MWM` variable

    ```bash
    #!/bin/bash

    read -p "Are you sending this transaction to the Devnet or the Mainnet? " MWM

    if [[ $MWM =~ ^[mM] ]]
    then
        echo "Setting minimum weight magnitude to 14."
        MWM=14
    else
        echo "Setting minimum weight magnitude to 9."
        MWM=9
    fi 
    ```

3. Ask the user for the slot number of the seed that the script should use to generate an input address and sign the bundle hash

    ```bash
    read -p "Please enter the slot number for a seed in the CryptoCore secure memory " slot

    while [[ ! $slot =~ ^[0-7]{1}$ ]]; do
            read -p "Please enter a valid slot number between 0 and 7" slot
    done
    ```

4. Set the default index and [security level](root://getting-started/0.1/clients/security-levels.md) to use to generate the input address

    ```bash
    # This demo always uses keyIndex 0
    keyIndex=0

    # This demo always uses security level 2
    securityLevel=2
    ```

    :::info:
    Input addresses are those that are used in [input transactions](root://getting-started/0.1/transactions/transactions.md#input-transactions), which withdraw IOTA tokens from addresses.
    :::

5. Generate the input address, using the CryptoCore [`getAddress`](../references/api-reference.md#getaddress) command

    ```bash
    # Create a generateAddress API request, using the user's answer
    template='{"command":"getAddress", "slot": %d, "keyIndex":%d, "number": 1, "security": %d}'

    json_string=$(printf "$template" "$slot" "$keyIndex" "$securityLevel")

    echo "Generating an address with index $keyIndex and security level $securityLevel"

    # Open the serial terminal and enter the API request to generate the address 
    input=$(node ../node-scripts/serial.js "$json_string" | jq ".trytes[]" | tr -d '"')

    echo "If you haven't already done so, make sure that this address contains IOTA tokens: $input"
    ```

    :::info:
    The `serial.js` file uses the [SerialPort package](https://serialport.io/docs/guide-installation) to open a serial connection to the CryptoCore.

    You can find the code for this file on [GitHub](https://github.com/JakeSCahill/cryptocore-scripts/blob/master/node-scripts/serial.js).
    :::

6. Ask the user for an output address into which to deposit the full balance of the input address

    ```bash
    read -p "To which address would you like to send your IOTA tokens? " output

    while [[ ! $output =~ ^[A-Z9]*{81}$ ]]; do
        echo "Address is invalid"
        read -p "Please enter an 81 tryte address (without a checksum)" output
    done
    ```

7. Execute the `create-unsigned-bundle.js` script and pass it the minimum weight magnitude, input address, output address, and security level

    ```bash
    # Execute the create-unsigned-bundle.js script to create an unsigned bundle from the user's input
    unsigned_bundle_hash=$(node ../node-scripts/create-unsigned-bundle.js $MWM $input $output $securityLevel)

    if [[ ! $unsigned_bundle_hash =~ [A-Z9]{81} ]]; then
        echo "$unsigned_bundle_hash"
        exit 0
    fi
    ```

8. Execute the `generate-auth.js` script to generate the `auth` argument and pass it to the [`signBundleHash`](../references/api-reference.md#signbundlehash) command to sign the bundle hash on the CryptoCore

    ```bash

    # Execute the generate-auth.js script to generate a valid auth parameter for the signTransaction endpoint
    auth=$(node ../node-scripts/generate-auth.js $slot $keyIndex $unsigned_bundle_hash)

    # Create an API request, using the user's answers
    sign_bundle_template='{"command":"signBundleHash", "slot": %d, "keyIndex":%d,"bundleHash":"%s","security":%d, "auth":"%s"}'

    sign_bundle_json_string=$(printf "$sign_bundle_template" "$slot" "$keyIndex" "$unsigned_bundle_hash" "$securityLevel" "$auth")

    echo "$sign_bundle_json_string"

    echo "Signing transaction"

    # Open the serial terminal and enter the API request to create a zero-value transaction
    signature=$(node ../node-scripts/serial.js "$sign_bundle_json_string" | jq ".trytes[]" | tr -d '"' | rev | cut -c 2- | rev|tr -d '\n')
    ```

9. Pass the signature to the `add-signature-to-bundle.js` script to add it to the bundle and attach the transactions to the Tangle

    ```bash
    result=$(node ../node-scripts/add-signature-to-bundle.js $MWM $signature)

    echo "$result"
    ```

## Step 4. Add the signature to the bundle

In this step, you write a script that adds a signature to the bundle that you saved in the `create-unsigned-bundle.js` script.

1. Create a new file called `create_tx.sh`

    ```bash
    sudo nano create_tx.sh
    ```

2. Use the first argument that is passed to the script to connect to a node on either the Devnet or the Mainnet

    ```js
    // Get the first argument that was passed to this script
    // This should be a minimum weight magnitude (14 or 9)
    const network = parseInt(process.argv[2]);

    // Define a node for each IOTA network
    const nodes = {
            devnet: 'https://nodes.devnet.iota.org:443',
            mainnet: `https://nodes.iota.org:443`
    }

    // Connect to the correct IOTA network, depending on the user's
    // selection in the main script
    if (network === 14) {
        iota = Iota.composeAPI({
            provider: nodes.mainnet
            });
    } else {
        iota = Iota.composeAPI({
            provider: nodes.devnet
            });
    }
    ```

3. Convert the signature in the second argument to trits and store it in a variable

    ```js
    // Get the second argument that was passed to this script
    // This should be a signature
    const signature = process.argv[3];
    const signatureTrits = Converter.trytesToTrits(signature)
    ```

4. Read the bundle trits from the file that was saved by the `create-unsigned-bundle.js` script

    ```js
    let bundle = new Int8Array(fs.readFileSync('../bash-scripts/bundle'));
    ```

5. Add the signature to the bundle, starting from the second transaction

    ```js
    // Transaction 0 is the output transaction, so start adding the signature fragments, starting from the second transaction in the bundle
    bundle.set(Bundle.addSignatureOrMessage(bundle, signatureTrits, 1));
    ```

    :::info:
    This method overwrites the bundle array with new transactions that include the signature fragments.
    :::

6. Convert each transaction in the bundle array to trytes

    ```js
    const trytes = []
    for (let offset = 0; offset < bundle.length; offset += Transaction.TRANSACTION_LENGTH) {
        trytes.push(Converter.tritsToTrytes(bundle.subarray(offset, offset + Transaction.TRANSACTION_LENGTH)));
    }
    ```

7. Use the `sendTrytes()` method to attach the transaction trytes to the Tangle

    ```js
    // We need the bundle to be in order head to tail before sending it to the node
    iota.sendTrytes(trytes.reverse(), depth, network)
    .then(bundle => {
        console.log(`Sent bundle: ${JSON.stringify(bundle, null, 1)}`)
    })
    .catch(error => {
        console.log(error);
    });
    ```

:::success:
You have just written a command-line interface (CLI) program that uses the CryptoCore API to sign a bundle, and attaches the transactions to the Tangle.
:::

## Run the code

To get started you need [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed on your device.

If you don't have a JavaScript development environment, or if this is your first time using the JavaScript client library, complete our [getting started guide](root://client-libraries/0.1/getting-started/js-quickstart.md).

In the command-line, do the following:

1. Clone the repository and change into the `cryptocore-scripts/node-scripts` directory

    ```bash
    git clone https://github.com/JakeSCahill/cryptocore-scripts
    cd cryptocore-scripts/node-scripts
    ```

2. Install the packages

    ```bash
    sudo npm i
    ```

    You should see something like the following:

    ```
    npm notice created a lockfile as package-lock.json. You should commit this file.
    added 128 packages from 68 contributors and audited 338 packages in 34.171s

    2 packages are looking for funding
    run `npm fund` for details

    found 0 vulnerabilities
    ```

3. Run the `send_value_tx.sh` script and respond to the prompts

    ```bash
    cd ~/cryptocore-scripts/bash-scripts
    sudo ./send_value_tx.sh
    ```

4. Follow the prompts

You should see the transaction object that was sent to the node. For example:

```js
{
  hash: 'YQNCKGIGOZJJJGMNMAVWKJGDJLQOXDTADIJTYU9HBIGKTDKUXHBMOBXVZYWAUWOSKKYUSUIDVPNZZ9999',
  signatureMessageFragment: 'ODGAADTCGDGDPCVCTCGADBGARBOBVBVBYBEAFCYBACVBNBEAPBACYBWBEAMBACHCZBCCYBMBYBACOBGAQD99999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999',
  address: '999999999999999999999999999999999999999999999999999999999999999999999999999999999',
  value: 0,
  obsoleteTag: 'CWYPTOCORE99999999999999999',
  timestamp: 1059930011,
  currentIndex: 0,
  lastIndex: 0,
  bundle: 'DAIHSSITPKSZRVHDTVLXIBPYGKLGKZBRDZSGRFYGIBXTPCZFIL9JAJDEGABAUVCJNHWAUULGXRBDGNDIZ',
  trunkTransaction: 'HEWJUPLE9GHUDOFGCARASDIUDMXDQSV99WBSJYIXWHVCMHFGIXRTTTHBIEJGUPHCQQQGEZMZPRSRA9999',
  branchTransaction: 'HEWJUPLE9GHUDOFGCARASDIUDMXDQSV99WBSJYIXWHVCMHFGIXRTTTHBIEJGUPHCQQQGEZMZPRSRA9999',
  tag: 'CRYPTOCORE99999999999999999',
  attachmentTimestamp: 1581607894939,
  attachmentTimestampLowerBound: 0,
  attachmentTimestampUpperBound: 11,
  nonce: 'ICCFPGA9B9999999UVNPNVVMMMM'
}
```

You can copy the `hash` field of your transaction object and paste it into a [Tangle explorer](https://utils.iota.org/).

If the Tangle explorer doesn't display your transaction after 5 minutes, the node may not have sent your transaction to its neighbors.

To resend your transaction, you can pass the transaction trytes in the `attached-transaction-trytes/zero_value_transaction.txt` file to the [`storeAndBroadcast()`](https://github.com/iotaledger/iota.js/tree/next/packages/core#corestoreandbroadcasttrytes-callback) method in the JavaScript client library.

## Next steps

Add how you could improve this script


