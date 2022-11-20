# Faucet_Contract_ERC20

A faucet is a program to give out a specified amount of tokens to anyone who requests them, by a time internal.

They're used for token promotion or to provide tokens to developers for testing.

We'll cover Interfaces, Events, Withdrawals, Time intervals, sendinf and receiving tokens.

We will use the existing ERC20 project we created in hardhat - https://github.com/AdeolaAdesina/ERC20_Token

Go into the contracts folder and create faucet.sol

We'll start with our license identifier and pragma solidity, then a contract definition:

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

contract Faucet {
    
}
```

Now we will create a state variable for the contract owner(type is address), we'll make it payable because we want to withdraw tokens from this address.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;



contract Faucet {
    address payable owner;
}
```


There has to be a limited amount to tokens that the faucet can distribute, so we'll need to use an Interface.

Interface allows one contract to talk to another contract on the blockchain, we will define the funcvtions we need to use.
Interface cannot have any function implementation, just the function signature.
All functions in an interface have to be external, cos they're to be called from another contract.

So we will define the transfer function.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

interface IERC20 {
    function transfer(address to, uint256 amount) external view returns (bool);
}

contract Faucet {
    address payable owner;
}
```

You'll notice there's no function body - that's because interfaces have no implementation.

Now let's create a function to check account balance.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

interface IERC20 {
    function transfer(address to, uint256 amount) external view returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

contract Faucet {
    address payable owner;
}
```


Now i'll declare a new variable of type IERC20 and name it token.

Next we will define our constructor.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

