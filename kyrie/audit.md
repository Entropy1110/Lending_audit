## Critical - 공격자가 임의의 토큰을 deposit하여, USDC 토큰을 인출할 수 있습니다.

### 설명
./DreamAcademyLending.sol : 41-44

function initializeLendingProtocol(address _usdc) external payable
```solidity
function initializeLendingProtocol(address _usdc) external payable {
    usdc = ERC20(_usdc);
    usdc.transferFrom(msg.sender, address(this), msg.value);
}
```
initializeLendingProtocol 함수를 통해 contract 내 usdc 토큰의 주소를 변경할 수 있습니다.


./DreamAcademyLending.sol : 63-67

function deposit(address token, uint256 amount) external payable
```solidity
function deposit(address token, uint256 amount) external payable {
    require(amount != 0, "Invalid amount");

    if (token == address(0x0)) {
        require(msg.value != 0, "Invalid amount");
        require(msg.value >= amount, "Invalid amount");
        _depositETHs[msg.sender] += amount;
    } else {
        require(
            usdc.allowance(msg.sender, address(this)) >= amount,
            "ERC20: insufficient allowance"
        );
        require(
            usdc.balanceOf(msg.sender) >= amount,
            "ERC20: transfer amount exceeds balance"
        );

        usdc.transferFrom(msg.sender, address(this), amount);
        _depositUSDCs[msg.sender].amount += amount;
        totalUSDC += amount;

        lenders.push(msg.sender);
    }
}
```
deposit 함수를 통해, 공격자가 변경한 임의의 usdc 토큰을 deposit할 수 있습니다. 이를 통해, 실제 USDC 토큰이 아닌 임의의 토큰으로 공격자의 _depositUSDCs.amount 값을 증가시킬 수 있습니다.
이후 initializeLendingProtocol로 USDC 토큰의 주소를 설정하고 withdraw 함수를 통해, 공격자의 _depositUSDCs.amount를 참고해 USDC 토큰을 인출할 수 있습니다.


### 파급력
공격자가 직접 발행한 임의의 토큰으로 contract 내의 USDC 토큰을 인출할 수 있습니다.


### 해결 방안
initializeLendingProtocol의 접근 권한을 contract owner로 제한하거나, 단 한번만 실행가능하도록 제한합니다.



## Medium - Contract의 USDC 토큰의 주소를 변경하여, 가용성을 해칠 수 있습니다.

### 설명

./DreamAcademyLending.sol : 41-44

function initializeLendingProtocol(address _usdc) external payable

```solidity
function initializeLendingProtocol(address _usdc) external payable {
    usdc = ERC20(_usdc);
    usdc.transferFrom(msg.sender, address(this), msg.value);
}
```

Lending contract의 owner를 검사하지 않아 악의적인 사용자가 initializeLendingProtocol 함수 호출을 통해 Contract의 usdc address를 변경할 수 있습니다.

### 파급력

usdc의 주소를 공격자가 임의로 발행한 ERC20 토큰의 주소로 변경할 경우, 사용자들의 contract에 대한 가용성이 저하되며, 다시 정상적인 USDC 주소로 initializeLendingProtocol를 호출하여야 합니다.

### 해결 방안

Ownable library를 import하여 해당 함수에 관리자만 접근할 수 있도록 제한합니다. 또는 한번 initialized 된 경우 다시 initialize하지 못하도록 제한하는 것이 좋습니다.



