[ :arrows_counterclockwise: WIP!!!!]

# MEV Loss Tracker

### Analysis Scope
The goal of this research paper is to: 
* Report on the state of MEV theft for the past 65 weeks.
* Evaluate the need for creating tools to analyze the performance of Rocket Pool validators in collecting priority fees and MEV.
* If the need is there, create reporting mechanisms and tools to surface MEV theft. 

This research is produced for the Rocket Pool GMC's Bounty [XXXX], and as a continuation of Bounty [BA032304](https://dao.rocketpool.net/t/july-2023-gmc-call-for-bounty-applications-deadline-is-july-15th/1936/6).

**For reference and full credits:** The scripts used to generate the raw data analyzed below were developed Ramana's and Valdorff's initial analysis of MEVTheft in the RP Protocol,  ["RocketTheft"](https://github.com/xrchz/rockettheft). The scripts used for the analysis of the data, as well as the proposed tools and mechanisms to track MEV theft within the Rocketpool Protocol, are designed and produced by ArtDemocrat. 

Following the same logic as in the original RocketTheft analysis, we start high level and then go specific. This analysis covers 65 weeks of ethereum slots. It starts right after the MEV grace period ended at slot 5203679 (2022-11-24 05:35:39Z UTC; see https://discord.com/channels/405159462932971535/405163979141545995/1044108182513012796), and ends at slot 8500000-1 (2024-02-25 01:20:11 UTC). We will name this set of datapoints "the entire distribution" in this analysis. 

## Rocketpool vs Non-Rocketpool Maximum Bid (Ξ) Consistency Check 

We start by evaluating whether Rocketpool ("RP") is being consistently lucky or unlucky against the non-RP Ethereum validating cohort when it comes to the maximum bids received by ethereum relayers. We do this by plotting a cumulative distribution function ("CDF") for the maximum bids on all Ethereum slots (blue dots/line) and another one for RP blocks (orange dots/line).  Besides doing a visual evaluation for each of the cohorts, we apply the Kolmogorov-Smirnov (K-S) statistical evaluation on the entire distribution, and on subsets of the entire distribution, in order to compare RP vs non-RP maximum bids distribution (see table below).

The Kolmogorov-Smirnov (K-S) test is a non-parametric test that compares two samples to see if they come from the same distribution. It's useful in this case because it doesn't assume any specific distribution of the data and is sensitive to differences in both location and shape of the empirical cumulative distribution functions of the two samples analyzed. The K-S test returns a D statistic and a p-value. The D statistic represents the maximum difference between the two cumulative distributions, and the p-value tells us the probability of observing the data assuming the null hypothesis (i.e. that the samples are from the same distribution) is true. 

* **K-S statistic (D)**: The greater this value (closer to 1.0), the larger the maximum difference between the CDFs, suggesting a greater discrepancy between the two groups. The lower this value (closer to 0.0), the more the distributions of the two samples are similar or the same.
* **p-value**: A small p-value (typically ≤ 0.05) suggests that the samples come from different distributions. If this value is less than or equal to 0.05, the difference in distributions is considered statistically significant, meaning it's unlikely the difference is due to random chance.

## Consistency Check - Global Conclusion
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


In the table below we display the details of the slots where theft happened, the ranking of repeated offenders, and the value which offenders have taken from the Rocketpool protocol.

![image](https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/1cec55c7-681e-4db6-84fc-f83be39948a0)

![image](https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/4eecc678-78c8-44d6-b784-273e33d037c3)



