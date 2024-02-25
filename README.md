[WIP!!!!]

# MEV Theft Tracker

### Analysis Scope
Proposal to create ongoing tools for analysing and analysis of the performance of Rocket Pool validators in collecting priority fees and MEV, and development of reporting mechanisms to surface MEV theft. Built for the Rocket Pool GMC's Bounty [XXXX], and as a continuation of Bounty [BA032304](https://dao.rocketpool.net/t/july-2023-gmc-call-for-bounty-applications-deadline-is-july-15th/1936/6).

For reference and credits: All the scripts used to generate the raw data analyzed below were developed Ramana's and Valdorff's initial analysis of MEVTheft in the RP Protocol,  ["RocketTheft"](https://github.com/xrchz/rockettheft). The analysis of the data itself, as well as the tracking tools and mechanisms around MEV theft within the Rocketpool Protocol are a result of this proposal. 

## Global vs RP consistency check [SLOTS ANALYZED CURRENTLY UP TO 8324999]
Following the same logic as in the original RocketTheft analysis, we start high level and then go specific. This analysis covers 65 weeks of data. It starts right after the MEV grace period ended at slot 5203679 (2022-11-24 05:35:39Z UTC; see https://discord.com/channels/405159462932971535/405163979141545995/1044108182513012796), and ends at slot 8500000 (2024-02.25 01:20:23 UTC). We will name this set of datapoints "the entire distribution" in this analysis. 

We start by evaluating whether Rocketpool ("RP") is being consistently lucky or unlucky against the non-RP Ethereum validating cohort. The plots below shows a cumulative distribution function ("CDF") for the maximum bids on all Ethereum blocks (blue dots/line) and just RP blocks (orange dots/line).  Besides doing a visual evaluation for each of the cohorts, we apply the Kolmogorov-Smirnov (K-S) statistical evaluation on the entire distribution, and on subsets of the entire distribution, in order to compare RP vs non-RP maximum bids distribution.

The Kolmogorov-Smirnov (K-S) test is a non-parametric test that compares two samples to see if they come from the same distribution. It's useful in this case because it doesn't assume any specific distribution of the data and is sensitive to differences in both location and shape of the empirical cumulative distribution functions of the two samples.

The K-S test returns a D statistic and a p-value. The D statistic represents the maximum difference between the two cumulative distributions, and the p-value tells us the probability of observing the data assuming the null hypothesis (that the samples are from the same distribution) is true. 

* **K-S statistic (D)**: The greater this value (closer to 1.0), the larger the maximum difference between the CDFs, suggesting a greater discrepancy between the two groups. The lower this value (closer to 0.0), the more the distributions of the two samples are similar or the same.
* **p-value**: A small p-value (typically ≤ 0.05) suggests that the samples come from different distributions. If this value is less than or equal to 0.05, the difference in distributions is considered statistically significant, meaning it's unlikely the difference is due to random chance.

If we take a look at the entire distribution, we see no evidence that RP gets better or worse bids vs non-RP validators.
* Number of 'Is RocketPool: TRUE' datapoints: 82631
* Number of 'Is RocketPool: FALSE' datapoints: 2961141
* K-S statistic: 0.0032768063669136316
* p-value: 0.3531105728186986

![Figure_1](https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/8d7cb61d-3877-4992-8ddb-f34933801e6c)

If we break this analysis down to specific maximum bid ranges, we do see discrepancies between the RP and non-RP cohorts, specifically in very low maximum bid ranges:

Range: 0.001-0.01 ETH
* Number of 'Is RocketPool: TRUE' datapoints: 248
* Number of 'Is RocketPool: FALSE' datapoints: 7769
* K-S statistic: 0.03451216372763547
* p-value: 0.927149445817151

![Figure_2_ 001- 01](https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/f8bcef00-f5ec-4aff-915b-51f55420808a)

Range: 0.01-0.1 ETH
* Number of 'Is RocketPool: TRUE' datapoints: 59505
* Number of 'Is RocketPool: FALSE' datapoints: 2133965
* K-S statistic: 0.004604945821817363
* p-value: 0.1710550097270287

![Figure_2_ 01- 1](https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/95c6fbba-032f-4bdf-8fdd-f86aa4dd3392)

Range: 0.1-1.0 ETH
* Number of 'Is RocketPool: TRUE' datapoints: 21803
* Number of 'Is RocketPool: FALSE' datapoints: 778734
* K-S statistic: 0.00868652282605109
* p-value: 0.08099163157008771

![Figure_2_ 1-0](https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/78998236-9256-48ef-aebb-24349fa80fef)

Range: 1.0-10 ETH
* Number of 'Is RocketPool: TRUE' datapoints: 1025
* Number of 'Is RocketPool: FALSE' datapoints: 38908
* K-S statistic: 0.033241467677347675
* p-value: 0.21479158055340364

![Figure_2_0-1](https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/6f055d98-f9a0-4427-8b90-02b6d7715c79)

Range: 10-1000 ETH
* Number of 'Is RocketPool: TRUE' datapoints: 50
* Number of 'Is RocketPool: FALSE' datapoints: 1768
* K-S statistic: 0.11309954751131222
* p-value: 0.5249217738316985

![Figure_2_1-3](https://github.com/ArtDemocrat/MEVTheftTracker/assets/137831205/c300fec4-0388-4fef-acc4-b89249b3ad9f)

