# Building a Serverless Image Thumbnail Generator with AWS Lambda

A hands-on AWS lab where I built a fully event-driven, serverless image processing pipeline: upload a photo to S3, and a Lambda function automatically resizes it into a thumbnail without any server ever being provisioned or managed.

## Scenario
The goal was to understand AWS Lambda's core value proposition — code that runs automatically in response to events, with AWS managing all the compute underneath it. The chosen example: a classic serverless pattern, image thumbnail generation triggered by an S3 upload.

## How it works
1. A user uploads an image to a source S3 bucket
2. S3 detects the object-created event
3. S3 invokes a Lambda function, passing the event data (bucket name, object key) as a parameter
4. The Lambda function downloads the image, resizes it using the Pillow library, and uploads the resized version to a separate output bucket

## What I did

### 1. Created two S3 buckets
One source bucket for original uploads, and one `-resized` destination bucket for processed thumbnails — uploaded a test image to the source bucket to use as sample event data later.

### 2. Built the Lambda function
Created a Python 3.12 function from scratch, attached to a custom execution role (scoped specifically to read/write the two S3 buckets) and to a VPC, subnet, and dedicated security group for network isolation. Loaded the actual function code — which downloads the uploaded image, resizes it to a 128x128 thumbnail with Pillow, and uploads the result — directly from a packaged .zip in S3, and set the correct handler so Lambda knew which function to invoke.

### 3. Wired up the S3 trigger
Added an S3 trigger on the source bucket for all object-create events, so any new upload would automatically invoke the function — no polling, no manual intervention.

### 4. Tested with a simulated event
Used Lambda's built-in test feature with an S3 Put event template, edited to point at the actual bucket and uploaded file, and ran it manually to confirm the function executed successfully end to end — then verified the resized thumbnail had actually landed in the output bucket.

### 5. Monitored execution with CloudWatch
Reviewed Lambda's built-in metrics — invocations, duration, error count, throttles, concurrent executions — then dug into the actual CloudWatch log stream to see request IDs, execution duration, billed duration, and memory usage for a specific invocation.

## Key takeaways
- The entire pipeline never required provisioning or managing a single server — Lambda handles scaling, availability, and compute allocation automatically based on the event volume
- Attaching a Lambda function to a VPC is an extra layer of network isolation, even though it's not required by default — a good example of defense in depth applied to serverless compute
- CloudWatch Logs turn a black-box function execution into something fully debuggable — memory usage and duration numbers are what you'd actually tune performance and cost against in a real deployment

## Tools
AWS Lambda, Amazon S3, Amazon CloudWatch, Python (Pillow), IAM, VPC

---
*Completed as an AWS hands-on lab, including a passed knowledge check assessment.*
