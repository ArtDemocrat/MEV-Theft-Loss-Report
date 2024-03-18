[ :arrows_counterclockwise: WIP!!!!]

# MEV Loss Tracker

### Analysis Scope
The goal of this research paper is to: 
* Report on the state of MEV theft for the past 65 weeks.
* Evaluate the need for creating tools to analyze the performance of Rocket Pool validators in collecting priority fees and MEV.
* If the need is there, create reporting mechanisms and tools to surface MEV theft. 

This research is produced for the Rocket Pool GMC [Retroactive Grant XXXX], and as a continuation of Bounty [BA032304](https://dao.rocketpool.net/t/july-2023-gmc-call-for-bounty-applications-deadline-is-july-15th/1936/6).

**For reference and full credits:** The scripts used to generate the raw data analyzed below were developed Ramana's and Valdorff's initial analysis of MEVTheft in the RP Protocol,  ["RocketTheft"](https://github.com/xrchz/rockettheft). The scripts used for the analysis of the data, as well as the proposed tools and mechanisms to track MEV theft within the Rocketpool Protocol, are designed and produced by ArtDemocrat. 

Following the same logic as in the original RocketTheft analysis, we start high level and then go specific. This analysis covers 65 weeks of ethereum slots. It starts right after the MEV grace period ended at slot 5203679 (2022-11-24 05:35:39Z UTC; see https://discord.com/channels/405159462932971535/405163979141545995/1044108182513012796), and ends at slot 8500000-1 (2024-02-25 01:20:11 UTC). We will name this set of datapoints "the entire distribution" in this analysis. 

## Rocketpool vs Non-Rocketpool Maximum Bid (Ξ) Consistency Check 

We start by evaluating whether Rocketpool ("RP") is being consistently lucky or unlucky against the non-RP Ethereum validating cohort when it comes to the maximum bids received by ethereum relayers. We do this by plotting a cumulative distribution function ("CDF") for the maximum bids on all Ethereum slots (blue dots/line) and another one for RP blocks (orange dots/line).  Besides doing a visual evaluation for each of the cohorts, we apply the Kolmogorov-Smirnov (K-S) statistical evaluation on the entire distribution, and on subsets of the entire distribution, in order to compare RP vs non-RP maximum bids distribution (see table below).

The Kolmogorov-Smirnov (K-S) test is a non-parametric test that compares two samples to see if they come from the same distribution. It's useful in this case because it doesn't assume any specific distribution of the data and is sensitive to differences in both location and shape of the empirical cumulative distribution functions of the two samples analyzed. The K-S test returns a D statistic and a p-value. The D statistic represents the maximum difference between the two cumulative distributions, and the p-value tells us the probability of observing the data assuming the null hypothesis (i.e. that the samples are from the same distribution) is true. 

* **K-S statistic (D)**: The greater this value (closer to 1.0), the larger the maximum difference between the CDFs, suggesting a greater discrepancy between the two groups. The lower this value (closer to 0.0), the more the distributions of the two samples are similar or the same.
* **p-value**: A small p-value (typically ≤ 0.05) suggests that the samples come from different distributions. If this value is less than or equal to 0.05, the difference in distributions is considered statistically significant, meaning it's unlikely the difference is due to random chance.

## Consistency Check - Global Conclusion
[Analysis Script](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/generate_slot_reward_distro_sections)

If we take a look at the entire distribution, **we see no evidence that RP gets better or worse bids vs non-RP validators.**
* Total number of rows being plotted between 0.001 ETH and 1000 ETH: 3,309,302
* Number of 'Is RocketPool: TRUE' datapoints: 90,220
* Number of 'Is RocketPool: FALSE' datapoints: 3,219,082
* :white_check_mark: K-S statistic: 0.0027206808206513555
* :white_check_mark: p-value: 0.5335313853854944

<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/b2596212-75bb-4eb9-925e-b929efa3188c" width="1000" height="600">

## Consistency Check - Cohort Breakdown Conclusion
If we break this analysis down to specific maximum bid ranges, we do see discrepancies between the RP and non-RP cohorts, specifically in very low and very high maximum bid ranges (where RP data becomes scarce). For the purpose of this document we will treat both datasets as similar (i.e. both RP and non-RP cohorts have the same "luck" when it comes to receiving maximum bids from Rocketpool-approved relays).

| **Metric / Range** | **0.001-0.01 ETH**                     | **0.01-0.1 ETH**                    | **0.1-1 ETH**                       | **1-10 ETH**                        | **10-1000 ETH**                     |
|     :---           |     ---:                               |     ---:                            |     ---:                            |     ---:                            |     ---:                            |
| # of Slots:        | 9,358                                  | 2,392,861                           | 862,936                             | 42,240                              | 1,914                               |
| # of RP Slots:     | 287                                    | 65,229                              | 23,564                              | 1,086                               | 54                                  |
| # of non-RP Slots: | 9,071                                  | 2,327,632                           | 839,372                             | 41,154                              | 1,860                               |
| K-S statistic:     | :warning: 0.029132930036640872         | :white_check_mark: 0.00389371092174 | :white_check_mark: 0.00865618477157 | :warning: 0.02224900031870          | :warning: 0.09510155316606          |
| p-value:           | :white_check_mark: 0.966603108312941   | :white_check_mark: 0.29039831890954 | :white_check_mark: 0.06408145819303 | :white_check_mark: 0.66300720498166 | :white_check_mark: 0.69313030426547 |

<p align="center">
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/1413cf07-f26d-4be2-b9bd-aadfb85c43b3" width="500" height="300"><img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/d3ebf9b7-1013-4a74-a47a-9a5f836423a4" width="500" height="300">
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/84d8c5a1-4a34-4d84-a63e-dfc2a21e685d" width="500" height="300"><img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/f70cb0d7-1106-4c2f-9923-70c4522f5d80" width="500" height="300">
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/6f75ec2e-d497-488b-af41-d84c47bb0965" width="500" height="300">
</p>

## Systematic Loss Analysis
In order to analyze MEV loss cases we define 3 types of losses:
1. **Theft**: the fee recipient for a block (according to either the relay's payload if mev_reward is present, or the Beacon chain otherwise) was incorrect. This happens when the fee recipient is not set to either the smoothing pool ("SP") if a node is opted-in the SP, or the node's fee recipient otherwise.
2. **Neglect**: the node accepts a vanilla block, losing profits against a scenario where MEV-boost would have been used.
3. A combination of **Neglect** + **Theft** or viceversa.

### Theft
[Analysis Script](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/generate_mevreward_theft)
[Analysis Script - Details](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/generate_mevtheft_details)

First we begin by plotting the MEV rewards of each slot where we deemed the fee recepient for a proposed block as incorrect. In the data series we are studying (slots 5203679 to 85000000-1), 51 cases of MEV Theft ocurred. If we analyze these cases we can see that the smoothing pool is slightly more affected (39 theft cases) vs non-opt-in validators (12 theft cases). This derived in a total loss of 6.29 ETH for the rocketpool protocol, split as shown below:

* Total number of rows being plotted between 0.001 ETH and 1000 ETH: 51
* Number of 'In Smoothing Pool: TRUE' datapoints: 39
* Number of 'In Smoothing Pool: FALSE' datapoints: 12
* Total ETH theft in smoothing pool: 4.361759122909182 ETH
* Total ETH theft outside of smoothing pool: 1.928289059541783 ETH
* K-S statistic: 0.2564102564102564
* p-value: 0.5039535766788686

In the first chart below we plot the cases of theft in the smoothing pool vs the non-opt-in cases. In the second chart below we plot the magnitude of the stolen MEV reward (Y axis) and the slot where it happened (X axis), separating between smoothing pool and non-opt-in cases. 
**Conclusion:** While 51 theft cases out of 90,220 Rocketpool block proposals analyzed in this time series represent a low incidence of 0.06%, it seems that theft is a phenomenon which is happening continuously within the protocol. MEV theft incidence seems to have become more prevalent in recent slots.
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/861d8e80-90d1-45e7-bd4e-a63fbcd97aa5" width="1000" height="600">
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/ac5e3d5f-54fb-4f0b-8c40-582dc4fc95c7" width="1000" height="600">


In the first table below we display the ranking of repeated offenders, and the value which offenders have taken from the Rocketpool protocol. In the second table below we display the details of the slots where theft happened.

<p align="center">
  <img width="500" height="400" src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/9498c847-ce4c-4107-9ffe-1507c84c4dda">
</p>

*The largest MEV reward channeled to an incorrect fee recipient happened in slot 6376024 and was due a configuration error after a solo migration took place. The Node Operator immediately sent the correct amount to the smoothing pool (see https://etherscan.io/tx/0x18a28f9bba987a05bc87515faa6490cef3fe61b02dc45d68cffcf3a4e6f791a0).

<p align="center">
  <img width="620" height="1000" src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/4eecc678-78c8-44d6-b784-273e33d037c3">
</p>


### Neglect

[Analysis Script](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/generate_mevreward_neglect)
The second case of revenue loss for the RP protocol is where validators do not choose maximize the MEV rewards made available for them by relayers. This happens when a RP validator does not register with any MEV relayer and produces so called "vanilla blocks", which don't follow the transaction-ordering reward-maximizing logic which MEV searchers, builders, and relayers pass on to validators. This is a complex matter to quantify since we cannot always asses with 100% certainty which validator is leveraging MEVboost, from which relayer, and to which extent. The reasons for this are:

1. Some relayers don't always make their MEV bid data available to the public, which could cause a wrong classification of vanilla blocks while these blocks actually had a bid from a relayer. For the scope of this report, we simply classify slots with no RP-approved relayer in our dataset (see specifics around the underlying dataset [here](https://github.com/xrchz/rockettheft/blob/main/README.md#data-notes)) as vanilla blocks. 
2. Secondly, "bloXroute ethical" (a RP-approved relayer) sunset its API, which prevented us from using it correctly even if it is classified as a RP-approved relayer. In order to mitigate this, RP vanilla blocks are checked to see if they had relay rewards according to beaconcha.in. If so, they are simply dropped from the dataset to avoid polluting the vanilla block category with known MEV. Also, if there are multiple relayers which provide the same block, since it is not possible to know which relay was actually used, such a slot is removed from this list. The estimation of the original bounty puts these "dropped" blocks at 19% of all vanilla blocks, which is a material percentage. See "Next Steps" section below for details on how we plan to mitigate this and other data quality issues in future iterations of this report.
3. If a validator is not registered with any MEV relayer in a slot, no max_bid will be visible in the dataset. This is an issue is solved by using the average of the 3 slots before and after the missing max_bid block to calculate the amount of ETH neglected. See "Next Steps" section below for details on how we plan to mitigate this and other data quality issues in future iterations of this report.
4. See other important data caveats for the underlying dataset [here](https://github.com/xrchz/rockettheft/blob/main/README.md#data-notes).

While it is important to consider the aforementioned data quality issues we face when it comes to neglected ETH rewards, and it is something which should be addressed in future iterations of MEV Loss reporting for RP (see "Next Steps" section below), we are still able to gather solid datapoints to quantify how the neglect issue has evolved in the recent months of operation. Specifically, within the dataset analyzed, we see that:

- Vanilla blocks have been proposed in 6,651 slots (3,3k SP operators and 3,3k non-opted-in operators).
- This leads to a total loss revenue of 620.4 ETH for RP (280.6 ETH loss for the SP, and 339.8 loss for rETH holders). This would represent a 5 basis point ("bps" - i.e. if APR is 1%, it would increase to 1.05%) APR improvement on the current 1.12M ETH staked in the beacon chain by RP (Status 2024-03-18, [source](https://dune.com/drworm/rocketpool)).
- There is a second level loss which comes from accepting an MEV bid which is lower than the max_bid registerd for a node operator in a particular slot. This can be due to several reasons which typically cannot be influenced by RP (such as a validator avoiding unregulated relayers, see details [here](https://docs.rocketpool.net/guides/node/mev#block-builders-and-relays)). However, it is worth noting that if we simply observe the sum of mev_reward captured by RP validators in the analyzed period (9,997.4 ETH) and compare it with the sum of max_bids which could have theoretically been captured by RP validators (11.729.0 ETH), capturing that difference (1.731,6 ETH) would derive on a 15bps APR improvement on RP's current staked capital. In case we are overlooking drivers which are actionable by RP in order to close the gap of MEV capture vs the theoretical maximum, we would appreciate the community's input on this in this research's retroactive grant posted in the RO [governance forum](XXXX).

The chart below plots the distribution of the 6,651 slots where vanilla blocks were proposed by RP validators. We see a random distribution which tends to become less prevalent towards recent slots (potentially due to the protocol moving MEV Capture Phase 2 "Opt-out" after November 2022). One recommendation of this report is to move to MEV capture Phase 3 "Required" as soon as possible, in order to minimize the losses aforementioned losses due to vanilla block proposals. The selection of regulated vs non-regulated relayer usage should be defined entirely, needless to say, by each node operator's preferences.

<p align="center">
  <img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/5f418ce1-b335-468f-b917-82edda57b792">
</p>

Analysis Data:
```
Number of 'In Smoothing Pool: TRUE' datapoints: 3362
Number of 'In Smoothing Pool: FALSE' datapoints: 3289
Total ETH neglect in smoothing pool: 280.5999492060128
Total ETH neglect outside of smoothing pool: 339.7532293889865
Total ETH rewards accepted (all Rocketpool): 9997.413153367683
Total ETH rewards offered (all Rocketpool): 11729.025782041706
% of MEV-neglect slots within Rocketpool: 7.62%
% of MEV-neglect slots outside of Rocketpool: 14.96%
```

It is worth mentioning that within the datapoints shown above, the % of MEV-neglected slots (i.e. no relayer was observed in a slot with a successfuly proposed block) is double as high in non-RP proposals (15% non-RP vs 7.6% RP). 

## Next Steps [ :arrows_counterclockwise: WIP!!!!]
