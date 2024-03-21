# MEV Loss Tracker

Authored by:
@ArtDemocrat
@ramana a.k.a. xrchz

### Analysis Scope
**The goal of this research paper is to:**
* Report on the state of MEV theft for the past 65 weeks.
* Evaluate the need for creating tools to analyze revenue loss within the Rocketpool protocol from either MEV theft or vanilla blocks.

**The conclusions of this research paper are:**
1. 👤 Rocketpool has faced 51 cases of MEV Theft (i.e. incorrect fee recipient) since the grace period ended after the Redstone release (39 opted-in smoothing pool, 12 opted-out). While this represents an incidence rate 0.06% across all blocks proposed since the grace period ended, this seems to become more prevalent in recent slots (+34 more cases vs. the 17 cases identified in the initial report up to September 2023). The ETH loss due to these MEV Theft cases stands at 6.29 ETH (+4,28 ETH more vs the 2.11 ETH identified by the initial report up to September 2023).
2. If we also consider slots where no `mev_reward` was registered for the block, we see a total of 883 cases where an incorrect fee recipient was used by RP validators, raising the theft incidence within RP to 1.02%. [**Report Section: "MEV Theft"**](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/README.md#mev-theft)
3. ⚠️ Rocketpool has faced seven repeat offenders (i.e. node addresses that have used an incorrect fee recipient), of which one gathered MEV 19 times outside of the protocol-defined fee recipients. The largest loss after the grace period ended was of 1.66 ETH (slot: 6376024, fee recepient: btoast777.eth). However, the MEV was manually returned to the smoothing pool by the node operator in this specific case (see [**Report Section: "MEV Theft"**](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/README.md#mev-theft)).
4. 💻 Rocketpool validators have proposed 6,651 vanilla blocks (3,3k SP operators and 3,3k non-opted-in operators) in the timeframe analyzed, leading to a revenue loss of 620.4 ETH (see [**Report Section: "Neglected Revenue"**](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/README.md#neglected-revenue)).
5. 🔁 Based on the GMC, pDAO, and community feedback on this report, we would evaluate the request of a grant to create an ongoing workstream to keep this protocol "blindspot" covered, with the following as next steps (see [**Report Section: Conclusions and Next Steps**](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/README.md#conclusions-and-next-steps)):
    1. refining and improving the data analyzed, specifically around neglected revenue (see [**Report Section: "Notes on Neglected Revenue Data"**](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/README.md#notes-on-neglected-revenue-data))
    2. evaluating lean, cost-efficient tools to track MEV loss events on an ongoing basis
    3. coordinate research to define in-protocol mechanisms that can act on and mitigate MEV loss cases.
6. Rocketpool should move to MEV capture Phase 3 "Required" as soon as possible, to minimize losses coming from vanilla block building (see [**Report Section: "Neglected Revenue**](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/README.md#neglected-revenue)).

This research is produced for the Rocket Pool GMC [Retroactive Grant XXXX], and as a continuation of Bounty [BA032304](https://dao.rocketpool.net/t/july-2023-gmc-call-for-bounty-applications-deadline-is-july-15th/1936/6).

**For reference and full credits:** The scripts used to generate the raw data analyzed below were developed Ramana's and Valdorff's initial analysis of MEVTheft in the RP Protocol,  ["RocketTheft"](https://github.com/xrchz/rockettheft). The scripts used for the analysis of the data, as well as the proposed tools and mechanisms to track MEV theft within the Rocketpool Protocol, are designed and produced by ArtDemocrat. 

Following the same logic as in the original RocketTheft analysis, we start high level and then go specific. This analysis covers 65 weeks of ethereum slots. It starts right after the MEV grace period ended at slot 5203679 (2022-11-24 05:35:39Z UTC; see https://discord.com/channels/405159462932971535/405163979141545995/1044108182513012796), and ends at slot 8,499,999 (2024-02-25 01:20:11 UTC). We will name this set of datapoints "the entire distribution" in this analysis. 

## Rocketpool vs Non-Rocketpool Maximum Bid (Ξ) Consistency Check 

The first thing that we analyze is whether Rocketpool ("RP") is consistently lucky or unlucky against the non-RP Ethereum validators when it comes to the maximum bids received by ethereum relayers. The conclusion: No. As expected, RP validator's "luck" in terms of bids received (and accepted) is aligned with the non-RP validator cohort.

We confirmed this by plotting a cumulative distribution function ("CDF") for the maximum bids on all Ethereum slots (blue dots/line) and another one for RP blocks (orange dots/line). See CDF charts below.  Besides doing a visual evaluation for each of the cohorts, we apply the Kolmogorov-Smirnov (K-S) statistical evaluation on the entire distribution, and on subsets of the entire distribution, in order to compare RP vs non-RP maximum bids distribution (see table below).

The Kolmogorov-Smirnov (K-S) test is a non-parametric test that compares two samples to see if they come from the same distribution. It's useful in this case because it doesn't assume any specific distribution of the data and is sensitive to differences in both location and shape of the empirical cumulative distribution functions of the two samples analyzed. The K-S test returns a D statistic and a p-value. The D statistic represents the maximum difference between the two cumulative distributions, and the p-value tells us the probability of observing the data assuming the null hypothesis (i.e. that the samples are from the same distribution) is true. 

* **K-S statistic (D)**: The greater this value (closer to 1.0), the larger the maximum difference between the CDFs, suggesting a greater discrepancy between the two groups. The lower this value (closer to 0.0), the more the distributions of the two samples are similar or the same.
* **p-value**: A small p-value (typically ≤ 0.05) suggests that the samples come from different distributions. If this value is less than or equal to 0.05, the difference in distributions is considered statistically significant, meaning it's unlikely the difference is due to random chance.

## Consistency Check - Global Conclusion
[Analysis Script](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/generate_slot_reward_distro_sections)

If we take a look at the entire distribution of slots which had `max_bid`, **we see no evidence that RP gets better or worse bids vs non-RP validators.**
* Total number of rows being plotted between 0.001 ETH and 1000 ETH: 3,123,540
* Number of 'Is RocketPool: TRUE' datapoints: 85,996
* Number of 'Is RocketPool: FALSE' datapoints: 3,037,544
* :white_check_mark: K-S statistic: 0.003275145183045336
* :white_check_mark: p-value: 0.3303187837164987

<p align="center"> 
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/c1bcfd46-f4e8-4d76-bb05-eb6b3a7bc5c4" width="1000" height="600">

## Consistency Check - Cohort Breakdown Conclusion
If we break this analysis down to specific maximum bid ranges, we do see discrepancies between the RP and non-RP cohorts, specifically in very low and very high maximum bid ranges (where RP data becomes scarce). For the purpose of this document we will treat both datasets as similar (i.e. both RP and non-RP cohorts have the same "luck" when it comes to receiving maximum bids from Rocketpool-approved relays).

| **Metric / Range** | **0.001-0.01 ETH**                     | **0.01-0.1 ETH**                     | **0.1-1 ETH**                        | **1-10 ETH**                        | **10-1000 ETH**                     |
|     :---           |     ---:                               |     ---:                             |     ---:                             |     ---:                            |     ---:                            |
| # of Slots:        | 9,355                                  | 2,263,236                            | 811,143                              | 38,167                              | 1,646                               |
| # of RP Slots:     | 287                                    | 62,262                               | 22,413                               | 991                                 | 43                                  |
| # of non-RP Slots: | 9,068                                  | 2,200,974                            | 788,730                              | 37,176                              | 1,603                               |
| K-S statistic:     | :warning: 0.029245545464465925         | :white_check_mark: 0.0044600383938   | :white_check_mark: 0.0085249763091   | :warning: 0.02339956205809          | :warning: 0.08498600008704          |
| p-value:           | :white_check_mark: 0.965449659876372   | :white_check_mark: 0.1791694749921   | :white_check_mark: 0.0837100753605   | :white_check_mark:  0.6572090402610 | :white_check_mark: 0.89695211324218 |

**Slots between 0.001-0.01 ETH MEV Rewards:**
<p align="center">
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/1413cf07-f26d-4be2-b9bd-aadfb85c43b3" width="600" height="360">

**Slots between 0.01-0.1 ETH MEV Rewards:**
<p align="center">
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/d3ebf9b7-1013-4a74-a47a-9a5f836423a4" width="600" height="360">

**Slots between 0.1-1 ETH MEV Rewards:**
<p align="center">
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/84d8c5a1-4a34-4d84-a63e-dfc2a21e685d" width="600" height="360">

**Slots between 1-10 ETH MEV Rewards:**
<p align="center">  
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/f70cb0d7-1106-4c2f-9923-70c4522f5d80" width="600" height="360">

**Slots between 10-1000 ETH MEV Rewards:**
<p align="center">
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/40e4aa90-1c2e-4bfb-b1f9-3a1b6319d9b6" width="600" height="360">
</p>

## Systematic Loss Analysis
Once that we confirmed that RP validators stand on a level playing field with non-RP validators, we proceed to analyze cases of revenue loss within the RP protocol. In order to analyze MEV loss cases we define 2 types of revenue losses for the RP protocol:
1. **MEV Theft**: the fee recipient for a block (according to either the relay's payload if mev_reward is present, or the Beacon chain otherwise) was incorrect. This happens when the fee recipient is not set to either the smoothing pool ("SP") if a node is opted-in the SP, or the node's fee recipient otherwise.
2. **Neglected Revenue**: the node proposes a vanilla block, losing profits against a scenario where an MEV-boost-optimized block (with traditionally higher MEV rewards) could have been proposed.

<p align="center">
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/faf00243-91cf-48a2-a5bc-261df0024db3" width="600" height="360">
</p>

### MEV Theft
[Analysis Script](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/generate_mevreward_theft)
[Analysis Script - Details](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/generate_mevtheft_details)

As explained in the **"Consinstency Check - Global Conclusion"** section of this report, in the time between the grace period ended, and until slot 8,499,999, 85,996 blocks were proposed by RP validators. In this section we analyze whether a RP validator behaved honestly by sending MEV rewards to the correct fee recipients defined by the protocol. 

First we begin by plotting the MEV rewards of each slot where we deemed the fee recepient for a proposed block as incorrect. During the analyzed timeframe, 51 cases of MEV Theft ocurred (vs. 17 such cases identified [in the initial MEV Theft report](https://github.com/xrchz/rockettheft/blob/main/README.md#current-losses) 6 months ago). If we analyze these cases we can see that the smoothing pool is slightly more affected (39 theft cases) vs non-opt-in validators (12 theft cases). This derived in a total loss of 6.29 ETH for the rocketpool protocol ((vs. 2.11 ETH identified [in the initial report](https://github.com/xrchz/rockettheft/blob/main/README.md#current-losses)), split as shown below:

* Total number of rows being plotted between 0.001 ETH and 1000 ETH: 51
* Number of 'In Smoothing Pool: TRUE' datapoints: 39
* Number of 'In Smoothing Pool: FALSE' datapoints: 12
* Total ETH theft in smoothing pool: 4.361759122909182 ETH
* Total ETH theft outside of smoothing pool: 1.928289059541783 ETH

It is worth mentioning that, although we identified 51 cases where a `mev_reward` amount was observed in the slot and was sent to an incorrect fee recipient, there are 832 additional cases where even if an `mev_reward` was not registered for the slot, an incorrect fee recipient was used. These 832 cases sum-up to a loss/theft of 79.6 ETH if we take the `max_bid` data available for these slots as a proxy for the ETH which could have been stolen. It is worth noting that all these slots are covered in the following section of this report (**"Neglected Revenue"**), since they fall under the category of "vanilla blocks" due to the absence of an MEV relayer and bid for the proposed block.

In the first chart below we plot the 51 cases of theft split between smoothing pool vs the non-opt-in cases. In the second chart below we plot the magnitude of the 51 stolen MEV rewards (Y axis) and the slot where these took place (X axis), separating between smoothing pool and non-opt-in cases. 

<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/861d8e80-90d1-45e7-bd4e-a63fbcd97aa5" width="1000" height="600">
<img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/ac5e3d5f-54fb-4f0b-8c40-582dc4fc95c7" width="1000" height="600">

In the table below we display the ranking of repeated offenders, and the value which offenders have taken from the Rocketpool protocol.

<p align="center">
  <img width="500" height="400" src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/9498c847-ce4c-4107-9ffe-1507c84c4dda">
</p>

*The largest MEV reward channeled to an incorrect fee recipient happened in slot 6376024 and was due a configuration error after a solo migration took place. The Node Operator immediately sent the correct amount to the smoothing pool (see https://etherscan.io/tx/0x18a28f9bba987a05bc87515faa6490cef3fe61b02dc45d68cffcf3a4e6f791a0).

In the second table below we display the details of the slots where theft happened.

<p align="center">
  <img width="620" height="1000" src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/4eecc678-78c8-44d6-b784-273e33d037c3">
</p>

**Conclusion:** While 51 theft cases out of 85,996 Rocketpool block proposals analyzed in this time series represent a low incidence of 0.06%, it seems that theft is a phenomenon which is happening continuously within the protocol. Secondly, MEV theft incidence seems to have become more prevalent in recent slots. Finally, if we consider not only the 51 slots where an `mev_reward` was observerd for the block, but rather the total (51+832)= 883 cases where an incorrect fee recipient was used, the theft incidence within RP climbs to 1.02%.

### Neglected Revenue
[Analysis Script](https://github.com/ArtDemocrat/MEVLossTracker/blob/main/generate_mevreward_neglect)

The second case of revenue loss for the RP protocol is driven by validators which do not choose maximize the MEV rewards made available to them by the Ethereum MEV supply chain. This happens when a RP validator does not register with any MEV relayer and produces so called "vanilla blocks". These blocks don't follow the transaction-ordering reward-maximizing logic which MEV searchers, builders, and relayers pass on to validators. For the purpose of this analysis, vanilla blocks were quantified based on the slots where no `mev_reward_relay` data was registered for a slot. Based on this logic, we can conclude the following:

- Vanilla blocks have been proposed by RP validators in 6,651 slots since the grace period ended (3,3k SP operators and 3,3k non-opted-in operators).
- This leads to a total loss revenue of 620.4 ETH for RP (280.6 ETH loss for the SP, and 339.8 loss for rETH holders). This would represent a 5 basis point ("bps" - i.e. if APR is 1%, it would increase to 1.05%) APR improvement on the current 1.12M ETH staked in the beacon chain by RP (Status 2024-03-18, [source](https://dune.com/drworm/rocketpool)). The amount of ETH loss in the APR calculation corresponds to a timeframe larger than 12 months (i.e the grace period ended in November 2022, and slot 8.5M took place on Feb. 25th, 2024). Therefore, since it is not not accurate, the APR loss calculation simply aims to give a sense of the magnitude of the APR loss RP faces on this front.
- There is a second level loss which comes from accepting an MEV bid which is lower than the max_bid registerd for a validator in a particular slot. This can be due to several reasons which typically cannot be influenced by RP (such as a validator avoiding unregulated relayers, see details [here](https://docs.rocketpool.net/guides/node/mev#block-builders-and-relays)). However, it is worth noting that if we simply observe the sum of mev_reward captured by RP validators in the analyzed period (9,997.4 ETH) and compare it with the sum of max_bids which could have theoretically been captured by RP validators (11.729.0 ETH), capturing that difference (1.731,6 ETH) would derive on a 15bps APR improvement on RP's current staked capital. Here again, the timelines observed are longer than 12 months, and the APR calculation is shared purely for illustrative purposes.

Regarding the last bullet piont, in case we overlooked drivers which could be actionable by by the RP protocol in order to close the gap of MEV capture vs the theoretical maximum, we would appreciate the community's input on this in this research's retroactive grant posted in the RP [governance forum](XXXX). 

In order to visually represent the loss driven by vanilla blocks we present the chart below which plots the distribution of the 6,651 slots where vanilla blocks were proposed by RP validators. We see a random distribution which tends to become less prevalent towards recent slots (potentially due to the protocol moving MEV Capture Phase 2 "Opt-out" after November 2022). One recommendation of this report is to move to MEV capture Phase 3 "Required" as soon as possible, in order to minimize the losses aforementioned losses due to vanilla block proposals. The selection of regulated vs non-regulated relayer usage should continue to be defined entirely, needless to say, by each node operator's preferences.

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

It is worth mentioning that within the datapoints shown above, the % of MEV-neglected slots (i.e. no relayer was observed in a slot with a successfuly proposed block) is double as high in non-RP proposals (15% non-RP vs 7.6% RP). This would hint towards the fact that MEVboost usage is more prevalent among the RP validator set than among other entities, which is net positive for the RP protocol's revenue generation. 

#### Notes on Neglected Revenue Data
Quantifying the losses incurred by vanilla blocks is a complex task since we cannot always asses with 100% certainty which validator is leveraging MEVboost, from which relayer, and to which extent. The reasons for this are:

1. Some relayers don't always make their MEV bid data available to the public, which could cause a wrong classification of vanilla blocks while these blocks actually had a bid from a relayer. For the scope of this report, we simply classify slots with no RP-approved relayer in our dataset (see specifics around the underlying dataset [here](https://github.com/xrchz/rockettheft/blob/main/README.md#data-notes)) as vanilla blocks. 
2. Secondly, "bloXroute ethical" (a RP-approved relayer) sunset its API, which prevented us from using it correctly even if it is classified as a RP-approved relayer. In order to mitigate this, RP vanilla blocks are checked to see if they had relay rewards according to beaconcha.in. If so, they are simply dropped from the dataset to avoid polluting the vanilla block category with known MEV. Also, if there are multiple relayers which provide the same block, since it is not possible to know which relay was actually used, such a slot is removed from this list. The estimation of the original bounty puts these "dropped" blocks at 19% of all vanilla blocks, which is a material percentage. See "Next Steps" section below for details on how we could mitigate this and other data quality issues in future iterations of this report.
3. If a validator is not registered with any MEV relayer in a slot, no max_bid will be visible in the dataset. This is an issue is solved by using the average of the 3 slots before and after the missing max_bid block to calculate the amount of ETH neglected. See "Next Steps" section below for details on how we could mitigate this and other data quality issues in future iterations of this report.
4. See other important data caveats for the underlying dataset [here](https://github.com/xrchz/rockettheft/blob/main/README.md#data-notes).

Another pattern which we see in the data, which we would like to take a closer look at in future iterations of this report, is that there are often blocks where the max_bid offered by a relayer is orders of magnitude higher than the actual mev_reward observed for a specific slot. This is something expected for cases where validators allowlist only a certain group of relayers. For example, RP validators can pick and choose certain relayers from a list of six options, see list [here](https://docs.rocketpool.net/guides/node/mev#block-builders-and-relays)). It could be that in those "pick and choose" cases, some of the more performant searcher/builder/relayer chains are omitted, and therefore `max_bid` is much hgher than the actual 'mev_reward`. That said, we see cases where the magnitude in which this phenomenon is observed (i.e. `max_bid raises the need to confirm whether such cases are correct, or whethere we need to implement adjustments or workarounds in our data pulls for future iterations (See "Next Steps" section below for details on ideas around how we could mitigate this and other data quality issues in future iterations of this report). 

To ilustrate this topic, the table below shows the number of slots which display a max_bid which is N times higher than the actual mev_reward registered for a slot:

| **N**        | **Number of Blocks where max_bid=N*mev_reward** |
|     :---     |     ---:                                        |
| 1.5          | 176,516                                         |
| 2            | 79,674                                          |
| 3            | 35,632                                          |
| 4            | 21,696                                          |
| 5            | 15,215                                          |
| 10           | 8,270                                           |
| 15           | 5,489                                           |
| 25           | 1,500                                           |

If we focus on the 15,215 blocks resulting from a threshold of N=5 (i.e. the max_bid received by a validator is 5 times larger than the actual mev_reward registered for that same slot), the average and median differences between the `max_bid` and `mev_reward` are as shown below:

```
Number of problematic blocks - Reward size: 15215
Average max_bid to mev_reward difference in Reward size problematic blocks: 0.7633415742569226
Median max_bid to mev_reward difference in Reward size problematic blocks: 0.27212222131634695
```

The tables below show how these 15,215 potentially "problematic" blocks are distributed across slots and MEV reward magnitudes

<p align="center">
  <img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/8ed210d8-f88b-4bd1-9c61-47226faf1fd0">
</p>

<p align="center">
  <img src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/69d8a92e-94b6-4382-a034-b0ffcc6a601e">
</p>

It is important to consider the aforementioned data quality issues we face when it comes to neglected ETH rewards. Therefore, in order for this to be addressed and researched further in future iterations of MEV Loss reporting for RP we proposed a path forward in the "Next Steps" section below. 

## Conclusions and Next Steps
Based on the information presented on this report we concluded that:
- We see an MEV theft incidence rate in up to 1.02% across RP-proposed blocks since the post-Redstone update grace period ended (0,06% incidence rate if we only take blocks that actually registered an mev_reward).
- There could potentially be up to 1.731,6 ETH left on the table from validators not capturing `max_bid` to the full extent in which MEV rewards are passed on to them (sometimes due to relayer preferences from validators). From that, 620.4 ETH is confirmed as actual vanilla block MEV losses, coming from slots where no `mev_reward_relay' was registered at all.
- The data analyzed, especially around vanilla block neglected revenue, is prone to have inacuracies due to the complex datasource landscape when it comes to the MEV supply chain. For this reason, we propose to join forces with NonFungibleYokem and Cayos from the Rocketpool community to keep on working on a unified data source which can become the source of truth for these types of analyis. This point, however, would become less relevant as soon as the protocol moves to MEV capture Phase 3 "Required", since the vanilla block loss would be de facto eliminated.

With that last point functioning as a segue to the next steps, we propose to:
1. Refresh this report once per quarter, working jointly with NonFungibleYokem and Cayos to produce a unified Rocketpool dataset.
2. Evaluate lean, cost-efficient tools to track MEV loss events on an ongoing basis. These could potentially replace a quarterly, manually-produced, report. Point 1. would anyhow still need to be completed for this purpose.
3. Coordinate research with RP community members to define in-protocol mechanisms that can act on and mitigate MEV loss cases.

We look forward to hearing the community, GMC, and pDAO thoughts/feedbacks/comments on this research in the retroactive grant posted in the RP [governance forum](XXXX). We specifically look for feedback and ideas on the three steps proposed above (which would serve as the basis to request a follow-up bounty to continue this project).

**Authored by:**
<p align="left">
  <img width="70" height="70" src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/da012a89-2ec8-4e2f-bd8d-6f4b7fec0a72">
</p>

**@ArtDemocrat**

</p>
  <img width="70" height="70" src="https://github.com/ArtDemocrat/MEVLossTracker/assets/137831205/5254358c-efca-482c-b9ff-67484da15be0">
</p>

**@ramana**
