## Medium - Contract의 totalEthSupply값을 1로 변경하여, contract의 가용성을 해칠 수 있습니다.


### 설명
./DreamAcademyLending.sol : 225-231
function initializeLendingProtocol(address token) external payable
```solidity
function initializeLendingProtocol(address token) external payable {
    require(totalEthSupply == 0 && totalUsdcSupply == 0, "Already initialized");
    require(token == usdcToken, "Unsupported token");
    require(msg.value == 1, "Incorrect ETH amount");

    totalEthSupply = 1;
}
```
initializeLendingProtocol 실행을 통해 totalEthSupply의 값을 1로 변경할 수 있습니다.


### 파급력
해당 함수가 매번 실행되면서 totalEthSupply의 값이 1이 되고, liquidate 시에 underflow 발생으로 contract의 가용성을 해칠 수 있습니다.


### 해결 방안
contract owner만 `initializeLendingProtocol`을 실행하도록 변경하거나, 한번만 실행되도록 변경합니다.
