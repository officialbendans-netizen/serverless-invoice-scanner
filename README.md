# 📄 Serverless Invoice Scanner

> A fully serverless, cloud-native invoice processing application built on **7 AWS services**.  
> Upload a PDF invoice — the system automatically extracts all text and displays it line by line in the browser.

---

## 🌐 Live Application

**👉 [https://dev.d276ks6kki9t7u.amplifyapp.com/](https://dev.d276ks6kki9t7u.amplifyapp.com/)**

---

## 👨‍💻 Built By

**Benjamin Asare Danquah**  
Amalitech Training Program | August 2026  
📍 Ghana 🇬🇭

---

## 🎯 Problem Statement

Small businesses in Ghana and across Africa process invoices manually — reading paper documents and typing data by hand. This is slow, error-prone and impossible to scale.

**The Serverless Invoice Scanner solves this** by automatically receiving invoice files, extracting all text content using PDF processing libraries and displaying the results instantly in the browser — with zero manual data entry required.

---

## ✅ What The System Does

Upload any PDF invoice and within seconds the system:

- Stores the file securely in **Amazon S3**
- Automatically triggers **AWS Lambda** via S3 event
- Extracts all text using **PDF processing libraries** inside Lambda
- Saves structured results to **Amazon DynamoDB**
- Displays every extracted line in the browser — numbered and clean

**Real output example — 85 lines extracted from a single invoice:**
```
01  GHANA TECH SOLUTIONS LTD
02  No. 5 Independence Avenue, Accra, Ghana
03  Tel: +233 30 277 8899 | Email: billing@ghanatechsolutions.com
04  TIN: GH-1234567890 | VAT Reg: VR-0099887
05  TAX INVOICE
06  Invoice Number:
07  GTS-2026-0891
08  Invoice Date:
09  August 21, 2026
...85 lines total
```

---

## ☁️ AWS Architecture

```
USER
  ↓
AWS Amplify ── Hosts frontend globally via CDN with HTTPS
  ↓
Amazon Cognito ── User signup, email verification and login
  ↓
Amazon API Gateway ── REST API routing all requests to Lambda
  ↓
AWS Lambda (Mode 1) ── Generates pre-signed S3 upload URL
  ↓
Amazon S3 ── File uploaded directly from browser via pre-signed URL
  ↓  (S3 PUT event fires automatically)
AWS Lambda (Mode 2) ── Downloads file, extracts text, saves to DynamoDB
  ↓                ↓
Amazon DynamoDB    Amazon CloudWatch
Stores results     Monitors and logs all executions
  ↓
Frontend polls DynamoDB → displays extracted text line by line
```

---

## 🛠️ AWS Services Used

| Service | Role | Key Feature Used |
|---|---|---|
| **AWS Amplify** | Host frontend web application globally | CDN, HTTPS, drag-and-drop deployment |
| **Amazon Cognito** | User authentication | Signup, email verification, JWT tokens |
| **Amazon API Gateway** | REST API management | Lambda proxy integration, CORS, routing |
| **AWS Lambda** | Serverless compute — brain of the system | S3 event trigger + API handler, Python 3.12 |
| **Amazon S3** | Invoice file storage | Pre-signed URLs, PUT event trigger, encryption |
| **Amazon DynamoDB** | Store extracted text and metadata | On-demand NoSQL, millisecond performance |
| **Amazon CloudWatch** | Monitoring and logging | Lambda execution logs, error tracking |

---

## 🔑 Key Architecture Decisions

### Pre-Signed URLs for File Upload
Files upload **directly from the browser to S3** — bypassing API Gateway completely.

```
❌ Avoid:  Browser → API Gateway → Lambda → S3
✅ Correct: Browser → S3 directly via pre-signed URL
```

This avoids the API Gateway 10MB payload limit and follows the AWS recommended pattern.

### Event-Driven Processing
S3 automatically fires a PUT event when a file uploads — Lambda processes it instantly without polling or manual triggers.

### PDF Text Extraction
Lambda uses PDF processing libraries packaged inside the deployment ZIP to extract text directly — no external API subscription required.

### Serverless First
Zero servers to manage. Every service scales automatically. Cost is near zero when idle.

---

## 🚀 How to Use the Application

**1 — Open** → [https://dev.d276ks6kki9t7u.amplifyapp.com/](https://dev.d276ks6kki9t7u.amplifyapp.com/)

**2 — Create account** → Click Create one → Enter name, email and password

**3 — Verify email** → Enter 6-digit code sent by AWS Cognito to your inbox

**4 — Sign in** → Use your email and password

**5 — Upload invoice** → Click Choose Invoice File → Select any PDF invoice

**6 — Watch the pipeline run:**
```
🔑 Getting secure upload URL from AWS Lambda...
📤 Uploading invoice directly to Amazon S3...
🔍 Lambda triggered — extracting text from PDF...
💾 Saving results to Amazon DynamoDB...
✅ Text extracted and displayed successfully!
```

**7 — See results** → Every line from the invoice displayed and numbered in the browser

---

## 📁 Project Structure

```
serverless-invoice-scanner/
│
├── index.html                          # Complete frontend
│   ├── Cognito authentication          # Signup, verify, login
│   ├── Pre-signed S3 upload            # Direct browser to S3
│   └── Results display with retry      # Polling and text rendering
│
├── lambda-deployment.zip               # Lambda function package
│   ├── lambda_function.py              # Python 3.12 handler
│   └── PDF libraries                   # Text extraction dependencies
│
└── README.md                           # This file
```

---

## ⚙️ Lambda Function — Two Modes

```python
def lambda_handler(event, context):

    # MODE 1 — API Gateway GET request
    # Returns pre-signed S3 URL for direct file upload
    if event.get('httpMethod') == 'GET':
        → Generate unique invoiceId
        → Create pre-signed S3 URL (valid 5 minutes)
        → Return URL to frontend

    # MODE 2 — S3 PUT event trigger
    # Automatically fires when file is uploaded
    if event.get('Records'):
        → Download file from S3
        → Extract all text using PDF library
        → Save results to DynamoDB
        → Frontend retrieves and displays results
```

---

## 🗄️ DynamoDB Schema

```json
{
  "invoiceid":     "16c0d01b-7355-4a1b-b99a-16981fe628ff",
  "fileName":      "ghana_tech_invoice.pdf",
  "s3Key":         "invoices/{invoiceId}/ghana_tech_invoice.pdf",
  "extractedText": "GHANA TECH SOLUTIONS LTD\nNo. 5 Independence Avenue...",
  "totalLines":    85,
  "uploadDate":    "2026-08-31T10:02:58",
  "status":        "processed"
}
```

---

## 🔒 Security Implementation

| Layer | Implementation |
|---|---|
| **Authentication** | AWS Cognito — JWT tokens on every API request |
| **File Storage** | S3 bucket — fully private, zero public access |
| **Data Transfer** | HTTPS everywhere — Amplify, API Gateway, S3 |
| **API Protection** | API Gateway — validates Authorization header |
| **Encryption** | S3 SSE-S3 — all files encrypted at rest |

---

## 🐛 Challenges Solved

| Challenge | Root Cause | Solution |
|---|---|---|
| CORS blocking API calls | Lambda proxy integration was False | Enabled Lambda proxy on GET and POST methods |
| Lambda never triggering | S3 trigger used Prefix not Suffix | Recreated trigger with Suffix .pdf |
| DynamoDB rejecting records | invoiceId vs invoiceid case mismatch | Matched partition key exactly in Lambda code |
| File not uploading to S3 | Frontend sent filename text not file bytes | Redesigned to use S3 pre-signed URLs |
| Cognito login failing | USER_PASSWORD_AUTH not enabled | Enabled auth flow in App Client settings |
| PDF text not extracting | PDF compression format incompatible | Used correct PDF library with full dependency package |

---

## 🔮 Future Improvements

| Priority | Improvement | AWS Service |
|---|---|---|
| 1 — Critical | Web Application Firewall | AWS WAF + CloudFront |
| 2 — High | Custom domain with HTTPS | Route 53 + ACM |
| 3 — High | Email notifications on processing | Amazon SES |
| 4 — Medium | Invoice search and filter | DynamoDB Query |
| 5 — Medium | AI field extraction — totals, dates | AWS Bedrock |
| 6 — Medium | Spending analytics dashboard | Amazon QuickSight |
| 7 — Low | Mobile application | AWS Amplify Mobile |

---

## 📊 Project Stats

| Metric | Value |
|---|---|
| AWS Services | 7 |
| Architecture | Serverless — event-driven |
| Lines extracted per invoice | Up to 85+ lines |
| Deployment | Fully live and accessible |
| Cost | Near zero — serverless pricing |

---

## 📬 Contact

**Benjamin Asare Danquah**
🇬🇭 Ghana, West Africa

---

*Built with ❤️ and ☁️ AWS Cloud Services*
