# Statistics Pipeline Service
This repository contains code that processes NDT data and provides aggregate
metrics by day for standard global, and some national geographies. The resulting
aggregations are made available in JSON format, for use by other applications.

The `stats-pipeline` service is written in Go, runs on GKE, and generates and
updates daily aggregate statistics. Access is provided in public BigQuery tables
and in per-year JSON formatted files hosted on GCS.

## Documentation Provided for the Statistics Pipeline Service
* (This document) Overview of the `stats-pipeline` service, fields provided
  (schema), output formats, available geographies, and API URL structure.
* [What Statistics are Provided by `stats-pipeline`, and How are They Calculated?][stats-overview]
* [Geographic Precision in `stats-pipeline`][geo-precision]


[stats-overview]:
[geo-precision]:

## General Recommendations for All Aggregations of NDT data
In general, [our recommendations][recommendations] for research aggregating NDT data are:

* Don't oversimplify
* Aggregate by ASN in addition to time/date and location
* Be aware and illustrate multimodal distributions
* Use histogram and logarithmic scales
* Take into account, and compensate for, client bias and population drift

[recommendations]: upcoming-blog-post

## Statistics Output Format
All statistics provided by this API are for a particular geography and day,
over a calendar year. In addition, we also provide aggregation of tests per
Autonomous System Number, identifying statistics per provider within each
geography and day/year. The Statistics Pipeline exports aggregate statistics
across supported years and geographies in JSON files named: **histogram_daily_stats.json**

Each file contains a JSON array with daily histograms and aggregate statistics.
Each object in the array represents a histogram bucket per one day, and in our
current histogram buckets there are 8 buckets generated. So for each aggregate
on a given day, there will be 8 JSON objects. For a complete year the file for
an aggregation will contain 365*8 objects.

### Field Schema and Description

Below is a list and description of the fields provided in a JSON object for a
single day and bucket:

| "date":"2020-01-01", | The date in `YYYY-MM-DD` format. |
| "bucket_min":0, | The lower bound of the bucket which the statistics in this
object represent. |
| "bucket_max":1.7782794100389228 | The upper bound of the bucket which the
statistics in this object represent. | 
| "dl_LOG_AVG_rnd1":25.591 | The LOG Average of download measurements in this
aggregation, using the first randomly selected test per IP address in the set.
Value is presented in megabits per second. |
| "dl_LOG_AVG_rnd2":25.577 | The LOG Average of download measurements in this
aggregation, using the second randomly selected test per IP address in the set. Value is presented in megabits per second. |
| "dl_minRTT_LOG_AVG_rnd1":26.256 | The LOG Average of Minimum Round Trip Time
of download measurements in this aggregation, using the first randomly selected test per IP address in the set. Value is presented in milliseconds. |
| "dl_minRTT_LOG_AVG_rnd2":26.268 | The LOG Average of Minimum Round Trip Time
of download measurements in this aggregation, using the second randomly selected test per IP address in the set. Value is presented in milliseconds. |
| "dl_frac_bucket":0.057 | The fraction of download measurements within this
histogram bucket. |
| "dl_samples_bucket":10695 | The number of download measurement samples in this
bucket. |
| "dl_samples_day":188725 | The number of download measurement samples on this day. |
| "download_MIN":0 | The minimum download speed in megabits per second on this day. |
| "download_Q25":8.141 | The first quartile (25th percentile) download speed in megabits per second on this day. |
| "download_MED":31.95 | The median download speed in megabits per second on this day. |
| "download_AVG":79.745 | The average or mean download speed in megabits per second on this day. |
| "download_Q75":97.572 | The upper quartile (75th percentile) download speed in megabits per second on this day. |
| "download_MAX":3655.15 | The maximum download speed in megabits per second on this day. |
| "download_minRTT_MED":25 | The median Minimum Round Trip Time in milliseconds
for download measurements on this day. |
| "ul_LOG_AVG_rnd1":6.589 | The LOG Average of upload measurements in this
aggregation, using the first randomly selected test per IP address in the set.
Value is presented in megabits per second. |
| "ul_LOG_AVG_rnd2":6.589 | The LOG Average of upload measurements in this
aggregation, using the second randomly selected test per IP address in the set.
Value is presented in megabits per second. |
| "ul_minRTT_LOG_AVG_rnd1":24.988 | The LOG Average of Minimum Round Trip Time
of upload measurements in this aggregation, using the first randomly selected test per IP address in the set. Value is presented in milliseconds. |
| "ul_minRTT_LOG_AVG_rnd2":25.003 | The LOG Average of Minimum Round Trip Time
of upload measurements in this aggregation, using the second randomly selected test per IP address in the set. Value is presented in milliseconds. |
| "ul_frac_bucket":0.113 | The fraction of upload measurements within this
histogram bucket. |
| "ul_samples_bucket":20769 | The number of upload measurement samples in this
bucket. |
| "ul_samples_day":183326 | The number of upload measurement samples on this day. |
| "upload_MIN":0 | The minimum upload speed in megabits per second on this day. |
| "upload_Q25":2.356 | The first quartile (25th percentile) upload speed in megabits per second on this day. |
| "upload_MED":7.857 | The median upload speed in megabits per second on this day. |
| "upload_AVG":28.034 | The average or mean upload speed in megabits per second on this day. |
| "upload_Q75":17.306 | The upper quartile (75th percentile) upload speed in megabits per second on this day. |
| "upload_MAX":3199.958 | The maximum upload speed in megabits per second on this day. |
| "upload_minRTT_MED":23.83 | The median Minimum Round Trip Time in milliseconds
for upload measurements on this day. |
},
```

<FUTURE> histogram_daily_stats.csv
Same data as the JSON, but in CSV. Useful for importing into a spreadsheet.
<FUTURE> histogram_daily_stats.sql
A SQL query which returns the same rows in the corresponding .json and .csv. Useful for verifying the exported data against the source and to tweak the query as needed by different use cases. 



GCS Output Paths
Global level
/v0/asn/<AS#####>

Continent level
/v0/<continent_code>/<year>/
/v0/<continent_code>/asn/<AS#####>/<year>

Country level
/v0/<continent_code>/<country_code>/<year>/
/v0/<continent_code>/<country_code>/asn/<AS#####>/<year>

ISO 3166-2 region level 1
/v0/<continent_code>/<country_code>/<region_code>/<year>/
/v0/<continent_code>/<country_code>/<region_code>/asn/<AS#####>/<year>

City level
/v0/<continent_code>/<country_code>/<region_code>/<city_name>/<year>/
/v0/<continent_code>/<country_code>/<region_code>/<city_name>/asn/<AS#####>/<year>

US Census tract level
/v0/NA/tracts/<GEOID>/<year>/
/v0/NA/tracts/<GEOID>/asn/<AS#####>/<year>

US County level
/v0/NA/US/counties/<GEOID>/<year>
/v0/NA/US/counties/<GEOID>/asn/<AS#####>/<year>

<FUTURE>
US Zip Codes, US Congressional Districts, Block Groups, Blocks (desired by NTIA)
