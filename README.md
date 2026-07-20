# Securing a Web App with Amazon Cognito

A project implementing authentication and authorization for a serverless web application ("Birds", a bird-sighting tracker) using **Amazon Cognito User Pools** and **Identity Pools**.

## What this demonstrates
- Configuring a Cognito **User Pool** as an identity provider (hosted login UI, OAuth 2.0 grant types, OIDC scopes, password auth flows)
- Managing users and **role-based access** via Cognito Groups (regular users vs. Administrators)
- Configuring a Cognito **Identity Pool** to exchange authenticated user tokens for temporary, scoped AWS credentials
- Wiring a static front-end (S3 + CloudFront) and a Node.js/Express API to validate Cognito JWTs and use temporary IAM credentials
- Using temporary AWS credentials from the browser to securely query **Amazon DynamoDB**, without embedding long-lived AWS keys in client-side code

## Architecture
Browser (S3 + CloudFront static site)
│
├── Auth: Cognito User Pool (hosted login, JWT issuance)
│
├── API calls → Node.js/Express server (validates JWT via cognito-express)
│
└── AWS SDK calls → Cognito Identity Pool → temporary IAM credentials
│
└── DynamoDB (BirdSightings table)

**Flow:**
1. A user logs in via the Cognito-hosted login page (User Pool). Cognito issues a JWT.
2. Protected pages/API routes validate that JWT server-side before returning data.
3. When a logged-in user needs direct AWS access (e.g., reading DynamoDB from the browser), the app exchanges the JWT with the Identity Pool for short-lived, scoped AWS credentials — no static AWS keys are ever shipped to the client.
4. An Administrators Cognito Group gates access to an admin-only page, driven by group claims in the JWT.

## Key configuration
- **User Pool**: username + email sign-up, `ALLOW_USER_PASSWORD_AUTH` flow, hosted login with Authorization Code + Implicit grants, OpenID/Email scopes
- **App Client**: configured with a callback URL pointing to the CloudFront distribution
- **Identity Pool**: trusts the User Pool as an authentication provider; maps authenticated users to an IAM role scoped for read access to a specific DynamoDB table
- **Groups**: an `Administrators` group used to gate an admin page in the front-end and (optionally) in the API layer

## Why this matters
This pattern — user pool for *authentication*, identity pool for *authorization* — is a standard, secure way to let a serverless front-end talk directly to AWS services (DynamoDB, S3, etc.) without a backend proxy for every call, while avoiding embedding permanent credentials in client code.

## Notes
This project was built as part of a guided AWS training lab. Only the configuration and integration code I modified/authored is included here; it is not a redistribution of the full training courseware.
