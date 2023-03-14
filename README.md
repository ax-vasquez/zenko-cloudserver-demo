# zenko/cloudserver Docker-Compose demo

This repo simply demonstrates how to configure zenko/cloudserver via `docker-compose`.

**Resources**
* [Scality - zenko/cloudserver docs](https://s3-server.readthedocs.io/en/latest/index.html)
* https://www.zenko.io/

## Why use this?

It's effectively a free playground where you can learn and experiment with S3 without incurring expenses from AWS. Since you host the
instance yourself, the only limitation is your data storage limits.

## First-time setup

From the root, run `docker-compose up -d`. Then, configure your local `aws` installation with the credentials set in the `docker-compose.yml`
file.

`aws configure` values:
|  |  |
| - | - |
| **Access Key** | The same value used for `SCALITY_ACCESS_KEY_ID` |
| **Secret Access Key** | The same value used for `SCALITY_SECRET_ACCESS_KEY` |
| **Region** | "`us-east-1`" (default) |
| **Output format** | *None* (default) |

## Creating a bucket

Before you can store objects, you'll need to create a bucket to place them in. To create a bucket, run the following command:

```bash
aws s3api create-bucket --bucket BUCKET_NAME --endpoint=http://localhost:8000
```
* Set `BUCKET_NAME` to whatever you want
* `--endpoint=...` is the key to this command - it just points the AWS command to our zenko/cloudserver instance instead of AWS

## Uploading files via the CLI

**Since you are using the `aws` CLI, there is no need to update CORS/policy for the bucket to perform these steps.**

To upload a single file to a bucket
```bash
aws s3 cp /path/to/LOCAL_FILE s3://BUCKET_NAME/ --endpoint=http://localhost:8000
```

If your command was successful, you'll see output similar to the following:

```bash
upload: test_images/dog_1.jpeg to s3://test/dog_1.jpeg
```

## Syncing files via the CLI

**Since you are using the `aws` CLI, there is no need to update CORS/policy for the bucket to perform these steps.**

1. `cd` into the directory you wish to sync the contents to
2. Run `aws s3 --endpoint-url=http://localhost:8000 sync s3://BUCKET_NAME .`
3. All files from the bucket should now be downloaded
4. You can re-run the command at any time to get _only_ the new files (`sync` doesn't create duplicates)

## Configuire CORS for bucket

These steps are necessary if you want to use the S3 server to store objects for an application. **This command grants the most-permissive
CORS policy and is only intended for demonstration purposes**:

```bash
aws s3api --endpoint-url=http://localhost:8000 put-bucket-cors --bucket BUCKET_NAME --cors-configuration "{ \"CORSRules\": [{ \"AllowedOrigins\": [\"*\"], \"AllowedMethods\":[\"GET\", \"PUT\", \"POST\", \"DELETE\"], \"AllowedHeaders\": [\"*\"] }]}"
```
* This command enables global GET, PUT, POST and DELETE permissions
    * WARNING: **Definitely** don't do this in production - this is just for demo purposes
    * In the real-world, access should be as granular and restrictive as possible

After running this command, a locally-running application will be able to upload, delete and retrieve assets from your local bucket.

## Configure the bucket policy

If you've only completed the CORS configuration, you will not be able to view the asset in-browser, such as an image. Any attempt to
do so will result in an "Access Denied" error. In order to do make an asset publicly-available, you'll also need to update the policy for your 
bucket.

**As with the CORS policy demo, this command is also very permissive and is only done this way for demonstration purposes. Always be mindful
of security concerns when implementing CORS _and_ policy changes.**

```bash
aws s3api --endpoint-url=http://localhost:8000 put-bucket-policy --bucket BUCKET_NAME --policy "{ \"Version\": \"2012-10-17\", \"Statement\":[{ \"Sid\":\"EnablePublicAccesToBucket\", \"Effect\":\"Allow\", \"Principal\":\"*\", \"Action\":[\"s3:GetObject\"], \"Resource\":\"arn:aws:s3:::BUCKET_NAME\/*\" }]}"
```

Once you've run this command, you'll _at least_ be able to download files from your locally-hosted bucket in your browser using the following
URL format:

> **`http://<BUCKET_NAME>.<S3_HOST>/<ASSET_KEY>`**
* Where `S3_HOST` would be `localhost:8000` if using the defaults in this repo

