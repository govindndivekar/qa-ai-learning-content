# AWS — Core Services, AI Services, and Testing in the Cloud

> Subject #9 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: basic networking; Python or Node (files 02/04). Estimated time: ~70 hours.
> Target outcome: Deploy real apps to AWS, articulate architecture trade-offs, test cloud systems, and be exam-ready for Cloud Practitioner (CCP / CLF-C02) and well on the way to Solutions Architect Associate (SAA-C03).

---

## 1. Overview & Learning Outcomes

By the end of this subject you should be able to:

- Stand up a secure AWS account with IAM, MFA, and cost controls.
- Design a VPC with public/private subnets, security groups, and NAT.
- Build a serverless app (API Gateway + Lambda + DynamoDB) via AWS CDK or Terraform.
- Run containerized workloads on ECS/Fargate or EKS.
- Operate a relational database on RDS and an object store on S3 with lifecycle and encryption.
- Use Amazon Bedrock and SageMaker for GenAI/ML workloads.
- Write integration tests with LocalStack / Moto, run load tests (Artillery/k6), and wire GitHub Actions to AWS via OIDC.
- Read a question on CCP or SAA and pick the most cost-effective / least-operational-overhead answer.
- Apply the six Well-Architected pillars to a design review.

---

## 2. Detailed Syllabus

### Part A — Core AWS (28 h)

1. Account setup, IAM, billing & cost controls — 4h
2. Global infrastructure & networking (VPC, subnets, SG, NACL) — 5h
3. Compute (EC2, ASG, ALB/NLB, Lambda) — 6h
4. Storage (S3, EBS, EFS, CloudFront) — 4h
5. Databases (RDS, DynamoDB, ElastiCache) — 4h
6. Messaging (SQS, SNS, EventBridge, Kinesis) + Step Functions — 3h
7. Containers (ECR, ECS/Fargate, EKS overview) — 2h

### Part B — Data & AI on AWS (12 h)

8. Analytics (Glue, Athena, Redshift) — 3h
9. Amazon Bedrock (Converse API, Guardrails, Knowledge Bases, Agents) — 4h
10. SageMaker (Studio, JumpStart, endpoints) — 3h
11. AI managed services (Rekognition, Textract, Comprehend, Polly) — 2h

### Part C — Testing & DevOps in AWS (15 h)

12. IaC: CloudFormation, AWS CDK (TS), Terraform — 4h
13. Local dev & testing: LocalStack, Moto, SAM Local, DynamoDB Local — 3h
14. Integration / contract / load / chaos testing — 4h
15. CI/CD: CodePipeline, GitHub Actions + OIDC, CodeDeploy — 2h
16. Observability & security testing (CloudWatch, X-Ray, Inspector, GuardDuty) — 2h

### Part D — Exam Prep (15 h)

17. CCP (CLF-C02) 30-day plan — 8h
18. SAA-C03 60-day plan (started, not finished here) — 7h

---

## 3. Topics — Explanations with Examples

### 3.1 Account setup, IAM, cost controls

**Concept.** The root account is for billing only. Everything else goes through IAM users, groups, and roles. Always enable MFA, set up a budget with email alarms, and use CLI profiles so you never embed keys in code.

**Example (AWS CLI v2).**
```bash
# Configure a named profile — never the default
aws configure --profile govind-dev
# Region: ap-south-1 (Mumbai) or us-east-1
# Key & Secret from IAM → Users → Security Credentials

# Test the identity
aws sts get-caller-identity --profile govind-dev

# Create a simple budget via CLI (better: do it in console first time)
aws budgets create-budget --account-id 123456789012 --budget file://budget.json --profile govind-dev
```

**Key pitfalls.**

- Never hard-code access keys. Prefer IAM roles or SSO.
- Public S3 buckets are the #1 breach vector — keep Block Public Access ON.
- Default region mismatch burns hours; set AWS_REGION in your shell.

