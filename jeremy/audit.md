## Critical - 공격자가 빌린 토큰을 임의의 토큰으로 repay하여 이익을 취할 수 있습니다.

### 설명
./src/UpsideAcademyLending.sol : 
function repay(address token, uint256 tokenAmount) public payable

```solidity
function repay(address token, uint256 tokenAmount) public payable {
    require(usdc.allowance(msg.sender, address(this)) > tokenAmount, "");
    tokenAccount[msg.sender] -= tokenAmount; // 갚는 토큰이니 사실상 같은 값을 갚는다.

    uint256 returnEth = (tokenAmount * 1 ether) / (oracle.getPrice(address(0x0)));

    if(block.number - borrowBlock[msg.sender] > 0) {
        returnEth = (returnEth * 1999 * (block.number - borrowBlock[msg.sender])) / (2000 * (block.number - borrowBlock[msg.sender]));
    }

    accounts[msg.sender] += returnEth; // 여기에 이율 곱해서 돌려받아야함

    IERC20(token).transferFrom(msg.sender, address(this), tokenAmount);
}

```
repay 내에, `address token`의 주소가 실제 repay되어야 할 주소와 같은지 확인하는 logic이 존재하지 않으며, 인자로 들어온 `address token`을 그대로 transferFrom 합니다.



### 파급력
공격자가 토큰을 빌린 후, 공격자가 직접 발행한 ERC20 토큰의 주소로 repay할 경우, 공격자가 빌린 토큰을 그대로 소유할 수 있게 됩니다.


### 해결 방안
repay 함수 내에, `address token`의 값이 USDC token의 주소와 동일한지 검증 과정을 추가합니다.
