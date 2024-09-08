# 1. Weak Logic Check
## Description
- `Dex.sol:swap(L: 69)`
`swap` 함수는 교환할 토큰을 받고, 또 다른 토큰으로 교환해 주는 함수입니다. 따라서 한 토큰은 0보다 커야하며, 다른 한 토큰은 0이어야 합니다. 그러나 해당 함수에서는 `_amountX`와 `_amountY`의 값 중 하나만 0이기만 하면 함수를 실행시킵니다. 따라서 두 토큰 모두 0일 경우에도 해당 함수가 실행됩니다.
## Impact
### Information
해당 컨트랙트의 자산에 큰 영향을 미치지는 않지만, 불필요한 비즈니스 로직이라고 사료됩니다. 따라서 좀 더 세부적인 조건식이 있다면, 불필요한 함수실행을 줄일 수 있습니다.
## Recommendation
`require((amountX > 0 && amountY == 0) || (amountX == 0 && amountY > 0), "Invalid input amounts");`와 같은 세부적인 조건식으로 업데이트하길 권장드립니다.
# 2. Non-update reserve
## Description
- `Dex.sol:removeLiquidity(L: 59-60)`
해당 컨트랙트의 `reserve` 변수는 DEX가 보유하고있는 token의 현재 유동성을 나타내며, 토큰의 변화량을 측정하기 위해 사용됩니다. `removeLiquidity`함수는 `reserve`변수를 업데이트하지 않고 그대로 사용합니다. LP토큰의 보유량에 따라 DEX 유동성 풀에 존재하는 자산을 출금받을 수 있어야 하지만, `reserve` 변수가 업데이트되지 않아 적절한 보상을 받을 수 없게 됩니다.
## Impact
### medium
만약 누군가가 dex에게 토큰을 보낸다면, 유동성 비율은 영향을 받게 될것입니다. 이때 보유한 lp 토큰으로 DEX에 유동성을 공급받으면, 누군가가 보낸 토큰도 일부분 반환받아야 합니다. 그러나 해당 컨트랙트에서는 `reserve`를 업데이트하지 않고 유동성을 공급하므로, 사용자에게 부적절한 유동성을 공급하게 됩니다.
```solidity
    function test_effectReserveUpdate() external {
        uint lp1 = dex.addLiquidity(1000 ether, 1000 ether, 0);
        (uint x, uint y) = dex.removeLiquidity(1 ether, 0, 0); 
        console.log(x, y); // 1000000000000000000 1000000000000000000
        tokenX.transfer(address(dex), 1 ether);
        (x, y) = dex.removeLiquidity(1 ether, 0, 0);
        console.log(x, y); // 1000000000000000000 1000000000000000000
    }
```
## Recommendation
함수 실행 이후에 `reserve` 변수를 업데이트하는 것을 권장합니다.