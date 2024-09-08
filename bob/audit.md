## Critical - 공격자가 USDC를 빌린 후, 직접 발행한 임의의 토큰으로 repay 하여 이익을 취할 수 있습니다.

### 설명

./Lending.sol : 156-162

function repay(address token, uint256 amount) external onlyUser accrueInterest(msg.sender)

```solidity
function repay(address token, uint256 amount) external onlyUser accrueInterest(msg.sender) {
    uint256 total_debt = account[msg.sender].BorrowAmount;
    require(total_debt >= amount, "check amount");
    account[msg.sender].BorrowAmount -= amount;
    TotalBorrowUSDC -= amount;
    ERC20(token).transferFrom(msg.sender, address(this), amount);
}
```
repay하는 토큰이 USDC token인지 확인하지 않고 `ERC20(token).transferFrom(msg.sender, address(this), amount);`로 토큰을 가져온 후 `account[msg.sender].BorrowAmount`를 업데이트 합니다.

### 파급력
공격자가 borrow한 USDC가 실제로 repay되지 않아 contract에 손실을 가져다 줄 수 있으며, 공격자는 직접 발행한 토큰으로 repay하여 기존 USDC로 이익을 취할 수 있습니다.

### 해결 방안
```solidity
function repay(address token, uint256 amount) external onlyUser accrueInterest(msg.sender) {
    uint256 total_debt = account[msg.sender].BorrowAmount;
    require(token == usdc, "only USDC"); // added check
    require(total_debt >= amount, "check amount");
    account[msg.sender].BorrowAmount -= amount;
    TotalBorrowUSDC -= amount;
    ERC20(token).transferFrom(msg.sender, address(this), amount);
}
```
repay로 들어온 token이 USDC 토큰의 주소와 동일한지 확인할 수 있습니다.
