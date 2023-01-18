# GCP Related Domain Knowledge

Google Cloud Platform

CREATED 2023/01/12

## Spanner

Spanner is a distributed SQL database management and storage service developed by Google.

### Optimizer

The optimizer's job is to create an efficient plan that is an efficient way to retrieve data from Spanner based on the given SQL query. It has a direct impact on performance of requests issued to your database.

### Version Retention Period (Recovery)

It affects the ability to perform a point in time recovery. By default, it retains all versions of data and schema for 1 hour.

### Encryption

We can use either Google-managed default encryption, or customer-managed encryption keys (CMEK) for Spanner.

## Reference

[1] <https://zhuanlan.zhihu.com/p/85616742>
