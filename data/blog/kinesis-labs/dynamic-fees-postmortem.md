---
title: Dynamic Fees
date: '2023-01-26'
tags: ['kinesis', 'crypto', 'reflections', 'defi']
draft: false
summary: 'A postmortem of the security exploit in the Kinesis Contracts.'
---

# Kinesis Dynamic Fees Postmortem

Contact Information

Telegram(Preferred): @kinesislabs

Email: team@kinesislabs.co

Discord: Dr.Kinesis#0157

# Key Points

- All user funds are safe
- The issue here is incorrect arthimetic leading to safemath reverting when attemping to swap tokens, meaning swaps are disabled. However, users can still unstake and withdraw from the pools.
- The plan is to set `rewardsPerSecond` to 0 on the three current Simple Rewarders, send the rest of the staking rewards to the new SimpleRewarders, and ask users to migrate their funds to the new SimpleRewarders

### The Issue

We initially got a report from multiple users in discord that they were having trouble swapping tokens in the new DAI/USDC/USDT pools. We initially chalked these up to RPC issues, which are not abnormal with the current Evmos infrastructure. However, when we got the following report from GV of Swiss Staking, we knew something was probably wrong as a power user like GV would know to change the RPCs and to try the transaction again after waiting a few minutes.

![GV's message](./../../../public/static/images/kinesis/gv_swiss_staking.png)

We then tested the exact swap ourselves, axlDAI for ibc G-DAI and got the following error.

![Swap Error Message](./../../../public/static/images/kinesis/swap-error-message.png)

The error is `execution reverted: SafeMath: division by zero`. This was a very bad sign and we immediately knew something went very badly wrong with the contracts - this error could no longer be from Evmos infrastructure or our frontend.

After triaging this a bit further, we found the exact line in the frontend that made this call to the `swap()` function and thus we knew the issue was in the `swap()` function of the contracts.

The swap function is very simple, just two lines:

```
    function swap(
        uint8 tokenIndexFrom,
        uint8 tokenIndexTo,
        uint256 dx,
        uint256 minDy,
        uint256 deadline
    )
        external
        payable
        virtual
        nonReentrant
        whenNotPaused
        deadlineCheck(deadline)
        returns (uint256)
    \{
        updateSwapFee(minDy);
        return swapStorage.swap(tokenIndexFrom, tokenIndexTo, dx, minDy);
    \}
```

There were only three functions that could have held this error, the `deadlineCheck()` modifier, the `updateSwapFee()` function, or the `swapStorage.swap()` function.

The `deadlineCheck()` does not have a SafeMath div call, meaning the error could not have been there. Additionally, the SafeMath divs in `swapStorage.swap()` both use constants, meaning it could not be those functions either.

This narrows down the scope of the issue to the `updateSwapFee()` function. For full disclosure, this is a newly added function, which dynamically changes the swap fee based on the volume swapped in the last hour. There is one SafeMath div in the function.

```
uint256 newSwapFee =
(feeFactor.mul(hourlyVolume))
.div(averageVolume).add(baseFee);
```

Since the error was `execution reverted: SafeMath: division by zero`, we know that averageVolume is zero, making the entire `swap()` function revert.

Average Volume is set in [Line 527](https://github.com/kinesis-labs/contract/blob/69c3449b1026bc217f12f7ac5f1f7ee14656ec0a/contracts/Swap.sol#L527) and we can quickly see the error.

```
averageVolume = averageVolume
.add(averageVolume.sub(recentVolume[recentVolumePointer].volume))
.mul((10 minutes / (block.timestamp.sub(creationTimestamp))));
```

10 minutes after the contract is created, `block.timestamp.sub(creationTimestamp)` will be greater than `10 minutes`. In solidity, there are no decimals, so `1/2` will round to zero. Thus, `(10 minutes/block.timestamp.sub(creationTimestamp))` will round to zero after the first 10 minutes, meaning that Average Volume will always be set to zero, and thus the `swap()` function will always revert.

## Triaging the Issue

The first thing we tested was withdrawing liquidity as we want to be 100% sure that user funds were not at risk. To this effect, we tested Deposit, Withdraw, Stake All, and Unstake All to ensure there are no issues with any of these functionality. Luckily, there were no issues here and USER FUNDS ARE SAFE.

Thus, there are a few issues to keep in mind. We want Kinesis to be a useful application on Evmos instead of a place to stake stablecoins to receive Evmos. Thus, we would need to either upgrade the contract or to deploy new contracts. Unfortunately, we do not have upgrade functionality to violate the core principles of decentralization and immutability.

We then decided the best way forward would be to redeploy the contracts and send the rest of the rewards(87,343.52 wEVMOS) to the new simple rewarders.

## Next Steps

### Redeploying Contracts

Contracts have already been redeployed to these addresses:

```
USDCPool: 0x35bF604084FBE407996c394D3558E58c90281000
USDCPoolLPToken: 0xfD2fd675176a8Ed1CF643886ee557929FDEcBBfD
USDTPool: 0x89E9703309DA4aC51C739D7d674F91489830310E
USDTPoolLPToken: 0x78549EF94dB08E8bf2e528F0aE97F186Fc51185E
DAIPool: 0x155377C4f5489026cD8340fF350ae6aa082FBE69
DAIPoolLPToken: 0x9607FFD92DC6846F913129d8351a848240BEC4E9
MiniChefV2: 0x82a4A94eC86f03bf1a5979B41e2Ae8478C93f9C3
SimpleRewarder_usdc: 0x5Ff26712e2Ec975d7F25a31Ba7c9FbA395655076
SimpleRewarder_usdt: 0xDc85D4f941B8adc7cF223C9AFd5305c2684D7090
SimpleRewarder_dai: 0x0BCC8b2E95e0e59a23fc6D8DD653636047cf3C3f
```

The frontend has also been redeployed here with the new pools and the old pools.
https://frontend-git-a-featpostmortem-kinesislabs.vercel.app/#/

### Sending Rewards

Afterwards, we need sent the rest of the wEVMOS liquidity mining rewards to the new Simple Rewarders. This process would be the exact same as what LPX already has done in the past.

- LPX, can you initiate the transactions for this when we have agreed to a path forward?

### Public Announcement to Migrate Funds

We will need to make a public announcement for users to migrate funds from the old pools which will be depreceated to the new pools. There are two options here:

- We can admit we messed up and explain the full scope of the issue
- We can hide this as a routine upgrade to activate dynamic fees.

We are leaning towards the former reason, however, if others in the Evmos ecosystem wish to use the latter reasoning to keep confidence in the ecosystem, that is understandable as well.

### Set Rewards Per Second to Zero

Finally, we would set rewards per second on the current SimpleRewarder Contracts. This is to ensure everyone can unstake their LP tokens from Minichef such that they can withdraw their LP tokens, as if there are not enough LP tokens on Minichef, users will no longer be able to unstake their LP tokens.

- LPX, can you initiate these three transactions? You would need to call
-

```
function setRewardPerSecond(uint256 _rewardPerSecond) public onlyOwner \{
        rewardPerSecond = _rewardPerSecond;
        emit LogRewardPerSecond(_rewardPerSecond);
    \}
```

in SimpleRewarder.sol with `_rewardPerSecond` set to 0.

### TODOs on Kinesis' End

Update Minichef to the new minichef contract address on the frontend when rewards are live on the new contracts
