# Deployment Exercise: AWS Serverless Image Handler

## Overview

The AWS Serverless Image Handler is a pre-built solution that provides real-time image manipulation through URL parameters. It uses Sharp (a Node.js image processing library) running on Lambda, fronted by CloudFront for global edge caching.

Common use cases:
- Generating thumbnails on-the-fly from high-resolution originals
- Applying transformations (resize, crop, rotate, watermark) without storing multiple versions
- Serving optimized image formats (WebP, AVIF) based on browser support
- Reducing storage costs by keeping only originals in S3

The solution eliminates the need to pre-generate image variants, saving both compute time and storage costs.

---

## Pre-Deployment Checklist

- [ ] AWS account with admin permissions (or sufficient IAM permissions for CloudFormation, Lambda, S3, CloudFront, API Gateway)
- [ ] An existing S3 bucket with test images, OR create one during deployment
- [ ] AWS CLI configured with appropriate credentials
- [ ] A test image (JPEG or PNG, at least 1000x1000px for visible transformation results)

---

## Step 1: Deploy the CloudFormation Stack

1. Navigate to the AWS Solutions Library page for Serverless Image Handler:
   ```
   https://aws.amazon.com/solutions/implementations/serverless-image-handler/
   ```

2. Click **Launch in the AWS Console** (or use the CLI approach below).

3. **CLI deployment** (alternative to console):
   ```bash
   # Download the template
   curl -O https://solutions-reference.s3.amazonaws.com/serverless-image-handler/latest/serverless-image-handler.template

   # Deploy (replace YOUR_BUCKET with your source image bucket name)
   aws cloudformation create-stack \
     --stack-name serverless-image-handler \
     --template-body file://serverless-image-handler.template \
     --parameters \
       ParameterKey=SourceBuckets,ParameterValue="YOUR_BUCKET" \
       ParameterKey=DeployDemoUI,ParameterValue="Yes" \
       ParameterKey=LogRetentionPeriod,ParameterValue="1" \
     --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
     --region us-east-1
   ```

4. Wait for the stack to reach `CREATE_COMPLETE` (approximately 10-15 minutes):
   ```bash
   aws cloudformation wait stack-create-complete --stack-name serverless-image-handler
   ```

5. Retrieve the CloudFront distribution URL:
   ```bash
   aws cloudformation describe-stacks \
     --stack-name serverless-image-handler \
     --query 'Stacks[0].Outputs[?OutputKey==`ImageHandlerDistribution`].OutputValue' \
     --output text
   ```

---

## Step 2: Upload Test Images

If you don't already have images in your source bucket:

```bash
# Upload a test image
aws s3 cp test-image.jpg s3://YOUR_BUCKET/images/test-image.jpg

# Upload a second image for comparison
aws s3 cp landscape.png s3://YOUR_BUCKET/images/landscape.png
```

---

## Step 3: Verify Transformations

The Image Handler uses a JSON request object encoded in Base64 as the URL path. Build requests as follows:

### Resize an image to 200x200

```bash
# Create the request JSON
echo '{"bucket":"YOUR_BUCKET","key":"images/test-image.jpg","edits":{"resize":{"width":200,"height":200,"fit":"cover"}}}' | base64

# Use the encoded string as the path:
# https://DISTRIBUTION_ID.cloudfront.net/BASE64_ENCODED_STRING
```

### Convert to WebP format

```bash
echo '{"bucket":"YOUR_BUCKET","key":"images/test-image.jpg","outputFormat":"webp"}' | base64
```

### Apply grayscale filter

```bash
echo '{"bucket":"YOUR_BUCKET","key":"images/test-image.jpg","edits":{"grayscale":true}}' | base64
```

### Verify each transformation

```bash
DIST_URL="https://YOUR_DISTRIBUTION.cloudfront.net"

# Fetch the resized image and check dimensions
curl -s -o resized.jpg "${DIST_URL}/$(echo '{"bucket":"YOUR_BUCKET","key":"images/test-image.jpg","edits":{"resize":{"width":200,"height":200,"fit":"cover"}}}' | base64 | tr -d '\n')"

# Verify the output file exists and is smaller than the original
ls -la resized.jpg
file resized.jpg
```

### Confirm CloudFront caching is working

```bash
# First request (cache MISS)
curl -s -o /dev/null -w "%{http_code} %{time_total}s" "${DIST_URL}/YOUR_ENCODED_PATH"

# Second request (cache HIT - should be significantly faster)
curl -s -o /dev/null -w "%{http_code} %{time_total}s" "${DIST_URL}/YOUR_ENCODED_PATH"
```

---

## Step 4: Explore the Demo UI (if deployed)

If you set `DeployDemoUI=Yes`, retrieve the demo URL:

```bash
aws cloudformation describe-stacks \
  --stack-name serverless-image-handler \
  --query 'Stacks[0].Outputs[?OutputKey==`DemoUrl`].OutputValue' \
  --output text
```

The demo UI provides a visual interface to experiment with transformations without constructing Base64 URLs manually.

---

## Step 5: Cleanup

Remove all resources to avoid ongoing charges:

```bash
# Empty the CloudFront logs bucket (if logging was enabled)
LOG_BUCKET=$(aws cloudformation describe-stacks \
  --stack-name serverless-image-handler \
  --query 'Stacks[0].Outputs[?OutputKey==`LogsBucket`].OutputValue' \
  --output text 2>/dev/null)

if [ -n "$LOG_BUCKET" ] && [ "$LOG_BUCKET" != "None" ]; then
  aws s3 rm s3://${LOG_BUCKET} --recursive
fi

# Delete the stack
aws cloudformation delete-stack --stack-name serverless-image-handler

# Verify deletion
aws cloudformation wait stack-delete-complete --stack-name serverless-image-handler
echo "Stack deleted successfully"
```

Note: Your source S3 bucket and its images are NOT deleted (they existed before the solution was deployed).

---

## Exploration Challenges

After completing the basic deployment and verification, try these:

### Challenge 1: Add a Watermark

Apply a text or image overlay to all processed images. Create a request that composites a semi-transparent PNG logo onto a source image using the Sharp `composite` edit.

```json
{
  "bucket": "YOUR_BUCKET",
  "key": "images/test-image.jpg",
  "edits": {
    "composite": [{
      "bucket": "YOUR_BUCKET",
      "key": "images/watermark.png",
      "gravity": "southeast"
    }]
  }
}
```

### Challenge 2: Create Named Presets

Instead of encoding full JSON in every URL, configure Thumbor-style URL paths by modifying the solution's configuration. Define presets like `/fit-in/300x300/filters:quality(80)/` that map to specific transformation sets.

### Challenge 3: Smart Cropping with Content Detection

Use Sharp's `attention` or `entropy` strategies to automatically crop images around the most visually interesting region:

```json
{
  "bucket": "YOUR_BUCKET",
  "key": "images/landscape.png",
  "edits": {
    "resize": {
      "width": 400,
      "height": 400,
      "fit": "cover",
      "position": "attention"
    }
  }
}
```

### Challenge 4: Automatic Format Negotiation

Configure CloudFront to pass the `Accept` header to the origin, then modify the Lambda function to detect `image/webp` or `image/avif` support and serve the optimal format automatically (without the client specifying `outputFormat`).

### Challenge 5: Performance Benchmarking

Measure cold-start vs. warm-start latency for the Lambda function:
1. Wait 15+ minutes for the function to go cold
2. Send a request and measure response time (cold start)
3. Immediately send another request (warm start)
4. Compare the difference and determine whether Provisioned Concurrency would be cost-effective for your traffic patterns
