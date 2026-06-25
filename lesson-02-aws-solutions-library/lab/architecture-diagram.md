# Serverless Image Handler: Architecture Diagram

## High-Level Architecture

```mermaid
graph LR
    Client[Client Browser] -->|HTTPS Request<br/>with encoded params| CF[CloudFront<br/>Distribution]
    
    CF -->|Cache HIT| Client
    CF -->|Cache MISS| APIGW[API Gateway]
    
    APIGW -->|Invoke| Lambda[Lambda Function<br/>Sharp Image Processing]
    
    Lambda -->|GetObject<br/>Original Image| S3Origin[(S3 Source Bucket<br/>Original Images)]
    Lambda -->|Processed Image| APIGW
    
    APIGW -->|Response| CF
    CF -->|Cache & Serve| Client
```

## Detailed Data Flow

```mermaid
sequenceDiagram
    participant Client
    participant CloudFront
    participant APIGateway as API Gateway
    participant Lambda
    participant S3 as S3 Source Bucket

    Client->>CloudFront: GET /eyJidWNrZXQiOiIuLi4= (Base64 encoded request)
    
    alt Cache HIT
        CloudFront-->>Client: Return cached image (< 10ms)
    else Cache MISS
        CloudFront->>APIGateway: Forward request to origin
        APIGateway->>Lambda: Invoke image handler function
        
        Note over Lambda: Decode Base64 path<br/>Parse transformation params
        
        Lambda->>S3: GetObject (original image)
        S3-->>Lambda: Image binary data
        
        Note over Lambda: Apply Sharp transformations:<br/>resize, crop, format convert,<br/>watermark, filters
        
        Lambda-->>APIGateway: Processed image + headers
        APIGateway-->>CloudFront: Image response with Cache-Control
        CloudFront-->>Client: Serve image + cache at edge
    end
```

## Component Details

```mermaid
graph TB
    subgraph Edge["Edge Layer (Global)"]
        CF[CloudFront Distribution<br/>Cache TTL: 24h default]
    end

    subgraph Origin["Origin Layer (Regional)"]
        APIGW[API Gateway<br/>REST API<br/>Binary media support enabled]
        Lambda[Lambda Function<br/>Node.js runtime<br/>Sharp library<br/>Memory: 1536MB<br/>Timeout: 60s]
    end

    subgraph Storage["Storage Layer (Regional)"]
        S3Source[(S3 Source Bucket<br/>Original images<br/>Versioning enabled)]
        S3Logs[(S3 Logs Bucket<br/>CloudFront access logs)]
    end

    subgraph Security["Security & Access"]
        OAI[Origin Access Identity<br/>CloudFront → API Gateway]
        Role[Lambda Execution Role<br/>S3 read-only access]
        WAF[AWS WAF<br/>Optional rate limiting]
    end

    CF --> APIGW
    APIGW --> Lambda
    Lambda --> S3Source
    CF --> S3Logs
    CF -.-> WAF
    Lambda -.-> Role
    CF -.-> OAI
```

## Request Encoding Format

The URL path carries the full transformation specification as Base64-encoded JSON:

```
https://<distribution>.cloudfront.net/<base64-encoded-json>
```

Decoded JSON structure:

```json
{
  "bucket": "source-bucket-name",
  "key": "path/to/image.jpg",
  "outputFormat": "webp",
  "edits": {
    "resize": { "width": 400, "height": 300, "fit": "cover" },
    "grayscale": true,
    "rotate": 90
  }
}
```

## Cost Flow

```mermaid
graph LR
    subgraph Per-Request Costs
        CF_Cost[CloudFront<br/>$0.085/GB transfer<br/>$0.0075/10K requests]
        Lambda_Cost[Lambda<br/>$0.0000166667/GB-s<br/>Only on cache MISS]
        S3_Cost[S3<br/>$0.0004/1K GETs<br/>Only on cache MISS]
    end

    subgraph Fixed Costs
        S3_Storage[S3 Storage<br/>$0.023/GB/month<br/>Original images only]
    end

    Request --> CF_Cost
    CF_Cost -->|MISS only| Lambda_Cost
    Lambda_Cost --> S3_Cost
```

The CloudFront cache is the primary cost optimization lever. Higher cache-hit ratios mean fewer Lambda invocations and S3 reads.
