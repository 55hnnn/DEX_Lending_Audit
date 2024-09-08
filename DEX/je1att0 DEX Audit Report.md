# 1. Miss interaction LP Token
## Description
- `Dex.sol:removeLiquidity(L: 49-63)`
`removeLiquidity` 함수는 LP 토큰을 통해 사용자가 공급한 유동성을 되돌려받는 함수입니다. 그러나 해당 함수에서는 사용된 LP 토큰을 회수하지 않아 재사용이 가능합니다. 따라서, 매우 소량의 LP토큰만으로도 유동성 풀의 모든 자산을 탈취할 수 있습니다.
## Impact
### Critical
LP 토큰이 존재하기만 한다면 유동성 풀에 존재하는 모든 자산을 탈취할 수 있습니다. 단순한 함수 호출로 피해가 발생할 수 있으므로, 반드시 수정이 필요해 보입니다.
```solidity
    function test_reuseLPtoken() external {
        uint lp1 = dex.addLiquidity(1000 ether, 1000 ether, 0);
        uint lp2 = dex.addLiquidity(1000 ether * 4, 1000 ether * 4, 0);

        dex.tokenLP().transfer(address(1), lp1);
        vm.startPrank(address(1));
        dex.removeLiquidity(lp1, 0, 0);
        dex.removeLiquidity(lp1, 0, 0);
        dex.removeLiquidity(lp1, 0, 0);
        assertEq(3000 ether, tokenX.balanceOf(address(1)));
        assertEq(3000 ether, tokenY.balanceOf(address(1)));
    }
```
## Recommendation
사용된 LP 토큰은 적절히 처리되어야 합니다.
# 2. Weak Logic Check
## Description
- `Dex.sol:swap(L: 65)`
`swap` 함수는 교환할 토큰을 받고, 또 다른 토큰으로 교환해 주는 함수입니다. 따라서 한 토큰은 0보다 커야하며, 다른 한 토큰은 0이어야 합니다. 그러나 해당 함수에서는 `_amountX`와 `_amountY`의 값 중 하나만 0이기만 하면 함수를 실행시킵니다. 따라서 두 토큰 모두 0일 경우에도 해당 함수가 실행됩니다.
## Impact
### Information
해당 컨트랙트의 자산에 큰 영향을 미치지는 않지만, 불필요한 비즈니스 로직이라고 사료됩니다. 따라서 좀 더 세부적인 조건식이 있다면, 불필요한 함수실행을 줄일 수 있습니다.
## Recommendation
`require((amountX > 0 && amountY == 0) || (amountX == 0 && amountY > 0), "Invalid input amounts");`와 같은 세부적인 조건식으로 업데이트하길 권장드립니다.
# 3. Insufficient Function Modifier
## Description
- `Dex.sol:swapXtoY(L: 74-83)`
- `Dex.sol:swapYtoX(L: 85-94)`
`swapXtoY` 또는 `swapYtoX`함수는 `swap`함수에서 사용되는 내부 로직 함수입니다. 그러나 내부함수가 `public`으로 선언되어있어, 직접적으로 내부함수의 호출이 가능합니다.
`swap`함수는 `_amountX`나 `_amountY`의 값이 모두 0보다 큰 경우를 제외하고 있습니다. 그러나 `swap`함수의 내부 함수는 이를 검증하지 않고 바로 함수를 실행시키므로, 외부에서 실행할 수 없도록 조치를 취하여야 합니다.
## Impact
### low
해당 취약점은 자산에 영향을 미치지 않지만, 개발자가 의도한 동작을 수행하지 않아 조치가 권장됩니다.
## Recommendation
`public` 선언을 `internal`함수로 수정하는 것을 권장합니다.