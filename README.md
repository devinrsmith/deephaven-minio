# deephaven-minio

A quick example of how to get [Deephaven](https://deephaven.io) and [MinIO](https://min.io) configured.

It's important to configure MinIO to accept virtual host-style resolution, as that's the preferred mode of operation for [AWS SDK for Java 2.x](https://docs.aws.amazon.com/sdk-for-java/)[^1]. This can be configured with the environment variable [MINIO_DOMAIN](https://min.io/docs/minio/linux/reference/minio-server/settings/core.html#envvar.MINIO_DOMAIN). For easy DNS via docker, each bucket can then be added as a network alias. This solution works for simple examples, but more complex setups may need to have a proper DNS service to support dynamic bucket names.

Assuming the parquet file exists, with bucket "test-data" and key "my-file.parquet", the following query should then succeed:

```python
from deephaven import parquet
from deephaven.experimental import s3
from datetime import timedelta

my_table = parquet.read(
    "s3://test-data/my-file.parquet",
    special_instructions=s3.S3Instructions(
        region_name="aws-global",
        endpoint_override="http://minio.example.com:9000",
        access_key_id="minioadmin",
        secret_access_key="minioadmin",
    ),
)
```

[^1]: Deephaven _could_ allow [forcePathStyle](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/s3/S3BaseClientBuilder.html#forcePathStyle(java.lang.Boolean)) to be configured to enable path-style resolution, which could simplify deployment complexity in cases where users don't want to manage subdomains.