interface IERC20 {
    function transfer(address to, uint256 amount) external view returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

contract Faucet {
    address payable owner;
    IERC20 public token;

    constructor(address tokenAddress) payable {
        token = IERC20(tokenAddress);
        owner = payable(msg.sender);
    }
}
```

Now let's write the main functions that users will call to request tokens.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

interface IERC20 {
    function transfer(address to, uint256 amount) external view returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

contract Faucet {
    address payable owner;
    IERC20 public token;

    uint256 public withdrawalAmount = 50 * (10 ** 18);

    constructor(address tokenAddress) payable {
        token = IERC20(tokenAddress);
        owner = payable(msg.sender);
    }

    function requestTokens() public {
        token.transfer(msg.sender, withdrawalAmount);
    }
}
```

Now we don't want invalid accounts to call this function.
We also want to make sure that the smart contract has sufficient balance to handle the request.
The last check is going to make sure that enough time has elapsed since the last request, to do that we're going to use a mapping, then declare a variable called lockTime.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

interface IERC20 {
    function transfer(address to, uint256 amount) external view returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

contract Faucet {
    address payable owner;
    IERC20 public token;

    uint256 public withdrawalAmount = 50 * (10 ** 18);

    uint256 public lockTime = 1 minutes;

    mapping(address => uint256) nextAccessTime;

    constructor(address tokenAddress) payable {
        token = IERC20(tokenAddress);
        owner = payable(msg.sender);
    }

    function requestTokens() public {
        require(msg.sender != address(0), "Request must not originate from a zero account");
        require(token.balanceOf(address(this)) >= withdrawalAmount, "Insufficient balance");
        require(block.timestamp >= nextAccessTime[msg.sender], "Insufficient time elapsed since last withdrawal");

        nextAccessTime[msg.sender] = block.timestamp + lockTime;

        token.transfer(msg.sender, withdrawalAmount);
    }
}
```

Now it's time to create a receive function so we can preload our faucet with tokens.

Inside the receive function, we can broadcast an event when the smart contract receives funds.
We will first define an event variable. Then we'll use the emit keyword in the receive function.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

interface IERC20 {
    function transfer(address to, uint256 amount) external view returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

contract Faucet {
    address payable owner;
    IERC20 public token;

    uint256 public withdrawalAmount = 50 * (10 ** 18);

    uint256 public lockTime = 1 minutes;

    event Deposit(address from, uint256 amount);

    mapping(address => uint256) nextAccessTime;

    constructor(address tokenAddress) payable {
        token = IERC20(tokenAddress);
        owner = payable(msg.sender);
    }

    function requestTokens() public {
        require(msg.sender != address(0), "Request must not originate from a zero account");
        require(token.balanceOf(address(this)) >= withdrawalAmount, "Insufficient balance");
        require(block.timestamp >= nextAccessTime[msg.sender], "Insufficient time elapsed since last withdrawal");

        nextAccessTime[msg.sender] = block.timestamp + lockTime;

        token.transfer(msg.sender, withdrawalAmount);
    }

    receive() external payable {
        emit Deposit(msg.sender, msg.value);
    }
}
```

Let's also create a basic function to return the balance of the contract.

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

interface IERC20 {
    function transfer(address to, uint256 amount) external view returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

contract Faucet {
    address payable owner;
    IERC20 public token;

    uint256 public withdrawalAmount = 50 * (10 ** 18);

    uint256 public lockTime = 1 minutes;

    event Deposit(address from, uint256 amount);

    mapping(address => uint256) nextAccessTime;

    constructor(address tokenAddress) payable {
        token = IERC20(tokenAddress);
        owner = payable(msg.sender);
    }

    function requestTokens() public {
        require(msg.sender != address(0), "Request must not originate from a zero account");
        require(token.balanceOf(address(this)) >= withdrawalAmount, "Insufficient balance");
        require(block.timestamp >= nextAccessTime[msg.sender], "Insufficient time elapsed since last withdrawal");

        nextAccessTime[msg.sender] = block.timestamp + lockTime;

        token.transfer(msg.sender, withdrawalAmount);
    }

    receive() external payable {
        emit Deposit(msg.sender, msg.value);
    }

    function getBalance() external view returns (uint256) {
        return token.balanceOf(address(this));
    }
}
```

Let's create some setter functions for setWithdrawalAmount and setBlockTime and give it a modifier so only the owner can call it.


```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

interface IERC20 {
    function transfer(address to, uint256 amount) external view returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

contract Faucet {
    address payable owner;
    IERC20 public token;

    uint256 public withdrawalAmount = 50 * (10 ** 18);

    uint256 public lockTime = 1 minutes;

    event Deposit(address from, uint256 amount);

    mapping(address => uint256) nextAccessTime;

    constructor(address tokenAddress) payable {
        token = IERC20(tokenAddress);
        owner = payable(msg.sender);
    }

    function requestTokens() public {
        require(msg.sender != address(0), "Request must not originate from a zero account");
        require(token.balanceOf(address(this)) >= withdrawalAmount, "Insufficient balance");
        require(block.timestamp >= nextAccessTime[msg.sender], "Insufficient time elapsed since last withdrawal");

        nextAccessTime[msg.sender] = block.timestamp + lockTime;

        token.transfer(msg.sender, withdrawalAmount);
    }

    receive() external payable {
        emit Deposit(msg.sender, msg.value);
    }

    function getBalance() external view returns (uint256) {
        return token.balanceOf(address(this));
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the contract owner can call this function");
        _;
    }

    function setWithdrawalAmount(uint256 amount) public onlyOwner {
        withdrawalAmount = amount * (10**18);
    }

    function setBlockTime(uint256 amount) public onlyOwner {
        lockTime = amount * 1 minutes;
    }
}
```


Now we need to set up our withdrawal function so the owner of the contract can withdraw their funds anytime.
Then we will also define an event for withdrawal too in the Interface.


```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.17;

interface IERC20 {
    function transfer(address to, uint256 amount) external view returns (bool);
    function balanceOf(address account) external view returns (uint256);
    function Transfer(address from, address to, uint256 value) external;
}

contract Faucet {
    address payable owner;
    IERC20 public token;

    uint256 public withdrawalAmount = 50 * (10 ** 18);

    uint256 public lockTime = 1 minutes;

    event Withdrawal(address indexed to, uint256 indexed amount); //optional   
    event Deposit(address indexed from, uint256 indexed amount);


    mapping(address => uint256) nextAccessTime;

    constructor(address tokenAddress) payable {
        token = IERC20(tokenAddress);
        owner = payable(msg.sender);
    }

    function requestTokens() public {
        require(msg.sender != address(0), "Request must not originate from a zero account");
        require(token.balanceOf(address(this)) >= withdrawalAmount, "Insufficient balance");
        require(block.timestamp >= nextAccessTime[msg.sender], "Insufficient time elapsed since last withdrawal");

        nextAccessTime[msg.sender] = block.timestamp + lockTime;

        token.transfer(msg.sender, withdrawalAmount);
    }

    receive() external payable {
        emit Deposit(msg.sender, msg.value);
    }

    function getBalance() external view returns (uint256) {
        return token.balanceOf(address(this));
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the contract owner can call this function");
        _;
    }

    function setWithdrawalAmount(uint256 amount) public onlyOwner {
        withdrawalAmount = amount * (10**18);
    }

    function setBlockTime(uint256 amount) public onlyOwner {
        lockTime = amount * 1 minutes;
    }

    function withdrawal() external onlyOwner {
        emit Withdrawal(msg.sender, token.balanceOf(address(this)));
        token.transfer(msg.sender, token.balanceOf(address(this)));
    }
}
```

Note that you can pick your interface directly from openzeppelin

Now we'll write a deploy script for our faucet contract..

Under the scripts folder, create a deployFaucet.js file.

Copy the code in deploy.js and paste it here.

Don't forget that faucet.deploy() needs to take in the address from the constructor of the smart contract.

```
const hre = require("hardhat");

async function main() {
  const Faucet = await hre.ethers.getContractFactory("Faucet");
  const faucet = await Faucet.deploy();

  await faucet.deployed();

  console.log("Faucet contract deployed: ", faucet.address);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

First we will deploy our token with ```npx hardhat run --network goerli scripts/deploy.js```

Copy the token deployment address and paste it in the faucet deploy script:



After deployment, copy the faucet contract address, check it on etherscan and do some testing on Remix