### 3.2 IAM policies and roles

**Concept.** An IAM policy is a JSON doc with `Effect`, `Action`, `Resource`, and optional `Condition`. Roles are assumable identities with trust policies; they're how Lambda, EC2, and CI pipelines get credentials without long-lived keys.

**Example — least-privilege S3 read policy.**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:ListBucket"],
    "Resource": [
      "arn:aws:s3:::govind-artifacts",
      "arn:aws:s3:::govind-artifacts/*"
    ],
    "Condition": { "StringEquals": { "aws:SourceIp": "203.0.113.0/24" } }
  }]
}
```

**Key pitfalls.**

- `"Resource": "*"` inside a managed role is a red flag in interviews.
- Don't use long-lived IAM user keys from GitHub Actions — use OIDC with an assumed role.

### 3.3 VPC networking

**Concept.** A VPC is your private IP space. Public subnets have a route to an Internet Gateway; private subnets reach outbound via a NAT Gateway. Security Groups are stateful (allow returns automatically); NACLs are stateless and subnet-level.

**Example (CDK TypeScript).**
```ts
import { Vpc, SubnetType, IpAddresses } from 'aws-cdk-lib/aws-ec2';

new Vpc(this, 'AppVpc', {
  ipAddresses: IpAddresses.cidr('10.10.0.0/16'),
  maxAzs: 2,
  natGateways: 1,
  subnetConfiguration: [
    { name: 'public',  subnetType: SubnetType.PUBLIC,           cidrMask: 24 },
    { name: 'private', subnetType: SubnetType.PRIVATE_WITH_EGRESS, cidrMask: 24 },
  ],
});
```

**Key pitfalls.**

- NAT Gateway is ~$0.045/hr per AZ — multiply by AZs and months.
- VPC Endpoints for S3/DynamoDB save NAT cost and keep traffic private.

### 3.4 EC2 and Auto Scaling

**Concept.** EC2 is a VM. Burstable (t4g, t3), general (m6i/m7g), compute (c7g), memory (r7g), GPU (g5, p4). Use spot for fault-tolerant batch (up to 90% off). Auto Scaling Groups manage capacity behind an ALB.

**Example — start a t3.micro with user data.**
```bash
aws ec2 run-instances \
  --image-id ami-0abcd1234 \
  --instance-type t3.micro \
  --key-name govind-key \
  --security-group-ids sg-0abc123 \
  --subnet-id subnet-0def456 \
  --user-data file://install-nginx.sh
```

**Key pitfalls.**

- Don't treat EC2 like a pet. Auto-build AMIs with Packer or use Fargate.
- User data runs as root — keep it small and idempotent.

### 3.5 Lambda

**Concept.** Serverless compute billed per ms. Cold starts (50–400ms typical, seconds for Java/.NET). Memory also scales CPU: 1769 MB ≈ 1 vCPU. Max 15 min, 10 GB memory, 10 GB `/tmp`. Triggers include API Gateway, S3, SQS, EventBridge, DynamoDB streams.

**Example — Node 20 Lambda via CDK.**
```ts
import { Function, Runtime, Code } from 'aws-cdk-lib/aws-lambda';

const fn = new Function(this, 'HelloFn', {
  runtime: Runtime.NODEJS_20_X,
  handler: 'index.handler',
  code: Code.fromAsset('lambda'),
  memorySize: 512,
  timeout: Duration.seconds(10),
  environment: { LOG_LEVEL: 'info' },
});
```

```ts
// lambda/index.ts
export const handler = async (event: any) => ({
  statusCode: 200,
  body: JSON.stringify({ msg: 'hello', event })
});
```

**Key pitfalls.**

- Cold starts: keep bundles small, prefer Node/Python, use SnapStart for Java.
- Don't put DB passwords in env vars — use Secrets Manager.
- Lambda-in-VPC adds ENI attach time; only do it when you must reach private RDS.

### 3.6 API Gateway

**Concept.** Two flavors: REST APIs (full featured, pricier) and HTTP APIs (cheaper, JWT native, 60–70% cost saving). Pick HTTP API by default; REST API if you need request validation, API keys/usage plans, or WAF deep integration.

**Example — HTTP API + Lambda integration (CDK).**
```ts
import { HttpApi } from 'aws-cdk-lib/aws-apigatewayv2';
import { HttpLambdaIntegration } from 'aws-cdk-lib/aws-apigatewayv2-integrations';

