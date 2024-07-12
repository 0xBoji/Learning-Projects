# Implement functions for Calculating interest and loan repayment

Hey y’all! we’re now at the endgame, we've assembled most of the core functionalities of our lending dApp. Now, it's time to suit up and make sure our dApp can help users repay their loans and retrieve their collateral without a hitch. Once we complete this mission, our dApp will be complete and ready to rock. So let’s make it happen! 👌

## Quick Checkout

Just like before let’s quickly swap out our boilerplate with the corresponding frontend with all the functionalities.

```solidity
git checkout origin/boilerplate04
```

## Clearing up the debts

Let’s start by creating an external function called `replayLoan`, this function will take the address of the user as its parameter whilst performing all checks and operations required for the Loan repayment.

```solidity
function repayLoan(address user) external  {
	// Code goes here
}
```

The very first thing we need to do is to check if the user i.e. `msg.sender` has an active loan and also whether that user hasn’t exceeded the loan duration. We can easily get this sorted by

- First, fetching the loan of the user from our `loans` mapping.
- Then, verify if the `active` field of `loan` object is true, in case it’s false then the user doesn’t have any active loans
- Lastly, ensure that the loan duration (30 days) has not been exceeded. Calculate `daysElapsed` by finding the difference between `block.timestamp` and the loan's `timestamp` field. This difference gives the elapsed time in seconds, which we then divide by `SECONDS_IN_A_DAY` (86400) to get the days elapsed.

```solidity
function repayLoan(address user) external  {
	Loan storage loan = loans[msg.sender];
	require(loan.active, "No active loan");
	uint256 daysElapsed = (block.timestamp - loan.timestamp) / SECONDS_IN_A_DAY;
	require(daysElapsed <= 30, "Loan duration exceeded 30 days");
}
```

Now we calculate the total interest on the `loan.amount` using the `INTEREST_RATE` constant and `daysElapsed`. Since `INTEREST_RATE` is annual, for a single day it equates to `INTEREST_RATE / 365`.

