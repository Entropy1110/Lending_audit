## Medium - Contract의 USDC 토큰의 주소를 변경하여, 가용성을 해칠 수 있습니다.

### 설명

./DreamAcademyLending.sol : 64-68

function initializeLendingProtocol(address addr) external payable

```solidity
function initializeLendingProtocol(address addr) external payable {
    usdc = IERC20(addr);
    usdc.transferFrom(msg.sender, address(this), msg.value);
}
```

Lending contract의 owner를 검사하지 않아 악의적인 사용자가 initializeLendingProtocol 함수 호출을 통해 Contract의 usdc address를 변경할 수 있습니다.

### 파급력

usdc의 주소를 공격자가 임의로 발행한 ERC20 토큰의 주소로 변경할 경우, 사용자들의 contract에 대한 가용성이 저하되며, 다시 정상적인 USDC 주소로 initializeLendingProtocol를 호출하여야 합니다.

### 해결 방안

Ownable library를 import하여 해당 함수에 관리자만 접근할 수 있도록 제한합니다. 또는 한번 initialized 된 경우 다시 initialize하지 못하도록 제한하는 것이 좋습니다.

## Critical - 공격자가 임의의 토큰을 deposit하여, 더 많은 양의 토큰을 빌리거나 인출할 수 있습니다.

### 설명

./DreamAcademyLending.sol : 69-84

function deposit(address token, uint256 amount) public payable

```solidity
function deposit(address token, uint256 amount) public payable {
    if (token == address(0x0)) { 
        require(msg.value == amount, "Invalid ETH amount");
        depositETH[msg.sender].amount += msg.value; // Ether를 담보로 예치
        depositETH[msg.sender].bnum = block.number; 
    } else {
        require(amount > 0, "Invalid usdc amount");
        if(depositUSDC[msg.sender]==0){ //새로운 유저의 deposit
            users.push(msg.sender);
        }
        depositUSDC[msg.sender] += amount;
        usdctotal += amount;
        console.log("total", usdctotal);
        IERC20(token).transferFrom(msg.sender, address(this), amount);
    }
}
```

deposit 함수에서 `address token`이 USDC token의 address와 같은지 검사하거나, 인자와 상관없이 USDC token을 trasferFrom 하는 logic이 구현되어있지 않습니다.


### 파급력
공격자가 임의의 ERC20 토큰을 발행해, deposit할 경우에도 공격자의 잔고가 업데이트 되므로, 실제 가진 것보다 많은 양의 USDC 토큰을 인출하거나, 실제로 빌릴 수 있는 것보다 많은 양의 Ether를 빌릴 수 있습니다.