const api = new HttpApi(this, 'HttpApi');
api.addRoutes({
  path: '/hello',
  methods: [HttpMethod.GET],
  integration: new HttpLambdaIntegration('HelloInt', fn),
});
```

**Key pitfalls.**

- Throttling defaults are fine for dev; set custom for prod.
- Caching ($) is on REST API only; usually not worth it vs Lambda@Edge or CloudFront.

### 3.7 S3

**Concept.** Object store priced per GB-month. Storage classes: Standard, IA, IntelligentTiering (default for variable access), Glacier Instant/Flexible/Deep Archive. Features: versioning, lifecycle, SSE-S3/KMS, event notifications, presigned URLs.

**Example — presigned URL (Python boto3).**
```python
import boto3
s3 = boto3.client("s3", region_name="ap-south-1")
url = s3.generate_presigned_url(
    "put_object",
    Params={"Bucket": "govind-uploads", "Key": "report.pdf", "ContentType": "application/pdf"},
    ExpiresIn=600,
)
print(url)  # share with frontend for direct browser upload
```

**Key pitfalls.**

- Don't serve 5 GB files from Lambda → use presigned URLs.
- Turn on `BlockPublicAccess` everywhere unless hosting a static site, and then use CloudFront + OAC.

### 3.8 RDS and DynamoDB

**Concept.** RDS = managed relational (Postgres, MySQL, Oracle, SQL Server, MariaDB, Aurora). DynamoDB = managed NoSQL with single-digit ms reads, pay per request or provisioned. Choose RDS when you need SQL/joins/ACID; DynamoDB when you need huge scale and access patterns are known up front.

**Example — DynamoDB query (boto3).**
```python
import boto3
from boto3.dynamodb.conditions import Key
table = boto3.resource("dynamodb").Table("Orders")
resp = table.query(
    KeyConditionExpression=Key("customer_id").eq("C123") & Key("order_date").gte("2026-01-01"),
    Limit=50,
)
for item in resp["Items"]:
    print(item)
