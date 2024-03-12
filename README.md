[ :arrows_counterclockwise: WIP!!!!]

# MEV Loss Tracker

### Analysis Scope
The goal of this research paper is to: 
* Report on the state of MEV theft for the past 65 weeks.
* Evaluate the need for creating tools to analyze the performance of Rocket Pool validators in collecting priority fees and MEV.
* If the need is there, create reporting mechanisms and tools to surface MEV theft. 

This research is produced for the Rocket Pool GMC's Bounty [XXXX], and as a continuation of Bounty [BA032304](https://dao.rocketpool.net/t/july-2023-gmc-call-for-bounty-applications-deadline-is-july-15th/1936/6).

**For reference and full credits:** The scripts used to generate the raw data analyzed below were developed Ramana's and Valdorff's initial analysis of MEVTheft in the RP Protocol,  ["RocketTheft"](https://github.com/xrchz/rockettheft). The scripts used for the analysis of the data, as well as the proposed tools and mechanisms to track MEV theft within the Rocketpool Protocol, are designed and produced by ArtDemocrat. 

Following the same logic as in the original RocketTheft analysis, we start high level and then go specific. This analysis covers 65 weeks of ethereum slots. It starts right after the MEV grace period ended at slot 5203679 (2022-11-24 05:35:39Z UTC; see https://discord.com/channels/405159462932971535/405163979141545995/1044108182513012796), and ends at slot 8500000 (2024-02.25 01:20:23 UTC). We will name this set of datapoints "the entire distribution" in this analysis. 

## Rocketpool vs Non-Rocketpool Maximum Bid (Ξ) Consistency Check 

We start by evaluating whether Rocketpool ("RP") is being consistently lucky or unlucky against the non-RP Ethereum validating cohort when it comes to the maximum bids received by ethereum relayers. We do this by plotting a cumulative distribution function ("CDF") for the maximum bids on all Ethereum slots (blue dots/line) and another one for RP blocks (orange dots/line).  Besides doing a visual evaluation for each of the cohorts, we apply the Kolmogorov-Smirnov (K-S) statistical evaluation on the entire distribution, and on subsets of the entire distribution, in order to compare RP vs non-RP maximum bids distribution (see table below).

The Kolmogorov-Smirnov (K-S) test is a non-parametric test that compares two samples to see if they come from the same distribution. It's useful in this case because it doesn't assume any specific distribution of the data and is sensitive to differences in both location and shape of the empirical cumulative distribution functions of the two samples analyzed. The K-S test returns a D statistic and a p-value. The D statistic represents the maximum difference between the two cumulative distributions, and the p-value tells us the probability of observing the data assuming the null hypothesis (i.e. that the samples are from the same distribution) is true. 

* **K-S statistic (D)**: The greater this value (closer to 1.0), the larger the maximum difference between the CDFs, suggesting a greater discrepancy between the two groups. The lower this value (closer to 0.0), the more the distributions of the two samples are similar or the same.
* **p-value**: A small p-value (typically ≤ 0.05) suggests that the samples come from different distributions. If this value is less than or equal to 0.05, the difference in distributions is considered statistically significant, meaning it's unlikely the difference is due to random chance.

# Consistency Check - Global Conclusion
If we take a look at the entire distribution, **we see no evidence that RP gets better or worse bids vs non-RP validators.**
* Total number of rows being plotted between 0.001 ETH and 1000 ETH: 3309302
* Number of 'Is RocketPool: TRUE' datapoints: 90220
* Number of 'Is RocketPool: FALSE' datapoints: 3219082
* :white_check_mark: K-S statistic: 0.0027206808206513555
* :white_check_mark: p-value: 0.5335313853854944

<img src="[https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/55a3a6e1-d760-4057-9cb6-5fa0b2e7ad50]" width="1000" height="600" img align="center">

# Consistency Check - Cohort Breakdown Conclusion
If we break this analysis down to specific maximum bid ranges, we do see discrepancies between the RP and non-RP cohorts, specifically in very low and very high maximum bid ranges (where RP data becomes scarce). For the purpose of this document we will treat both datasets as similar (i.e. both RP and non-RP cohorts have the same "luck" when it comes to receiving maximum bids from Rocketpool-approved relays).

| **Metric / Range** | **0.001-0.01 ETH**                     | **0.01-0.1 ETH**                    | **0.1-1 ETH**                       | **1-10 ETH**                        | **10-1000 ETH**                     |
|     :---           |     ---:                               |     ---:                            |     ---:                            |     ---:                            |     ---:                            |
| # of Slots:        | 8,017                                  | 2,193,470                           | 800,537                             | 39,933                              | 1,818                               |
| # of RP Slots:     | 248                                    | 59,505                              | 21,803                              | 1,025                               | 50                                  |
| # of non-RP Slots: | 7769                                   | 2,133,965                           | 778,734                             | 38,908                              | 1,768                               |
| K-S statistic:     | :warning: 0.034512163727635            | :white_check_mark: 0.00460494582182 | :white_check_mark: 0.00868652282605 | :white_check_mark: 0.03324146767735 | :warning: 0.11309954751131          |
| p-value:           | :white_check_mark: 0.927149445817151   | :white_check_mark: 0.17105500972702 | :white_check_mark: 0.08099163157009 | :white_check_mark: 0.21479158055340 | :white_check_mark: 0.52492177383170 |

<p align="center">
<img src="https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/f8bcef00-f5ec-4aff-915b-51f55420808a" width="500" height="300"><img src="https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/95c6fbba-032f-4bdf-8fdd-f86aa4dd3392" width="500" height="300">
<img src="https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/78998236-9256-48ef-aebb-24349fa80fef" width="500" height="300"><img src="https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/6f055d98-f9a0-4427-8b90-02b6d7715c79" width="500" height="300">
<img src="https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/c300fec4-0388-4fef-acc4-b89249b3ad9f" width="500" height="300">
</p>

## Systematic Loss Analysis
In order to analyze MEV loss cases we define 3 types of losses:
1. **Theft**: the fee recipient for a block (according to either the relay's payload if mev_reward is present, or the Beacon chain otherwise) was incorrect. This happens when the fee recipient is not set to either the smoothing pool ("SP") if a node is opted-in the SP, or the node's fee recipient otherwise.
2. **Neglect**: the node accepts a vanilla block, losing profits against a scenario where MEV-boost would have been used. 
