# 1. Insufficient Function Modifier
## Description
- `Dex.sol:update(L: 138-142)`
- `Dex.sol:mint(L: 94-112)`
`update`함수는 자산이 변하는 함수가 실행된 이후 자산의 변화를 적용시켜주기 위한 내부함수로 사용되어야 합니다. 그러나 해당 함수는 `public`으로 구현되어있으며, 다른 조건을 검사하지 않고, 인자로 넘어온 값들을 그대로 반영시킵니다. 

`mint` 함수는 `addLiquidity`함수의 내부 함수로 사용되며, 사용자가 전달한 토큰과 기존 컨트랙트에 존재하였던 토큰의 양을 계산하여 LP 토큰을 발행시켜줍니다. 그러나 이 함수는 external로 구현되어 있어, `addLiquidity`함수에서의 조건식에 구애받지 않고 함수실행이 가능합니다.

이를 통해 LP 토큰을 무제한으로 발행시킬 수 있습니다.
## Impact
### Critical
LP 토큰을 발행할 때, 현재 컨트랙트의 유동성을 계산하는 `reserve`변수를 참조하여 계산합니다. 그러나 `update`함수를 이용하여 이 값을 조작할 수 있고, 조작된 값을 통해 어떠한 비용 없이 현재 유동성 풀에 대한 LP토큰을 발행할 수 있습니다.
```solidity
    function test_mintWithoutAnything() external {
        uint lp1 = dex.addLiquidity(1000 ether, 1000 ether, 0);
        dex.update(1, 1);
        // uint lp2 = dex.addLiquidity(1 ether, 1 ether, 0);
        uint lp2 = dex.mint(address(this));
        console.log(lp1); // 1000000000000000000000
        console.log(lp2); // 999999999999999999999000000000000000000000
    }
```
## Recommendation
내부함수가 외부에서 호출될 수 없도록 적절한 접근제어자가 권장됩니다.