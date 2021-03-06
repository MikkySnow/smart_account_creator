# Smart Account Creator for EOS

This is a smart contract for EOS specifically designed for people who have their EOS 
on an exchange and don't have their own EOS account yet. It removes the need for a third-party like a friend or
an account creation service. Since it's a smart contract, the account creation happens instantly, automatically and trustless.

## How to use?
There are two ways this smart contract can be used. Method 1 is the easiest since the memo can be constructed without the help of software. The disadvantage of Method 1 is that due to the length of the resulting memo, it does not work with some exchanges such as Binance or Huobi. Method 2 requires code to register the order with the smart contract as well as to generate the memo, but it's the only method that works with those exchanges.

### Method 1 (does *not* work with Binance or Huobi)
Send at least 3 EOS to the contract which is deployed at the EOS account ```accountcreat```. In the memo, 
you give the desired account name, the owner public key and the active public key separated by the ```-``` character. 

For example, if your account name is ```mynewaccount```, your owner key is ```EOS6ra2QHsDr6yMyFaPaNwe3Hz8XmYRj3B68e5tbDchyPTTasgFH9``` 
and your active key is ```EOS8WcL1CroNrXfdphkohCmea1Jgp7TpqQXrkpcF1gETweeSnphmJ```, the memo string would be:

```
mynewaccount-EOS6ra2QHsDr6yMyFaPaNwe3Hz8XmYRj3B68e5tbDchyPTTasgFH9-EOS8WcL1CroNrXfdphkohCmea1Jgp7TpqQXrkpcF1gETweeSnphmJ
```

Some exchanges like Binance don't allow to use a long memo. In this case or if you use the same key for owner and active, the second key including the ```-``` separator can be omitted. 
So that would be a valid memo string as well:

```
mynewaccount-EOS6ra2QHsDr6yMyFaPaNwe3Hz8XmYRj3B68e5tbDchyPTTasgFH9
```
If you need help, visit the [EOS Account Creator Website](https://eos-account-creator.com/eos/). It will assist you in generating they keys and building the correct memo string.

### Method 2 (*does* work with Binance and Huobi)
This method uses a commit-reveal scheme to register the owner and active public keys with the smart contract.
#### Registering your Public Keys with the smart contract
Let ACCOUNT_NAME be your 12 character EOS account name that you want to register. Generate a nonce like this:
```
openssl rand 8 -hex
```

Generate a sha256 hash like this, by appending the nonce you just created to your account name:
```
echo -n '$(ACCOUNT_NAME)$(NONCE)'| shasum -a256 | awk '{print $1}'
```
Now we're ready to register our account creation order with the smart contract like this:

```
cleos push action $(CONTRACT_ACCOUNT) regaccount '["$(SENDER)", "$(HASH)", "$(OWNER_PK)", "$(ACTIVE_PK)"]' 
```
The SENDER account will pay for the RAM required to store this order in the smart contract table. After successful account creation, this RAM will be released. If the account is not created, the RAM can be released after 3 hours by sending the action "clearexpired" to the contract.

We can now finally register the account by making an EOS token transfer to the CONTRACT_ACCOUNT like this:
```
cleos transfer $(SENDER) $(CONTRACT_ACCOUNT) "1.0000 EOS" '$(ACCOUNT_NAME)$(NONCE)'
```

## How does it work?
When you withdraw your EOS to the accountcreat smart contract, it will perform the following steps in order:

1. Create a new account using your specified name, owner key and active key
1. Buy 4KB of RAM for your new account with parts of the transferred EOS. Every account that is created on the EOS network needs 4 KB of RAM to exist.
1. Delegate and transfer 0.1 EOS for CPU and 0.1 EOS for NET.
1. Deduct our fee of 0.5% or a minimum of 0.1 EOS and forward the remaining EOS balance to your new account.

Should any of the above actions fail, the transaction will be rolled back which 
means the money will automatically be refunded to you.

If you want to use this code or need help modifying it to your needs, please contact me at: hello@eos-account-creator.com

## Peer reviews
If you want to review the code, that would be great. It's important for the community to have peer-reviewed, trusted, account creation smart contract code. Quick note to anyone wanting to verify the deployment (as there is no such things as etherscan yet, where you can upload the code): The code currently deployed on ```accountcreat``` is compiled with ```-Oz``` to save RAM.