```

**Key pitfalls.**

- Scan on DynamoDB is the enemy of cost and performance.
- Multi-AZ on RDS doubles cost but gives automatic failover.

### 3.9 Messaging: SQS / SNS / EventBridge / Kinesis

**Concept.**

- **SQS** — decouples producers and consumers; Standard (at-least-once) or FIFO (exactly-once per group).
- **SNS** — pub/sub fanout to SQS/Lambda/HTTP/SMS/email.
- **EventBridge** — event bus with schema registry and rules; ideal for event-driven arch.
- **Kinesis Data Streams** — ordered, replayable, high throughput; for analytics pipelines.

**Example — send & receive SQS (boto3).**
```python
sqs = boto3.client("sqs", region_name="ap-south-1")
q = "https://sqs.ap-south-1.amazonaws.com/123456789012/orders"
sqs.send_message(QueueUrl=q, MessageBody='{"order":"1"}')
msgs = sqs.receive_message(QueueUrl=q, MaxNumberOfMessages=10, WaitTimeSeconds=20)
```

**Key pitfalls.**

- Always set `VisibilityTimeout` > max handler processing time.
- SQS long polling (`WaitTimeSeconds=20`) is free and cuts cost.

### 3.10 Step Functions

**Concept.** Managed state machine service. Standard workflows (up to 1 year, audit-grade) vs Express workflows (cheaper, short, at-least-once). Great for orchestrating Lambdas with retries, parallel fan-out, waits.

**Example — wait-then-notify state machine.**
```json
{
  "StartAt": "Wait30",
  "States": {
    "Wait30":  { "Type": "Wait", "Seconds": 30, "Next": "Notify" },
    "Notify":  { "Type": "Task",
                 "Resource": "arn:aws:lambda:ap-south-1:123:function:notify",
                 "End": true }
  }
}
```

### 3.11 ECS/Fargate vs EKS

**Concept.** ECS is AWS-native container orchestration; Fargate is serverless capacity for ECS/EKS. EKS is managed Kubernetes — worth it if you already know k8s or need its ecosystem.

**Rule of thumb.** New team, AWS-only, no k8s expertise → ECS + Fargate. Existing k8s shop or multi-cloud → EKS.

### 3.12 CloudWatch, X-Ray, and the observability story

**Concept.** CloudWatch Logs/Metrics/Alarms/Dashboards; Logs Insights for querying. X-Ray for distributed tracing. `aws.emf` Embedded Metric Format lets you emit metrics via log lines — cheap and clean.

**Example — Logs Insights query.**
```
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() by bin(5m)
```

### 3.13 Infrastructure as Code: CloudFormation, CDK, Terraform

**Concept.** CloudFormation is the native JSON/YAML engine. CDK is a TypeScript/Python/Java layer that compiles to CloudFormation with reusable constructs. Terraform is cloud-agnostic HCL.

**Recommendation for this plan.** Use AWS CDK with TypeScript — it leverages your TypeScript learning and is the fastest-growing option in AWS shops.

### 3.14 Bedrock

**Concept.** Fully managed GenAI API. One endpoint, many providers (Anthropic Claude, Meta Llama, Mistral, Amazon Titan, Cohere, Stability). Key features: Converse API (unified chat), Guardrails (policy enforcement), Knowledge Bases (managed RAG), Agents (tool use), model evaluation.

**Example — Converse call (boto3).**
```python
import boto3, json
client = boto3.client("bedrock-runtime", region_name="us-east-1")
resp = client.converse(
    modelId="anthropic.claude-3-5-sonnet-20241022-v2:0",
    messages=[{"role": "user", "content": [{"text": "Summarize QA in 3 bullet points."}]}],
    inferenceConfig={"maxTokens": 300, "temperature": 0.2},
)
print(resp["output"]["message"]["content"][0]["text"])
```

**Key pitfalls.**

- Not all models are enabled by default — request access per-model in the Bedrock console.
- Prices vary by ~10x between small and frontier models; show costs in any design.

### 3.15 SageMaker

**Concept.** The full ML lifecycle platform. Studio (IDE), Training Jobs, Endpoints (real-time, async, batch), JumpStart (pre-built models), Pipelines, Model Monitor, Feature Store.

**Example — deploy a JumpStart model (boto3 / SDK).**
```python
from sagemaker.jumpstart.model import JumpStartModel
model = JumpStartModel(model_id="huggingface-text2text-flan-t5-base")
predictor = model.deploy()
print(predictor.predict({"inputs": "translate English to German: Hello"}))
```

### 3.16 Local testing: LocalStack and Moto

**Concept.** LocalStack emulates AWS APIs inside Docker — S3, SQS, SNS, Lambda, DynamoDB, Step Functions, etc. Moto patches boto3 calls for unit tests.

**Example — Moto unit test.**
```python
from moto import mock_aws
import boto3

@mock_aws
def test_bucket_created():
    s3 = boto3.client("s3", region_name="us-east-1")
    s3.create_bucket(Bucket="test-bucket")
    assert s3.list_buckets()["Buckets"][0]["Name"] == "test-bucket"
```

**LocalStack up:** `docker run --rm -p 4566:4566 localstack/localstack`. Then point `AWS_ENDPOINT_URL=http://localhost:4566`.

### 3.17 Integration, contract, and load tests in AWS

