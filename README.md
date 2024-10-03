# evm-data-location

Summary of [How to fix 'Data location must be memory or calldata‘ | Where can the EVM read and write data?](https://www.cyfrin.io/blog/fixing-data-location-must-be-memory-or-calldata)

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

Calldata as an EVM concept is the data sent to a transaction as part of a smart contract transaction. For example, when creating a contract, calldata would be the constructor code of the new contract. Calldata is immutable, and can be read with instructions CALLDATALOAD, CALLDATASIZE, and CALLDATACOPY. 

In Solidity, only function parameters can be considered calldata because only functions can be called with calldata. Calldata cannot be changed once sent in a transaction. It must be stored in another data structure (like the stack, memory, storage, etc) to be manipulated. 




