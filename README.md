# A Proxy Server for GCS 

This is Caddy configured to proxy requests to a GCS bucket. I use it to get SSL termination more cheaply than the official document [Hosting a Static Website on GCS](https://cloud.google.com/storage/docs/hosting-static-website). Setting up [HTTPS load balancing](https://cloud.google.com/load-balancing/docs/https) costs about $20 per month which raises my cost from just pennies a month. 

## Automatic SSL Termination on CloudRun 

[Google Cloud Run](https://cloud.google.com/run) is a system for running stateless containers. When your containers are not getting requests they spin down, like AppEngine. Cloud Run services can be [mapped to a custom domain](https://cloud.google.com/run/docs/mapping-custom-domains) with automatic SSL termination handled by Google. 

This container simply rewrites requests and proxies them to Google's servers that handle storage bucket requests. 

> WARNING: It's possible to expose non-public buckets in the same project with this proxy.

## Cost 

Cloud Run has a free tier that makes low traffic sites free to do this way. If you keep an instance running all the time so you don't get cold starts and the cost depends on how much time it spends idle, so it's still pretty cheap. I just tried this so I don't have data yet!

## Limitations 

Sites served this way are served from a single Google region. 

## How To 

### Step 1: Create the Container 

Here's the guide to [deploying containers](https://cloud.google.com/run/docs/deploying).

Change `my-bucket-name` to any GCS bucket name in the next command to run the proxy: 

```
gcloud run deploy caddy-proxy \
    --image docker.io/mikematera/caddy-gcs-proxy:latest \
    --set-env-vars=STORAGE_BUCKET_NAME=my-bucket-name
```

You can proxy any public bucket and possibly private buckets in the same project. I haven't tried that. 

### Step 2: Map the Domain 

Here's the guide to [mapping custom domains](https://cloud.google.com/run/docs/mapping-custom-domains). 

Update this command with your domain: 

```
gcloud beta run domain-mappings create --service caddy-proxy --domain www.yourdomain.com 
```

> Google has to validate your domain before this works. 

