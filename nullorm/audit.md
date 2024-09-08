## Critical - 공격자가 USDC를 빌린 후, 직접 발행한 임의의 토큰으로 repay 하여 이익을 취할 수 있습니다.

### 설명

./DreamAcademyLending.sol : 114-128

function repay(address _token, uint256 _amount) external

```solidity
function repay(address _token, uint256 _amount) external {
    _debtUpdate(msg.sender);
    User memory user = user_info[msg.sender];

    require(user.borrow_usdc >= _amount, "EXCEEDS_AMOUNT_TOKEN_REPAY");
    require(IERC20(_token).balanceOf(msg.sender) >= _amount, "USER_INSUFFICIENT_TOKEN");
    require(IERC20(_token).allowance(msg.sender, address(this)) >= _amount, "INSUFFICIENT_ALLOWANCE");
    
    IERC20(_token).transferFrom(msg.sender, address(this), _amount);
    user.borrow_usdc -= _amount;
    total_borrowed -= _amount;

    user_info[msg.sender] = user;
    _depositUpdate();
}
```
repay하는 토큰이 USDC token인지 확인하지 않고 `IERC20(_token).transferFrom(msg.sender, address(this), _amount);`로 토큰을 가져온 후 `user.borrow_usdc`를 업데이트 합니다.

### 파급력
공격자가 borrow한 USDC가 실제로 repay되지 않아 contract에 손실을 가져다 줄 수 있으며, 공격자는 이익을 취할 수 있습니다.

### 해결 방안
```solidity
function repay(address _token, uint256 _amount) external {
    _debtUpdate(msg.sender);
    User memory user = user_info[msg.sender];
    require(_token == usdc, "ONLY_USDC_REPAYABLE"); //added check
    require(user.borrow_usdc >= _amount, "EXCEEDS_AMOUNT_TOKEN_REPAY");
    require(IERC20(_token).balanceOf(msg.sender) >= _amount, "USER_INSUFFICIENT_TOKEN");
    require(IERC20(_token).allowance(msg.sender, address(this)) >= _amount, "INSUFFICIENT_ALLOWANCE");
    
    IERC20(_token).transferFrom(msg.sender, address(this), _amount);
    user.borrow_usdc -= _amount;
    total_borrowed -= _amount;

    user_info[msg.sender] = user;
    _depositUpdate();
}
```
repay로 들어온 _token이 USDC 토큰의 주소와 동일한지 확인할 수 있습니다.

