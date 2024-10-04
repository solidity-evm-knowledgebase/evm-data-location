# evm-data-location

Based on [How to fix 'Data location must be memory or calldata‘ | Where can the EVM read and write data?](https://www.cyfrin.io/blog/fixing-data-location-must-be-memory-or-calldata)

## EVM Read & Write Locations:

- stack
- memory
- storage
- transient storage
- calldata
- code
- return data

## EVM Write only Locations:

- logs

## EVM Read only locations:

- Transaction data (& Blobhash)
- Chain data
- Gas data
- Program Counter

## The EVM Stack

Data structure where items can only be added (push) or removed (pop) from the top. It is the only place in the EVM where operations can be done on the data (addition, subtraction, multiplication, left shifts...).

When a variable inside functions is created in Solidity, it is transformed into its 32 bytes representation and pushed onto the top of the stack. (objects being pushed to the stack must be <= 32 bytes).

The stack currently has a maximum limit of 1024 values (“stack too deep” error occurs when the solidity code results in too many variables on the stack)

The stack is temporary and objects on it are destroyed after the call context is comleted. It is also cheapest place (gas-wise) to store and retrieve data in the EVM.

## EVM Memory

There are times when the stack isn’t good enough to place data, so instead we use memory. Arrays for example, wouldn’t fit onto the stack (For arrays, we need to store each of the elements and the array length). So, under the hood, we call the MSTORE opcode to store data in the memory data structure of the EVM. (MLOAD is used to retrieve data from memory)

## EVM Storage

When data is stored as a state variable (in storage), it is stored permanently.

To store a variable in storage we use the SSTORE opcode and to retrieve a variable we use SLOAD.

For example:

```
contract MyContract{
uint256 myStorageVar = 7;     // This is in storage

function doStuff() public {
	uint256 myStackVar = 7; // This is on the stack
}
}
```

Storing data into storage is the most expensive way (gas wise) to store data in the EVM: data is stored permenantly, all EVM nodes must have the data persist even after a transaction has ended.

## EVM Calldata

Two types of calldata:
- The Solidity keyword calldata
- The EVM concept calldata

Calldata as an EVM concept is the data sent to a transaction as part of a smart contract transaction. For example, when creating a contract, calldata would be the constructor code of the new contract. Calldata is immutable, and can be read with instructions CALLDATALOAD, CALLDATASIZE, and CALLDATACOPY. (when the EVM needs to read the data we sent to a contract, it reads from the calldata).

In Solidity, only function parameters can be considered calldata because only functions can be called with calldata. Calldata cannot be changed once sent in a transaction. It must be stored in another data structure (like the stack, memory, storage, etc) to be manipulated. 

### Difference between memory and calldata in Solidity

```
function doStuff(string stuff) public {
// The above will not compile, throwing an error saying:
// TypeError: Data location must be "memory" or "calldata" for parameter in 
// function, but none was given
```

We need to tell the solidity compiler whether the data that is coming in will be stored in memory or calldata.

If memory:

- We can manipulate the stuff object (add to the string, save new strings, etc)
- We can call the doStuff function with data stored in memory or calldata

If calldata:

- We cannot manipulate the stuff object
- We can only call the doStuff function with data stored as calldata


Whenever we call a function from outside the blockchain (for example, calling transfer on an ERC20 contract and signing with Metamask or other browser wallets), that data is always sent as calldata. However, if a contract calls another function parameter, it can either send data as a calldata or memory. 

Calldata is deleted once call context has ended, and can be considered a temporary data location like the stack and memory.

## EVM Transient Storage

As of EIP-1153, there is now an additional location that acts like storage but is deleted after the transaction ends, making it another temporary storage location. However, unlike the stack, memory, and calldata, which are deleted after the calling context ends, transient storage is deleted after the transaction ends.

### Call Context

Whenever a function is called in a transaction (an external function call or internal), a new call context is created.

It works as an isolated environment for functions to store and manipulate data. It includes:
- Program Counter
- Available gas
- Stack
- Memory

A call context is ended when a RETURN, STOP, INVALID, or REVERT opcode is reached, or when a transaction reverts. 

In the example below, the two functions can have exactly the same variable name but will never overlap. That's because each function will get its own call context, with their own stack, memory, calldata...

## Code

One of the final places we can store data, is as a contract, aka in the “code” location of the EVM. This is pretty straightforward, and it’s why using variables in solidity labeled constant and immutable are unable to be changed. They are stored in the contract bytecode itself.

```
uint256 constant MY_VAR = 7;
uint256 immutable i_myVar = 7;
```
## EVM Data Structure cheat sheet


## Return data

The return data is the way a smart contract can return a value after a call. It can be set by contract calls through the RETURN and REVERT instructions and can be read by the calling contract with RETURNDATASIZE and RETURNDATACOPY.

Essentially, whenever you see the return keyword, that is going to create the RETURN opcode to store data into the return data location. 

```
function doStuff() public returns(uint256) {
	return uint256(7);
}
```

The return data is a bit odd, where calling the RETURN opcode will end the current call context and then pass the resulting data to the parent calling context as return data.The data can then be accessed with RETURNDATASIZE and RETURNDATACOPY. 

There is only one return data, and calling these opcodes will return the return data of the most recently ended call context. Return data does not persist and can be easily overwritten by a sub context calling the RETURN opcode. 


## Write but Not Read

### Logs

Logs are a storage location in the EVM where code is purely written. In Solidity, this is done with the emit keyword. 

```
event myEvent();
emit myEvent();
```

## Read, but not write

In the EVM, there are many places where the EVM can read data but not write.

Examples of this in Solidity:

```
msg.sender; 
block.chainid;
blobhash(0);
gasleft();
```

And many more [globally available units](https://docs.soliditylang.org/en/latest/units-and-global-variables.html)





