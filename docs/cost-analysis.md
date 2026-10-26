# Cost Analysis & Optimization Report — OpenNotetaker

## 1. Estimated AWS Infrastructure Cost Breakdown

*(Assumed exchange rate: 1 USD = 84.00 INR; AWS Region: `ap-south-1` Mumbai)*

| Service | Component & Sizing | Monthly Cost (USD) | Monthly Cost (INR) |
| :--- | :--- | :--- | :--- |
| **AWS EC2** | `t3.medium` (2 vCPU, 4 GB RAM) | $30.37 | ₹2,551 |
| **Amazon EBS** | 30 GB gp3 General Purpose SSD | $2.88 | ₹242 |
| **Amazon ECR** | 2 GB Container Image Storage | $0.20 | ₹17 |
| **CloudWatch** | Basic logs & metrics (within free tier) | $2.00 | ₹168 |
| **Data Transfer** | 10 GB Outbound Data Transfer | $1.09 | ₹92 |
| **Elastic IP** | 1 IPv4 address (attached to EC2) | $0.00 | ₹0 |
| **TOTAL** | **Estimated Monthly Cost** | **~$36.54** | **~₹3,070** |

---

## 2. External AI API Usage Costs (Pay-As-You-Go)

- **Deepgram STT (`nova-3` Multi):** ~$0.0043 per minute of recorded meeting audio.
- **Google Gemini 1.5 Flash:** ~$0.00001875 per 1k input tokens (negligible for typical meeting summaries).

---

## 3. Cost Optimization Strategies

1. **Right-sizing:** Use `t3.medium` for single-worker deployments; scale to `t3.large` only if running >2 concurrent recording workers.
2. **EC2 Savings Plans:** Opting into 1-year Compute Savings Plan reduces EC2 cost by ~30-40%.
3. **Log Retention:** Configure CloudWatch log retention policy to 14 days to prevent log storage accumulation.
4. **ECR Image Pruning:** Set up ECR Lifecycle Policies to keep only the latest 5 image builds.