**Concept.** Four test layers matter for cloud apps:

- **Unit (Moto)** — fast, no AWS.
- **Integration (LocalStack or ephemeral stack)** — real AWS API calls; per-PR stack via CDK.
- **Contract (Pact / Postman / Playwright API tests)** — against API Gateway, in a dev stack.
- **Load (Artillery / k6 / Locust)** — against a staging stack, with CloudWatch dashboards open.

**Example — Artillery load test (yaml).**
```yaml
config:
  target: "https://abc123.execute-api.ap-south-1.amazonaws.com"
  phases:
    - duration: 60
      arrivalRate: 20
scenarios:
  - flow:
      - get: { url: "/hello" }
```

### 3.18 CI/CD and OIDC

**Concept.** Use GitHub Actions + AWS OIDC so GitHub can assume an IAM role without long-lived secrets. CodePipeline/CodeBuild are native alternatives but heavier.

**Example — GitHub Actions OIDC.**
```yaml
permissions:
  id-token: write
  contents: read
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123:role/GhaDeployRole
          aws-region: ap-south-1
      - run: npm ci && npx cdk deploy --require-approval never
```

### 3.19 Security testing and guardrails

**Concept.** Managed services that act as a "free red team":

- **IAM Access Analyzer** — public/cross-account access findings.
- **Inspector** — CVE scanning of EC2, ECR, Lambda.
- **GuardDuty** — anomaly detection on VPC/DNS/CloudTrail.
- **Security Hub** — aggregate findings, map to CIS/PCI benchmarks.

### 3.20 The Well-Architected Framework

**Six pillars.** Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability. Every architecture discussion in an interview should be framed by these.

**Example answer structure for "design a photo-sharing app":** requirements → high-level (S3 + CloudFront + Lambda + DynamoDB + Cognito) → pillar-by-pillar trade-offs → two deep-dive topics.

---

## 4. Exercises (Hands-On)

### Beginner

- **A1.** Create a new AWS account; enable MFA on root; create an IAM user with `PowerUserAccess`; configure CLI profile.
  - *Acceptance:* `aws sts get-caller-identity` returns the IAM user ARN, not root.
- **A2.** Host a static site on S3 + CloudFront with OAC; route it via Route 53 (optional).
- **A3.** Launch a `t3.micro` with user-data that installs nginx; hit its public IP.

### Intermediate

- **B1.** Serverless CRUD: HTTP API + Lambda (Node/TS) + DynamoDB; deploy with CDK; unit-tested with Jest + Moto, integration-tested against a real dev stack.
- **B2.** Build a VPC with public/private subnets, a bastion host, and an RDS Postgres in the private subnet; connect from a Lambda-in-VPC.
- **B3.** S3 → Lambda → SNS email pipeline: upload a file, receive an email with metadata.
- **B4.** Run Artillery against your CRUD API; add a CloudWatch alarm on p95 latency > 500 ms.

### Advanced

- **C1.** Bedrock Knowledge Bases over an S3 corpus; call from a Lambda that returns grounded answers with citations.
- **C2.** Deploy a SageMaker JumpStart endpoint; write a pytest suite that asserts latency and schema.
- **C3.** GitHub Actions OIDC pipeline that deploys a preview CDK stack per pull request; teardown on merge/close.
- **C4.** Chaos drill: use AWS FIS to fail an AZ; verify ALB target-group health and end-to-end resilience.

### Capstone — "Test Platform on AWS"

Two-account setup (Dev / Prod) using AWS Organizations-style controls. CDK/TypeScript infra: VPC, API Gateway, Lambda (Node), DynamoDB, S3, Bedrock-backed summarization endpoint, CloudWatch dashboard, cost alarms, GitHub Actions OIDC deploys, Artillery load tests, and a written Well-Architected review of every pillar. Deliver a README with an architecture diagram, a cost estimate, and a test strategy.

---

## 5. Additional Reading & Resources

