### [H-1] Variable stored in storage on-chain are visible to anyone.

**Description:** All data stored on-chain , and can be read directly from the blockchain . The`PasswordStore::s_password` variable is intended to be private variable and only accessed through the `PasswordStore::getPassword` function , which is intended to be only called by the owner of the contract . 

**Impact:** Anyone can read the private password , severaly breaking the functionality of the protocol. 

**Proof of Concept:** (Proof of Code)

The below test case shows how anyone anyone read the password directly from the blockchain . 

1. Create a locally running chain 
```bash
make anvil
```

2. Deploy the contract to the chain 

```
make deploy
```

3. Run the storage tool 
We use `1` because that's the storage slot of `s_password` in the contract.

```
cast stroage <ADDRESS_HERE> 1 --rpc-url http://127.0.0.1:8545
```

You'll get an output looks like this : 
```
0x6d7950617373776f726400000000000000000000000000000000000000000014
```


You can then parse that hex to the strong with : 

```
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014 
```

And you got the output of :

```
myPassword
```

**Recommended Mitigation:**
Due to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain, and then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the password. However, you'd also likely want to remove the view function as you wouldn't want the user to accidentally send a transaction with the password that decrypts your password.




### [H-2] `PasswordStore::setPassword` has no access controls meaning a non-owner could change the password . 

**Description:**  `The PasswordStore::setPassword` function is set to be an `external` function, however the natspec of the function and overall purpose of the smart contract is that `This function allows only the owner to set a new password`.

```javascript
    function setPassword(string memory newPassword) external {
@>      // @audit - There are no access controls here
        s_password = newPassword;
        emit SetNetPassword();
    }

```

**Impact:**  Anyone one set/change the password of the contract , severly breaking the contract intended functionality . 


**Proof of Concept:**
Add the following to the `PasswordStore.t.sol` test file.

<details>
<summary> Code </summary>

```javascript

    function test_anyone_can_setpassword(address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory expectedPassword = "myNewPassword";
        passwordStore.setPassword(expectedPassword);

        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }
```

</details>



**Recommended Mitigation:** Add an access control modifier to the `setPassword` function . 

```javascript
if (msg.sender != s_owner) {
    revert PasswordStore__NotOwner();
}
```



### [I-1] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect

**Description:** 

```javascript
    /*
    * @notice This allows only the owner to retrieve the password.
     // @audit their is no newPassword parameter
     * @param newPassword The new password to set.
     */
    function getPassword() external view returns (string memory) {

    }
```
The natspec for the function `PasswordStore::getPassword` indicates it should have a parameter with the signature `getPassword(string)`. However, the actual function signature is `getPassword()`.

**Impact:** The natspec is incorrect

**Recommended Mitigation:** Remove the incorrect natspec line .

```diff
- * @param newPassword The new password to set.
```