![img](https://lh7-us.googleusercontent.com/docsz/AD_4nXd7qkaHCT5EpMBBfbMHa5eaMxUepo-3fHsnzjPSLmd5ubd_s_CWqN2etXwifZMwcIoP76AfYylciycky-kEGhRZdcyMfkz2KOzxNFEuzApV9jvQRwYCR0O7TJrF9KpxNV_lAM0hfcC5Lv1uz9X-KwCRBDaT?key=MMRLnc_CBL9PgIVmx3Nxig)

By adding the `interest` to `loan.amount`, we can also calculate the total amount to be repaid.

```solidity
function repayLoan(address user) external  {
	Loan storage loan = loans[user];
	require(loan.active, "No active loan");
	uint256 daysElapsed = (block.timestamp - loan.timestamp) / SECONDS_IN_A_DAY;
	require(daysElapsed <= 30, "Loan duration exceeded 30 days");
	uint256 interest = (loan.amount * INTEREST_RATE * daysElapsed) / 36500;
	uint256 totalRepayment = loan.amount + interest;
}
```

## Core-cting Debts

Now all that’s left for us to do is to receive back the BTC that was borrowed and update the concerned fields.

Using the `safeTransferFrom` function (more on these in the next lesson), we securely send the `totalRepayment` amount to our smart contract. `safeTransferFrom` is reliable because it includes extra checks to prevent accidental transfers. If any conditions aren't met like insufficient funds then the transfer is automatically reverted, ensuring tokens are never lost

```solidity
function repayLoan(address user) external  {
	Loan storage loan = loans[user];
	require(loan.active, "No active loan");
	uint256 daysElapsed = (block.timestamp - loan.timestamp) / SECONDS_IN_A_DAY;
	require(daysElapsed <= 30, "Loan duration exceeded 30 days");
	uint256 interest = (loan.amount * INTEREST_RATE * daysElapsed) / 36500;
	uint256 totalRepayment = loan.amount + interest;
	BTC.safeTransferFrom(user, address(this), totalRepayment);
}
```

Now we can change the `active` field of the loan to false, indicating that the loan is now inactive. Let’s also decrease the `totalBorrowed` by loan amount and finally emit the `LoanRepaid` event to broadcast that the Loan repayment is successful. This is how our function will look at the end.

```solidity
function repayLoan(address user) external  {
	Loan storage loan = loans[user];
	require(loan.active, "No active loan");
	uint256 daysElapsed = (block.timestamp - loan.timestamp) / SECONDS_IN_A_DAY;
	require(daysElapsed <= 30, "Loan duration exceeded 30 days");
	uint256 interest = (loan.amount * INTEREST_RATE * daysElapsed) / 36500;
	uint256 totalRepayment = loan.amount + interest;
	BTC.safeTransferFrom(user, address(this), totalRepayment);
	loan.active = false;
	totalBorrowed = totalBorrowed - loan.amount;
	emit LoanRepaid(msg.sender, loan.amount, interest);
}
```

## Some more Query Helper Functions

Let’s quickly implement additional functions to retrieve details stored in our contract.

- **calculateInterest(address user)**: Calculates interest on an active loan for a specific user.
- **getLoanDetails(address borrower)**: Retrieves details of an active loan for a specific borrower.
- **getLenderBalance(address lender)**: Retrieves the current balance of BTC deposited by a lender.
- **getTotalStaked()**: Retrieves the total amount of BTC currently staked across all lenders.
- **getTotalBorrowed()**: Retrieves the total amount of BTC borrowed by all users.
- **getUserBorrowed(address user)**: Retrieves the current amount of BTC borrowed by a user.

```solidity
function calculateInterest(address user) external view returns (uint256) {
	Loan storage loan = loans[user];
	if (loan.active) {
		uint256 daysElapsed = (block.timestamp - loan.timestamp) / SECONDS_IN_A_DAY;
		uint256 interest = (loan.amount * INTEREST_RATE * daysElapsed) / 36500;
		return interest;
	}
}

function getLoanDetails(address borrower) external view returns (Loan memory) {
	return loans[borrower];
}

function getLenderBalance(address lender) external view returns (uint256) {
	return lenderBalances[lender];
}

function getTotalStaked() external view returns (uint256) {
	return totalStaked;
}

function getTotalBorrowed() external view returns (uint256) {
	return totalBorrowed;
}
```

## Let’s fire up the dApp

You’ll need to follow the same steps as in the previous lessons to get things running. We’ll drop the same set of instructions here as well for you guys to follow.

First, let’s create a `.env` in the root directory and paste in your private key against the `PRIVATE_KEY` variable.

![img](https://lh7-us.googleusercontent.com/docsz/AD_4nXcA6qf17Wo1_K3WUN2OvtBA4-PnmqWNRwldmdNtpnn4O8Qfa_X_GqrHEXapCyY7ee9wz_ViyOsRXTs5iCwtoVBn70RqoXMNbL0wdZ-j4-NwbNmVZgYfTy6zrXZ2vZLLfrI9mf54tE_YvncNZBFRXEUtfEZF?key=MMRLnc_CBL9PgIVmx3Nxig)

Now let’s install all the dependencies and compile our contract using :

```solidity
npm install
npx hardhat compile
```

Let’s deploy our contract using hardhat ignition just like last time

```solidity
npx hardhat ignition deploy .\\ignition\\modules\\Deploy.js --network core_testnet
```

Copy the addresses of **DAPP, USD, and BTC** Contract addresses from Terminal and paste them against the corresponding variables in the **`.env`** file present inside `*./interface/folder.*`

![img](https://lh7-us.googleusercontent.com/docsz/AD_4nXdYW0w1QPgj8n33J_Ip2U_5WsJ1EzXhHFnkz_h7cbBI64uyf5-Erv8R_evpJggw7vWXX0OlGk_aCdiNnaaTE8VSLJQojNDhAMQYFY-iCl9odMBWReEld95s_9MI8Pc_Yyi9iEXeMy6dQRAPvykRwYwFcO2K?key=MMRLnc_CBL9PgIVmx3Nxig)

Navigate to the react app and install the dependencies :

```solidity
cd interface
npm install
```

Copy the .json files containing the ABI from `*./artifacts/contracts*` to `*./interface/src/ABI/*` folder. You'll need to copy the following `.json` files from their respective folders

- Bitcoin.json
- CoreLoanPlatform.json
- IERC20.json
- USD.json

![img](https://lh7-us.googleusercontent.com/docsz/AD_4nXfMdG0npnqmorm-ci0ya55yOnM81dj3IsTLAuE1yhqEEJxDci7NmOn_ELTcqC-DrCtphyqWMkzWz4qy7eYw8UZ0UuPtJG-J3wlyeTnr8w-IguytJN_EwlhjSagCGsokqfCcO41So3VlsxrZnp5zZBv4Zzy9?key=MMRLnc_CBL9PgIVmx3Nxig)

Finally, let’s launch the front end using:

```solidity
npm start
```

## The Payback

Let’s test out all the functionalities we have imbued into our dApp. We’ll start with staking some BTC because our App repurposes the staked BTC for lending out BTC to users in need.

### Stake ‘em: Start Lending BTC

Follow these steps to start lending BTC:

- Click on the Connect Wallet button to connect your wallet.
- Use the faucet to get 100 USD and 10 BTC tokens.
- Click on the Start Lending button to navigate to the lending page.
- Enter the amount of BTC you want to stake (up to 10 BTC).
- Click the Stake button.
- Approve the transaction in the MetaMask popup that appears.

![img](https://lh7-us.googleusercontent.com/docsz/AD_4nXdd6sOb5AJQcGszmAUdRYij5_qyy3TkHqGeRs0Va52atMJkWTZnHIgUVxphbda3s4mCow0ztk-iFja3YiMzf5SoNtZluK4gN2Uw6SZU3xfqpzWjydJjLrrLwTfd1Zz3FrIdjVXIY7xL6ZueL2v1DQQq6vXD?key=MMRLnc_CBL9PgIVmx3Nxig)

### Borrow ‘em: Borrow BTC Using USD Collatera**l**

Use your staked BTC to provide funds for users who want to borrow BTC by using USD as collateral. Follow the steps below:

- Click on the 'Borrowing' link in the navigation bar to access the Borrow page.
- Enter the amount of USD you wish to deposit as collateral.
- Click the *Stake* button.
- Approve the transaction in the MetaMask popup that appears.
- Enter the amount of BTC you want to borrow.
- Click the *Borrow BTC* button.
- Approve the transaction in the MetaMask popup that appears.

![img](https://lh7-us.googleusercontent.com/docsz/AD_4nXcgbjkpP7D35QAaBa8H2Ly7bJEyWGQEn_NfwJH9TlX48Za61jzIWMI4fTGr_Ikto4Du6j2aexEn_ll_SH11UGqsDWo7ULRqp0v5C5_XbaYvaJIoVMM7140bKfvxi9V0mM3ImeE878s0kwVynCth63Z9yCc?key=MMRLnc_CBL9PgIVmx3Nxig)

### Repay ‘em: Repay Borrowed BTC and Withdraw Collateral

Repay the borrowed BTC with interest and then withdraw your deposited collateral. Follow these steps:

- Click on the *Repay loan* button.
- Approve the transaction in the MetaMask popup that appears.
- Enter the amount of collateral you wish to withdraw in the *Stake and Withdraw Collateral* section.
- Click the *Withdraw* button.
- Approve the transaction in the MetaMask popup that appears.

![img](https://lh7-us.googleusercontent.com/docsz/AD_4nXcJnT9zZmeaWgiSSTUrwt_QmTht3GDrpAPNvZxliwGSSbpfN6neXo1H3X1EMsyx0jXbs0WTGEvCys7l4JiAZVwdvfZxH6NYFQ9tzoL9YN6uK6VMIZuFJSuhO7rZPBtOwgpDvlZdTHS0sMZxRNiw1JUxXZfr?key=MMRLnc_CBL9PgIVmx3Nxig)

And there we have it, all the functionalities of our Lending dApp and works well as intended.

## Pushing it to Git

Make sure you are in `build-lending-dapp-on-core` folder.

As always, remember to push your code to your GitHub repository using the following commands.

```solidity
git add .
git commit -m "code update"
git push
```

## That’s a Wrap

Hey y’all! We’ve reached the Endgame! We’ve successfully assembled the core functionalities of our lending dApp. From staking BTC and borrowing with USD collateral to repaying loans and withdrawing collateral, we’ve ensured everything works seamlessly. Our dApp is now complete and ready to rock. Great job 🎉