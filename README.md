# CATrustData
Analysis &amp; results from the SURGe Certificate Authority Trustworthiness project

## What is this repo?
Security relies on trust, especially when it comes to Certificate Authorities. Browsers ship with many root CAs built in, but are they all equally trustworthy? Splunk's SURGe research team examined over 5 billion recent TLS certificates used for secure web sites (WebPKI) to try to find the answer to this question.  

This repo is a companion to David Bianco's 2023 RSA talk, "[Trust Unearned? Evaluating CA Trustworthiness Across 5 Billion Certificates](https://www.rsaconference.com/USA/agenda/session/Trust%20Unearned%20Evaluating%20CA%20Trustworthiness%20Across%202%20Billion%20Certificates)". It contains the raw risk rankings for all root CAs and the top 10k issuing CAs we encountered in our research.

## What does the data look like?
The risk rankings are distributed in the form of two CSV files:

* **ca-trust-root-risk-scores.csv** ("the root file", 497 entries)
* **ca-trust-issuer-risk-scores-10k.csv** ("the issuer file", 10k entries)

Each file has a consistent set of fields:
* The name of the CA (the `root_ca` or the `issuer` field, depending on which file you're looking at)
* `total_certs`: the total number of certificates issued by this CA. For roots, this is the total number of certificates where the trust chain is anchored to the root CA)
* `risky_certs`: The number of certificates associated with this CA that our threat intel partners observed participating in malicious activity
* `risk_percent`: The ratio of risky to benign certificates (risky_certs / total_certs)
* `tier`: In order to ensure fair comparisons, we divided the CAs into tiers, based on their total_certs values. This allows us to do an apples-to-apples comparison of CAs which are (roughly) the same size. These are strings and will be one of : "tier1", "tier2", "tier3" or "tier4". 
  * **Tier 1 Roots**: >= 10m certificates
  * **Tier 2 Roots**: 1m - 10m certificates
  * **Tier 3 Roots**: 100k - 1m certificates
  * **Tier 4 Roots**: <= 100k certificates
  * **Tier 1 Issuers**: >= 100k certificates
  * **Tier 2 Issuers**: 10k - 100k certificates
  * **Tier 3 Issuers**: 1k - 10k certificates
  * **Tier 4 Issuers**: <= 1k certificates
* `zscore`: The number of standard deviations away from the mean for this CA's `risky_percent`, compared to the rest of it's tier. Larger numbers are more risky, smaller numbers are more trustworthy. In our research, we used an outlier threshold of +/- 3.0 to identify risky or trusty outliers, but here we just provide the zscore. You will need to impose your own interpretation.

Each file contains one record per root/issuing CA we encountered in our dataset. The root file is comprehensive (i.e., it contains data for all root CAs we discovered), while the issuer file only contains the top 10,000 issuing CAs by risk score. Although we counted ~78,000 distinct issuers, there was an extremely long tail of issuers with 0% risk. Every issuer with any risk rating at all is present in the issuer file. If an issuer is not present, assume that we either didn't find it in our data or (more likely) the risk percentage was 0%.

## How should I use this data?
That's up to you! We think this would be a great start to a risk-based approach to alerting or hunting (RBA or RBH). Our findings don't support directly alerting on certificates just because they happen to be issued or anchored by any of these CAs, but if used in combination with other factors or as part of an RBA/RBH strategy, these scores can be very helpful.

Please note, however, that **this is a point-in-time snapshot** and will become increasingly unreliable as time goes on. We feel that this is good data right now (Spring 2023), but *downloadeat emptor*. 

