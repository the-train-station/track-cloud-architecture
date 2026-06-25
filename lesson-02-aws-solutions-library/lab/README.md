# Lab: AWS Solutions Library - Serverless Image Handler

## Objective

Deploy a production-grade AWS Solution from the Solutions Library, understand its architecture, estimate operational costs, and extend it through hands-on challenges.

## Prerequisites

- AWS account with permissions to create CloudFormation stacks, Lambda functions, S3 buckets, CloudFront distributions, and API Gateway endpoints
- AWS CLI configured
- At least one test image (JPEG or PNG, 1000x1000px or larger)
- Basic understanding of serverless architecture concepts

## Exercise Flow

### Step 1: Study the Architecture (15 min)

Open `architecture-diagram.md` and trace the request flow:
1. How does a client request reach the Lambda function?
2. What role does CloudFront play beyond caching?
3. Why is API Gateway in the path instead of invoking Lambda directly?

### Step 2: Deploy the Solution (20 min)

Follow `deploy-exercise.md` to deploy the Serverless Image Handler:
1. Deploy the CloudFormation stack
2. Upload test images to your source bucket
3. Verify transformations work (resize, format conversion, filters)
4. Confirm CloudFront caching behavior

### Step 3: Estimate Costs (20 min)

Open `cost-estimation-template.md` and estimate monthly costs for your use case:
1. Define your traffic assumptions (requests/month, image sizes)
2. Fill in the cost table for each service
3. Compare against the worked example
4. Complete the build-vs-buy analysis for your scenario

### Step 4: Exploration Challenges (30+ min)

Complete at least 2 of the 5 challenges in `deploy-exercise.md`:
- Watermark overlay
- Named presets
- Smart cropping
- Format negotiation
- Performance benchmarking

### Step 5: Clean Up

Follow the cleanup instructions in the deploy exercise to delete all resources and avoid ongoing charges.

## Key Takeaways

- AWS Solutions Library provides vetted, production-ready architectures
- Understanding cost models before deploying prevents billing surprises
- Serverless architectures trade cold-start latency for zero idle cost
- Edge caching (CloudFront) is the primary lever for both performance and cost
