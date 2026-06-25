# Cost Estimation: Serverless Image Handler

Use this template to estimate the monthly running cost of the Serverless Image Handler solution based on your expected traffic and image processing volume.

---

## Assumptions

Fill in your workload parameters:

| Parameter | Your Estimate |
|-----------|--------------|
| Total image requests per month | _____________ |
| Average original image size | _____________ MB |
| Average processed image size | _____________ MB |
| Expected CloudFront cache-hit ratio | _____________ % |
| Average Lambda execution time | _____________ ms |
| Lambda memory allocation | 1536 MB |
| Region | us-east-1 |

---

## Cost Breakdown by Service

| Service | Pricing Model | Estimated Monthly Usage | Unit Cost | Monthly Cost |
|---------|--------------|------------------------|-----------|--------------|
| **Lambda** | Per-request + duration | 100,000 invocations x 800ms avg @ 1536MB | $0.0000166667/GB-s + $0.20/1M requests | $2.22 |
| **CloudFront** | Data transfer + requests | _________ requests, _________ GB transfer | $0.085/GB (first 10TB) + $0.0075/10K HTTPS requests | $_______ |
| **S3 (Storage)** | Per-GB stored | _________ GB of original images | $0.023/GB/month | $_______ |
| **S3 (Requests)** | Per-request (GET) | _________ GET requests (cache misses only) | $0.0004/1,000 GET requests | $_______ |
| **API Gateway** | Per-request | _________ requests (cache misses only) | $3.50/1M requests | $_______ |
| **CloudWatch Logs** | Ingestion + storage | _________ GB/month | $0.50/GB ingested + $0.03/GB stored | $_______ |
| | | | **Total** | **$_______** |

---

## Worked Example: 500,000 Images/Month

Assumptions: 500K total requests, 80% cache-hit ratio (100K Lambda invocations), 0.5MB average output size, 800ms average processing time.

| Service | Calculation | Monthly Cost |
|---------|-------------|--------------|
| Lambda (requests) | 100,000 / 1,000,000 x $0.20 | $0.02 |
| Lambda (duration) | 100,000 x 0.8s x 1.5GB x $0.0000166667 | $2.00 |
| CloudFront (transfer) | 500,000 x 0.5MB = 250GB x $0.085 | $21.25 |
| CloudFront (requests) | 500,000 / 10,000 x $0.0075 | $0.38 |
| S3 (storage, 50GB originals) | 50 x $0.023 | $1.15 |
| S3 (GET requests) | 100,000 / 1,000 x $0.0004 | $0.04 |
| API Gateway | 100,000 / 1,000,000 x $3.50 | $0.35 |
| CloudWatch Logs | ~2GB x $0.50 | $1.00 |
| **Total** | | **$26.19** |

---

## Build vs. Buy Comparison

Compare the AWS Solution against building your own image processing pipeline:

| Factor | AWS Solution (Serverless Image Handler) | Self-Built (EC2/ECS + ImageMagick) |
|--------|----------------------------------------|-------------------------------------|
| Monthly infrastructure cost | $_______ (from above) | $_______ |
| Development time (initial) | 1-2 hours (deploy + configure) | 40-80 hours (estimate) |
| Maintenance burden | AWS-managed updates, patched by AWS | You own patching, scaling, monitoring |
| Scaling | Automatic (Lambda + CloudFront) | Manual (ASG policies, capacity planning) |
| Availability | Multi-AZ Lambda + global CloudFront | Depends on your architecture |
| Cold start latency | 500-2000ms (first request only) | None (always running), but costs more idle |
| Max image size | 6MB Lambda payload limit | Limited by instance memory |
| Customization | Moderate (Sharp options + Lambda code) | Full control |
| Vendor lock-in | High (CloudFront + Lambda + S3) | Low (portable containers) |

### When the AWS Solution wins:
- Traffic is variable or unpredictable (pay-per-use beats always-on)
- Team doesn't want to maintain image processing infrastructure
- Requirements fit within Sharp's capabilities
- Global low-latency delivery is important

### When self-built wins:
- Extremely high volume (millions/day) where reserved EC2 is cheaper
- Complex processing pipelines (video, ML-based enhancement)
- Images exceed Lambda payload limits (>6MB processed output)
- Strict latency requirements that can't tolerate cold starts

---

## Pricing Links

- [AWS Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
- [Amazon CloudFront Pricing](https://aws.amazon.com/cloudfront/pricing/)
- [Amazon S3 Pricing](https://aws.amazon.com/s3/pricing/)
- [Amazon API Gateway Pricing](https://aws.amazon.com/api-gateway/pricing/)
- [Amazon CloudWatch Pricing](https://aws.amazon.com/cloudwatch/pricing/)

---

## Cost Optimization Tips

1. **Maximize cache-hit ratio** - Set appropriate Cache-Control headers; most images don't change after upload
2. **Use CloudFront Price Class** - Restrict to cheaper regions if your users are concentrated geographically
3. **Right-size Lambda memory** - Test with different allocations; sometimes more memory = faster execution = lower cost
4. **Enable S3 Intelligent-Tiering** - For source buckets with infrequently accessed originals
5. **Set CloudFront TTL aggressively** - 30 days+ for immutable images (use versioned keys for updates)
