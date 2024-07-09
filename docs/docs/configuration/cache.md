# Cache
The cache directory includes 

- [Vulnerability Database][trivy-db][^1]
- [Java Index Database][trivy-java-db][^2]
- [Misconfiguration Checks][misconf-checks][^3]
- Cache of previous scans.
 
The cache option is common to all scanners.

## Clear Caches
`trivy clean` subcommand removes caches.

```bash
$ trivy clean --scan-cache
```

<details>
<summary>Result</summary>

```
2024-06-21T21:58:21+04:00       INFO    Removing scan cache...
```

</details>

If you want to delete cached vulnerability databases, use `--vuln-db`.
You can also delete all caches with `--all`.
See `trivy clean --help` for details.

## Cache Directory
Specify where the cache is stored with `--cache-dir`.

```bash
$ trivy --cache-dir /tmp/trivy/ image python:3.4-alpine3.9
```

## Scan Cache Backend
!!! warning "EXPERIMENTAL"
    This feature might change without preserving backwards compatibility.

Trivy utilizes a scan cache to store analysis results, such as package lists.
It supports four types of backends for this cache:

- Local File System (`fs`)
    - The cache path can be specified by `--cache-dir`
- Memory (`memory`)
- Redis (`redis://`)
    - `redis://[HOST]:[PORT]`
    - TTL can be configured via `--cache-ttl`
- S3 (`s3://`)
    - `s3://[BUCKET]/[PREFIX]`

### Local File System
The local file system backend is the default choice for container and VM image scans.
When scanning container images, it stores analysis results on a per-layer basis, using layer IDs as keys.
This approach enables faster scans of the same container image or different images that share layers.

!!! note
    Internally, this backend uses [BoltDB][boltdb], which has an important limitation: only one process can access the cache at a time.
    Subsequent processes attempting to access the cache will be locked.
    For more details on this limitation, refer to the [troubleshooting guide][parallel-run].

### Memory
The memory backend stores analysis results in memory, which means the cache is discarded when the process ends.
This makes it useful in scenarios where caching is not required or desired.
It serves as the default for repository, filesystem and SBOM scans and can also be employed for container image scans when caching is unnecessary.

To use the memory backend for a container image scan, you can use the following command:

```bash
$ trivy image debian:11 --cache-backend memory
```

### Redis
The Redis backend is particularly useful when you need to share the cache across multiple Trivy instances.
You can set up Trivy to use a Redis backend with a command like this:

### S3
Authentication is obtained via the [AWS SDK's default credential provider chain](https://aws.github.io/aws-sdk-go-v2/docs/configuring-sdk/#specifying-credentials).

This feature can be used with any S3 compatible API (ex: GCS, Cloudflare R2, etc) via the [AWS SDK's endpoint url configuration feature](https://docs.aws.amazon.com/sdkref/latest/guide/feature-ss-endpoints.html).

```bash
$ trivy server --cache-backend redis://localhost:6379
```

This approach allows for centralized caching, which can be beneficial in distributed or high-concurrency environments.

If you want to use TLS with Redis, you can enable it by specifying the `--redis-tls` flag.

```bash
$ trivy server --cache-backend redis://localhost:6379 --redis-tls
```

Trivy also supports for connecting to Redis with your certificates.
You need to specify `--redis-ca` , `--redis-cert` , and `--redis-key` options.

```
$ trivy server --cache-backend redis://localhost:6379 \
  --redis-ca /path/to/ca-cert.pem \
  --redis-cert /path/to/cert.pem \
  --redis-key /path/to/key.pem
```

[trivy-db]: ./db.md#vulnerability-database
[trivy-java-db]: ./db.md#java-index-database
[misconf-checks]: ../scanner/misconfiguration/check/builtin.md
[boltdb]: https://github.com/etcd-io/bbolt
[parallel-run]: https://aquasecurity.github.io/trivy/v0.52/docs/references/troubleshooting/#running-in-parallel-takes-same-time-as-series-run

[^1]: Downloaded when scanning for vulnerabilities
[^2]: Downloaded when scanning `jar/war/par/ear` files
[^3]: Downloaded when scanning for misconfigurations
