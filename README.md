# s3cf-deploy

A Python script to sync a local folder with S3 and then generate a CloudFront invalidation based on the difference.

I previously had some Bash monstrosity to do the same thing, but as is often the case with Bash, it was brittle and
hard to maintain consistently across distributions and versions of Bash. This script is very vanilla Python 3 and has
no external dependencies apart from the AWS CLI. So long as `aws` is on your `PATH`, this thing should do the needful.

## Usage

### Simple Synchronization

Sync the local `_site` directory to the
`mybucket` S3 bucket under `/blog/`, generating an invalidation for the given CloudFront distribution:

```
s3cf-deploy -c A3KNEPBLWRNDQ _site mybucket /blog/
```

### Advanced Synchronization

Same as before, but now pass custom arguments to `aws s3 sync` using the `S3_OPTS` environment variable:

```
S3_OPTS="--delete --sse AES256" s3cf-deploy -c $CF_DIST_ID public/ mybucket
```

> **NOTE**: `shlex.split` is used to break apart arguments defined in `S3_OPTS`.

As noted in the program help output, the default remote path to sync to is `/`, the root of the bucket.

### Global Invalidations

Global invalidations are possible with CloudFront in order to avoid charges for individual invalidations. CloudFront
[starts charging for invalidations](https://aws.amazon.com/cloudfront/pricing/) after the first 1,000 path invalidations
requested in a month, so if you're invalidating a lot of paths consistently, it can get expensive to invalidate specific
paths as opposed to using a wildcard invalidation, which counts only as a single path. Global invalidations can be
enabled with the `-a` flag:

```
s3cf-deploy -a -c A3KNEPBLWRNDQ _site mybucket /blog/
```
