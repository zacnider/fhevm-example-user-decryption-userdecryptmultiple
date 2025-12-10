# EntropyUserDecryptMultiple

User decrypt multiple values using EntropyOracle and FHE.allow

## 🚀 Quick Start

1. **Clone this repository:**
   ```bash
   git clone https://github.com/zacnider/fhevm-example-user-decryption-userdecryptmultiple.git
   cd fhevm-example-user-decryption-userdecryptmultiple
   ```

2. **Install dependencies:**
   ```bash
   npm install --legacy-peer-deps
   ```

3. **Setup environment:**
   ```bash
   npm run setup
   ```
   Then edit `.env` file with your credentials:
   - `SEPOLIA_RPC_URL` - Your Sepolia RPC endpoint
   - `PRIVATE_KEY` - Your wallet private key (for deployment)
   - `ETHERSCAN_API_KEY` - Your Etherscan API key (for verification)

4. **Compile contracts:**
   ```bash
   npm run compile
   ```

5. **Run tests:**
   ```bash
   npm test
   ```

6. **Deploy to Sepolia:**
   ```bash
   npm run deploy:sepolia
   ```

7. **Verify contract (after deployment):**
   ```bash
   npm run verify <CONTRACT_ADDRESS>
   ```

**Alternative:** Use the [Examples page](https://entrofhe.vercel.app/examples) for browser-based deployment and verification.

---

## 🚀 Standard workflow
- Install (first run): `npm install --legacy-peer-deps`
- Compile: `npx hardhat compile`
- Test (local FHE + local oracle/chaos engine auto-deployed): `npx hardhat test`
- Deploy (frontend Deploy button): constructor arg is fixed to EntropyOracle `0x75b923d7940E1BD6689EbFdbBDCD74C1f6695361`
- Verify: `npx hardhat verify --network sepolia <contractAddress> 0x75b923d7940E1BD6689EbFdbBDCD74C1f6695361`

## 📋 Overview

This example demonstrates **user decrypting multiple values** in FHEVM with **EntropyOracle integration**:
- Integrating with EntropyOracle for batch operations
- Storing multiple encrypted values with user-specific access control
- Using entropy to enhance user decryption patterns for multiple values
- Batch operations for efficient multi-value user decryption

## 🎯 What This Example Teaches

This tutorial will teach you:

1. **How to store multiple encrypted values** with user-specific access
2. **How to allow multiple users to decrypt different values** using FHE.allow
3. **How to perform batch operations** for multiple values
4. **How to enhance multiple values with entropy** from EntropyOracle
5. **The importance of `FHE.allow()`** for user-specific decryption in batch operations

## 💡 Why This Matters

Batch user decryption is essential for efficient FHEVM operations. With EntropyOracle, you can:
- **Process multiple values** for different users in a single transaction
- **Add randomness** to multiple encrypted values without revealing them
- **Enhance security** by mixing entropy with user-encrypted data in batch
- **Reduce gas costs** by batching operations
- **Learn efficient patterns** for handling multiple user-specific encrypted values

## 🔍 How It Works

### Contract Structure

The contract has four main components:

1. **Basic Single Storage**: Store and allow user to decrypt a single value at a specific key
2. **Batch Storage**: Store and allow multiple values for different users at once
3. **Entropy Request**: Request randomness from EntropyOracle
4. **Entropy-Enhanced Storage**: Combine user values with entropy (single and batch)

### Key Functions

#### 1. Single Value Storage

```solidity
function storeAndAllow(
    uint256 key,
    externalEuint64 encryptedInput,
    bytes calldata inputProof,
    address user
) external {
    euint64 internalValue = FHE.fromExternal(encryptedInput, inputProof);
    FHE.allowThis(internalValue);
    FHE.allow(internalValue, user);  // Allow specific user
    encryptedValues[key] = internalValue;
    allowedUsers[key] = user;
}
```

#### 2. Batch Storage

```solidity
function storeAndAllowBatch(
    uint256[] calldata keys,
    externalEuint64[] calldata encryptedInputs,
    bytes[] calldata inputProofs,
    address[] calldata users
) external {
    for (uint256 i = 0; i < keys.length; i++) {
        euint64 internalValue = FHE.fromExternal(encryptedInputs[i], inputProofs[i]);
        FHE.allowThis(internalValue);
        FHE.allow(internalValue, users[i]);  // Allow specific user for each value
        encryptedValues[keys[i]] = internalValue;
        allowedUsers[keys[i]] = users[i];
    }
}
```

#### 3. Entropy-Enhanced Batch

```solidity
function storeAndAllowBatchWithEntropy(
    uint256[] calldata keys,
    externalEuint64[] calldata encryptedInputs,
    bytes[] calldata inputProofs,
    address[] calldata users,
    uint256 requestId
) external {
    euint64 entropy = entropyOracle.getEncryptedEntropy(requestId);
    FHE.allowThis(entropy);
    
    for (uint256 i = 0; i < keys.length; i++) {
        euint64 internalValue = FHE.fromExternal(encryptedInputs[i], inputProofs[i]);
        FHE.allowThis(internalValue);
        euint64 enhancedValue = FHE.xor(internalValue, entropy);
        FHE.allowThis(enhancedValue);
        FHE.allow(enhancedValue, users[i]);  // Allow user to decrypt enhanced value
        encryptedValues[keys[i]] = enhancedValue;
        allowedUsers[keys[i]] = users[i];
    }
}
```

## 🧪 Testing

Run the test suite:

```bash
npm test
```

The tests cover:
- Single value storage and allow
- Batch storage and allow
- Entropy-enhanced storage (single and batch)
- Error handling (mismatched arrays, empty arrays, duplicate keys)
- Value retrieval by key

## 📚 Related Examples

- [EntropyUserDecryption (Single)](../user-decryption-userdecryptsingle/) - Single value user decryption
- [EntropyEncryptMultiple](../encryption-encryptmultiple/) - Encrypt multiple values
- [EntropyPublicDecryptMultiple](../public-decryption-publicdecryptmultiple/) - Public decrypt multiple values

## 🔗 Links

- [EntropyOracle Documentation](https://entrofhe.vercel.app)
- [FHEVM Documentation](https://docs.zama.org/protocol)
- [Examples Hub](https://entrofhe.vercel.app/examples)

## 📝 License

BSD-3-Clause-Clear