### Books
- *AWS Certified Cloud Practitioner Study Guide* (Sybex, Piper/Clinton).
- *AWS Certified Solutions Architect Associate Study Guide* (Sybex, latest ed.).
- *The AWS Well-Architected Framework* (official whitepaper).

### Official docs
- AWS Documentation portal — <https://docs.aws.amazon.com/>
- AWS Skill Builder (free) — <https://skillbuilder.aws/>
- AWS Workshops — <https://workshops.aws/>
- AWS CDK Developer Guide — <https://docs.aws.amazon.com/cdk/v2/guide/home.html>
- Bedrock User Guide — <https://docs.aws.amazon.com/bedrock/>
- SageMaker Developer Guide — <https://docs.aws.amazon.com/sagemaker/>

### Courses
- Adrian Cantrill's *AWS Certified Cloud Practitioner* / *Solutions Architect Associate* (gold standard).
- Stephane Maarek's Udemy CCP and SAA courses.
- *AWS Developer Associate* on A Cloud Guru (for a Dev-flavored view).

### YouTube
- AWS official channel (re:Invent keynotes, "This Is My Architecture").
- Be A Better Dev (Daniel Smith) — AWS deep dives.
- Rahul Shetty Academy — AWS for testers (for QA-specific examples).

### Practice
- Tutorials Dojo practice tests (single best resource for both exams).
- AWS Sample Questions (official) — read every explanation.

---

## 6. Index

- Access Analyzer — §3.19
- ALB / NLB — §3.4
- API Gateway — §3.6
- Artillery load test — §3.17, capstone
- Auto Scaling Groups — §3.4
- Bedrock — §3.14
- budget alarms — §3.1
- CDK — §3.13
- chaos testing (FIS) — §3.19, exercise C4
- CloudFormation — §3.13
- CloudFront — §3.7
- CloudWatch — §3.12
- contract testing — §3.17
- DynamoDB — §3.8
- EC2 — §3.4
- EventBridge — §3.9
- Fargate — §3.11
- GuardDuty — §3.19
- IAM policy — §3.2
- Inspector — §3.19
- Kinesis — §3.9
- KMS — §3.2, §3.7
- Lambda — §3.5
- LocalStack — §3.16
- Moto — §3.16
- NAT Gateway — §3.3
- OIDC with GitHub — §3.18
- RDS — §3.8
- Route 53 — exercise A2
- S3 — §3.7
- SageMaker — §3.15
- Security Hub — §3.19
- SNS / SQS — §3.9
- Step Functions — §3.10
- Terraform — §3.13
- VPC — §3.3
- Well-Architected — §3.20
- X-Ray — §3.12

---

## 7. References

1. AWS Documentation — <https://docs.aws.amazon.com/>
2. AWS Well-Architected Framework whitepaper — <https://aws.amazon.com/architecture/well-architected/>
3. AWS CDK v2 Developer Guide — <https://docs.aws.amazon.com/cdk/v2/guide/home.html>
4. AWS Bedrock User Guide — <https://docs.aws.amazon.com/bedrock/latest/userguide/>
5. AWS SageMaker Developer Guide — <https://docs.aws.amazon.com/sagemaker/latest/dg/>
6. LocalStack Documentation — <https://docs.localstack.cloud/>
7. Moto Documentation — <https://docs.getmoto.org/>
8. Tutorials Dojo — <https://tutorialsdojo.com/>
9. AWS Skill Builder — <https://skillbuilder.aws/>
10. Adrian Cantrill's Learn Cantrill site — <https://learn.cantrill.io/>
11. Stephane Maarek — <https://courses.datacumulus.com/>
12. AWS Workshops — <https://workshops.aws/>
13. *AWS Certified Cloud Practitioner Study Guide*, Wiley / Sybex.
14. AWS Architecture Center — <https://aws.amazon.com/architecture/>

---

*End of Subject 09 — AWS.*
