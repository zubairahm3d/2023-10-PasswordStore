# Security Issues in PasswordStore Smart Contract

## [H-1] Storing Passwords on Blockchain is Not Safe as Anyone Can Read the Password

### Description
On a blockchain, any data, even if marked as private, can be read by users. Therefore, storing sensitive information such as passwords directly on the blockchain, even in private variables, is not secure.

### Impact
Anyone with access to the blockchain can read the stored password, leading to potential unauthorized access and security breaches.

### Proof of Concept
The following demonstrates how a password stored in a smart contract can be retrieved:

```bash
forge script script/DeployPasswordStore.s.sol --rpc-url http://127.0.0.1:8545 --broadcast --private-key $PRIVATE_KEY
```

**Output:**
```
0: contract PasswordStore 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

Retrieve the storage slot containing the password:
```bash
cast storage 0x5FbDB2315678afecb367f032d93F642f64180aa3 1 --rpc-url http://127.0.0.1:8545
```

**Output:**
```
0x6d7950617373776f726400000000000000000000000000000000000000000014
```

Decode the bytes to reveal the password:
```bash
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

**Output:**
```
myPassword
```

### Recommended Mitigation
To securely handle passwords, encrypt them off-chain before storing on the blockchain. Only the encrypted version should be stored, and decryption should occur off-chain when needed.

## [H-2] Lack of Access Control in `PasswordStore::setPassword` Function

### Description
Access controls are critical for smart contract security, ensuring that only authorized users, such as the contract owner, can perform sensitive operations. The `PasswordStore::setPassword` function currently lacks access control, allowing anyone to set the password.

### Impact
Any user, including non-owners, can call the `setPassword` function and modify the password, undermining the contract's security and owner privileges.

### Proof of Concept
The following test demonstrates that a non-owner can set the password:

```javascript
function test_anyone_can_set_the_password() public {
    vm.startPrank(zubair);
    string memory myPassword = "Zubair";
    // Non-owner sets the password
    passwordStore.setPassword(myPassword);
    vm.stopPrank();

    vm.startPrank(owner);
    // Owner retrieves the password
    string memory password = passwordStore.getPassword();
    // Confirms that the non-owner successfully set the password
    assertEq(password, myPassword);
}
```

### Recommended Mitigation
Add an `onlyOwner` modifier to restrict the `setPassword` function to the contract owner:

```diff
-    function setPassword(string memory newPassword) external
+    function setPassword(string memory newPassword) external onlyOwner
```

## [I-1] Incorrect Parameter Reference in `PasswordStore::getPassword` Documentation

### Description
The documentation for the `PasswordStore::getPassword` function incorrectly references a `newPassword` parameter. As a getter function, `getPassword` does not accept any parameters, unlike a setter function.

### Impact
None, as this is a documentation error and does not affect the contract's functionality.

### Recommended Mitigation
Remove the incorrect parameter reference in the documentation:

```diff
-    * @param newPassword The new password to set.
```