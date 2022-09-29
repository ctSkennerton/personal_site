---
title: 'Datelist tables at Roblox Data Engineering Meetup'
date: Thu, 29 Sep 2022 08:00:00 -0700
tags: ['data engineering']
---

Yesterday I went to a data engineering meetup hosted by [Roblox](https://www.roblox.com). The
talk in the meetup was by [Yan Shen](https://www.linkedin.com/in/ynshn/) and [William Ng](https://www.linkedin.com/in/william-ng-337a9525/) on how they cut down on processing
costs in their data processing pipelines by making use of datelist
tables.

A datelist table acts as an intermediate incremental accumulating
aggregate of a quantity from a fact table. Their key feature is
that they have a column that contains an array or map of dates where
the this quantity was observed.

Consider a raw fact table like (but imagine that it's huge and partitioned by day):

| userid | date       | quantity |
|--------|------------|----------|
| 1      | 2022-09-01 | 1        |
| 3      | 2022-09-01 | 5        |
| 1      | 2022-09-04 | 6        |
| 1      | 2022-09-05 | 5        |

A datelist table would convert this information into a summary like:

| userid | fisrt_date | last_date  | date_list                                           | dt         |
|--------|------------|------------|-----------------------------------------------------|------------|
| 1      | 2022-09-01 | 2022-09-05 | {"2022-09-01": 1, "2022-09-04": 6, "2022-09-05": 5} | 2022-09-05 |
| 3      | 2022-09-01 | 2022-09-01 | {"2022-09-01": 5}                                   | 2022-09-05 |

In this case the table granularity is an individual user (one user
per row) and contains a column recording all of the dates at which
quantity has been recorded. This table also has a partitioning
column `dt` that is used to record the date at which the rows were
last updated. The table can also record other important attributes, in
this case the first date and last date that the quantity was recorded.

The value in this structure comes from updating the information when 
new partitions are added to the original fact table. Let's say that
new data comes in that looks like the following:

| userid | date       | quantity |
|--------|------------|----------|
| 3      | 2022-09-06 | 9        |

We can then update the datelist table using only this new partition of the fact table:

| userid | fisrt_date | last_date  | date_list                                           | dt         |
|--------|------------|------------|-----------------------------------------------------|------------|
| 1      | 2022-09-01 | 2022-09-05 | {"2022-09-01": 1, "2022-09-04": 6, "2022-09-05": 5} | 2022-09-06 |
| 3      | 2022-09-01 | 2022-09-06 | {"2022-09-01": 5, "2022-09-06": 9}                  | 2022-09-06 |

This can be achieved incrementally as the datelist table contains
all of the history, so to generate the current datelist table we
need only the previous days datelist table and the current partition
of the fact table.

For small datasets, it's possible to easily scan over the historical
data, but as the dataset grows that becomes an unreasonable compute
burden. In his talk, Yan and WIlliam gave the example of scanning
over a raw fact table that had 10 TB of data generated per day.
Scanning all the historical data every time a query was run required
looking at petabytes of data. In comparison, using datelist table
they only needed to scan through 10TB + ~0.5TB per day.

The datelist table can then act as an efficient intermediate table
for calculating historical metrics like retention and totals of the
quantity over the last 7 days, 28 days etc., from the table above
it's easy to aggregate the quantity over the previous 7 days:

| userid | quantity_L7 | dt         |
|--------|-------------|------------|
| 1      | 12          | 2022-09-06 |
| 3      | 14          | 2022-09-06 |
