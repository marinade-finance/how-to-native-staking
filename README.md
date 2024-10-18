# How to: Marinad Native Staking

This tutorial is made for users who do not wish to interact with Marinade's app and still want to deposit SOL, or for apocapyltic events - to allow Marinade Native users to get out of staking even if Marinade's UI/BE services were turned off for whatever reason.

This is one of the points of Marinade Native - the SOL is always present in Solana's native stake accounts of the Solana's Stake program - and users always maintains `withdraw authority` while only giving away `stake authority`.

More on stake authorities in Solana docs: https://solana.com/docs/economics/staking/stake-accounts#understanding-account-authorities

The tutorial assumes that Solana CLI is installed (https://solana.com/docs/intro/installation#install-the-solana-cli) and user has a hardware wallet.

## Discover wallet to be used with CLI
First you need to figure out how to use your hardware wallet with CLI. To do that, run the following command, it should print out the primary public key of your wallet.
```bash
solana-keygen pubkey "usb://ledger?key=0"
```
> Tip: If you use multiple public keys, try changing the number at the end of the command, so e.g. `solana-keygen pubkey "usb://ledger?key=1"` or `solana-keygen pubkey "usb://ledger?key=1/0"`

> Tip: For the device to be discoverable through `solana-keygen` CLI, the device must not be in use by e.g. browser extension at the same time.

## Fund a stake account and authorize Marinade
Let's create a new stake account. Start by creating a new random wallet and storing it as `stake-account.json` file on your disk.
```bash
solana-keygen new --no-bip39-passphrase -s -o stake-account.json
```

Next, let's create the stake account on-chain, fund it with 1 SOL (Change the amount as needed) and authorize Marinade to manage it. (If you want to use different signer, change `"usb://ledger?key=0"` to reflect your needs - but keep the quote marks `"`.)
```bash
solana create-stake-account --keypair "usb://ledger?key=0" --stake-authority stWirqFCf2Uts1JBL1Jsd3r6VBWhgnpdPxCTe1MFjrq stake-account.json 1
```

You can now preview the transaction on Solana's explorer (e.g. https://explorer.solana.com). You will see that a new stake account is created and funded. You will also see that withdraw authority is set to signer of the transaction (you) and stake authority is set to Marinade (`stWirqFCf2Uts1JBL1Jsd3r6VBWhgnpdPxCTe1MFjrq`).

You can now use Solana CLI to preview your newly created stake account:
```bash
solana stake-account $(solana-keygen pubkey stake-account.json)
```
You should see output similar to this:
```
Balance: 1 SOL
Rent Exempt Reserve: 0.00228288 SOL
Stake account is undelegated
Stake Authority: stWirqFCf2Uts1JBL1Jsd3r6VBWhgnpdPxCTe1MFjrq
Withdraw Authority: YOUR PUBLIC KEY WILL BE PRESENT HERE
```

> Notice that the stake account is undelegated. Since you used Solana CLI, it will take Marinade systems some time to notice the account. Marinade delegates accounts automatically towards the end of each epoch, so no need to worry if nothing happens immediately.

> The `stake-account.json` file is no longer needed and can be deleted. You only need your ledger to access the funds if needed.

## Check your stake accounts in Marinade Native
You can use Solana CLI to see all of your stake account at any time:
```bash
solana stakes --withdraw-authority YOUR-PUBLIC-KEY-GOES-HERE
```
This is very useful as Marinade will split your initial stake account into multiple ones and will delegate them to many validators.

> This command supports JSON output (add `--output json`) and you can easily build e.g. your own monitoring utilities.

## Manual withdraw from Marinade Native
This is not a recommended way of removing stake from Marinade - this is a way to be used in case of emergency/apocalyptic-like scenarions when Marinade's UI is not available for whatever reason and you want to withdraw funds back to your wallet.

First, find your stake accounts using:
```bash
solana stakes --withdraw-authority YOUR-PUBLIC-KEY-GOES-HERE
```
Example output:
```
Stake Pubkey: 4mFGSGdeMZsfnAh58g9PpxFN5KtcqHWQRpZVBgxnFoLH
Balance: 1000 SOL
Rent Exempt Reserve: 0.00228288 SOL
Stake account is undelegated
Stake Authority: stWirqFCf2Uts1JBL1Jsd3r6VBWhgnpdPxCTe1MFjrq
Withdraw Authority: YOUR-PUBLIC-KEY-IS-HERE

Stake Pubkey: C6kBqxp2tvxvMaXz2FPN1f1iiQJD3R25N8kdM9PnxVXP
Balance: 1000.048358174 SOL
Rent Exempt Reserve: 0.00228288 SOL
Delegated Stake: 1000.046075294 SOL
Active Stake: 1000.046075294 SOL
Delegated Vote Account Address: 4f9n6atJ9jDXBfTCkk4tJAACq2amnW4rAEZUkg61Zbw7
Stake Authority: stWirqFCf2Uts1JBL1Jsd3r6VBWhgnpdPxCTe1MFjrq
Withdraw Authority: YOUR-PUBLIC-KEY-IS-HERE
```

In the example above, there are 2 stake accounts listed:
- `4mFGSGdeMZsfnAh58g9PpxFN5KtcqHWQRpZVBgxnFoLH` - this one is undelegated (could be because of some re-balancing by Marinade)
- `C6kBqxp2tvxvMaXz2FPN1f1iiQJD3R25N8kdM9PnxVXP` - this one is delegated

### Manually withdrawing fully de-activated account
The account which is NOT delegated, is NOT activating and is NOT in the de-activation process can be withdrawn using a single command:
```bash
solana withdraw-stake --keypair "usb://ledger?key=0" 4mFGSGdeMZsfnAh58g9PpxFN5KtcqHWQRpZVBgxnFoLH YOUR-PUBLIC-KEY ALL
```

> Tip: `4mFGSGdeMZsfnAh58g9PpxFN5KtcqHWQRpZVBgxnFoLH` is a public key of the stake account, see the example above. You will have to adjust this part of the command after you find your own stake accounts.

> Tip: Notice the `ALL` at the end of the command. This can be replaced by a number and you can control how much you are withdrawing.

### Manually withdrawing activated account
The account which IS activated or activating can be withdrawn in the following steps:

Regain the stake authority, so you can control the delegation state of the account:
```bash
solana stake-authorize --keypair "usb://ledger?key=0" --new-stake-authority YOUR-PUBLIC-KEY-GOES-HERE  C6kBqxp2tvxvMaXz2FPN1f1iiQJD3R25N8kdM9PnxVXP
```
> Tip: `C6kBqxp2tvxvMaXz2FPN1f1iiQJD3R25N8kdM9PnxVXP` is a public key of the stake account, see the examples above. You will have to adjust this part of the command after you find your own stake accounts.

Now, in case the account is not already in the process of being de-activated (e.g. because of Marinade's re-balancing) but is fully active or is just getting activated, let's de-activate it:
```bash
solana deactivate-stake --keypair "usb://ledger?key=0" C6kBqxp2tvxvMaXz2FPN1f1iiQJD3R25N8kdM9PnxVXP
```

Solana has a cool-down period of 1 epoch. So just wait for the end of the current epoch (you can see the progress using `solana epoch-info`). Your account will be fully de-activated and you will be able to withdraw as [in the simpler case above](#manually-withdrawing-fully-de-activated-account):
```bash
solana withdraw-stake --keypair "usb://ledger?key=0" C6kBqxp2tvxvMaXz2FPN1f1iiQJD3R25N8kdM9PnxVXP YOUR-PUBLIC-KEY ALL
```
