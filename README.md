<div style="display: flex; align-items: center;">
  <img src="logo.png" alt="Logo" width="50" height="50" style="margin-right: 10px;"/>
  <span style="font-size: 2.2em;">Invalidate AWS CloudFront Distribution</span>
</div>

![GitHub Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-Invalidate%20AWS%20CloudFront%20Distribution-blue?logo=github)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

<p>
This GitHub Action invalidates one or more paths in an AWS CloudFront distribution. It can be used to refresh cached content after updating files/objects in an associated S3 bucket or other origin.
</p>

See more [GitHub Actions by DevOpspolis](https://github.com/marketplace?query=devopspolis&type=actions)

---

## 📚 Table of Contents

- [✨ Features](#features)
- [📥 Inputs](#inputs)
- [📤 Outputs](#outputs)
- [📦 Usage](#usage)
- [🚦 Requirements](#requirements)
- [🔧 Troubleshooting](#troubleshooting)
- [🧑‍⚖️ Legal](#legal)

<!-- trunk-ignore(markdownlint/MD033) -->
<a id="features"></a>
## ✨ Features
- ✅ Simple to use with comprehensive validation
- ✅ Full or partial invalidation via path patterns
- ✅ Automatic jq installation if not available
- ✅ Smart AWS region detection
- ✅ Support for both full ARNs and short role names
- ✅ Optional wait for completion
- ✅ Comprehensive error handling and logging
- ✅ Input validation (distribution ID format, path format, path limits)
- ✅ Multiple output parameters for monitoring
- ✅ Designed to work with [devopspolis/deploy-to-aws-s3](devopspolis/deploy-to-aws-s3) and [devopspolis/deploy-artifact-to-aws-s3](https://github.com/marketplace/actions/deploy-artifact-to-aws-s3) actions

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="inputs"></a>
## 📥 Inputs

| Name                   | Description                                                                | Required | Default |
| ---------------------- | -------------------------------------------------------------------------- | -------- | ------- |
| `distribution-id`      | The CloudFront Distribution ID (14 alphanumeric characters)               | true     | —       |
| `paths`                | Paths to invalidate (space-separated, e.g., `"/* /index.html /css/*"`)   | false    | `/*`    |
| `role`                 | IAM role ARN or short name to assume for deployment                       | false    | —       |
| `wait-for-completion`  | Wait for invalidation to complete (max 30 minutes)                        | false    | `false` |
| `caller-reference`     | Custom caller reference for the invalidation                              | false    | —       |

### Path Format Guidelines
- All paths must start with `/`
- Use `/*` to invalidate all objects
- Use specific paths like `/index.html` for individual files
- Use wildcards like `/css/*` for directories
- Multiple paths should be space-separated: `"/index.html /css/* /js/*"`
- Maximum 1000 paths per invalidation

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="outputs"></a>
## 📤 Outputs

| Name                     | Description                              |
| ------------------------ | ---------------------------------------- |
| `invalidation-id`        | The ID of the CloudFront invalidation   |
| `invalidation-status`    | The status of the invalidation           |
| `invalidation-create-time` | The creation time of the invalidation  |

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="usage"></a>
## 📦 Usage

### Example 1 – Basic invalidation with AWS credentials pre-configured

```yaml
jobs:
  invalidate:
    runs-on: ubuntu-latest
    steps:
      - name: Invalidate CloudFront Distribution
        uses: devopspolis/invalidate-aws-cloudfront-distribution@v1
        with:
          distribution-id: E123EXAMPLE456
          paths: '/*'
```

### Example 2 – Invalidation with role assumption

```yaml
jobs:
  invalidate:
    runs-on: ubuntu-latest
    permissions:
      id-token: write

    steps:
      - name: Invalidate CloudFront Distribution
        uses: devopspolis/invalidate-aws-cloudfront-distribution@v1
        with:
          distribution-id: E123EXAMPLE456
          paths: '/*'
          role: cloudfront-invalidation-role  # Short name
        env:
          AWS_REGION: us-east-1
          AWS_ACCOUNT_ID: ${{ vars.AWS_ACCOUNT_ID }}
```

### Example 3 – Multiple specific paths with completion wait

```yaml
jobs:
  invalidate:
    runs-on: ubuntu-latest
    permissions:
      id-token: write

    steps:
      - name: Invalidate specific paths
        uses: devopspolis/invalidate-aws-cloudfront-distribution@v1
        with:
          distribution-id: E123EXAMPLE456
          paths: '/index.html /css/* /js/* /images/*'
          wait-for-completion: true
          role: arn:aws:iam::123456789012:role/CloudFrontInvalidationRole
        env:
          AWS_REGION: us-west-2
```

### Example 4 – Integration with S3 deployment

```yaml
jobs:
  deploy-and-invalidate:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy to S3
        uses: devopspolis/deploy-to-aws-s3@v1
        with:
          bucket-name: my-website-bucket
          source-directory: dist/
          role: deployment-role

      - name: Invalidate CloudFront
        uses: devopspolis/invalidate-aws-cloudfront-distribution@v1
        with:
          distribution-id: E123EXAMPLE456
          paths: '/*'
          role: deployment-role
        env:
          AWS_REGION: us-east-1
          AWS_ACCOUNT_ID: ${{ vars.AWS_ACCOUNT_ID }}
```

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="requirements"></a>
## 🚦 Requirements

### 1. GitHub Workflow Permissions
The calling workflow must have the following permissions when using role assumption:

```yaml
permissions:
  id-token: write  # Required for OIDC
```

### 2. AWS IAM Permissions
The assumed role or AWS credentials must have the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation",
        "cloudfront:GetInvalidation",
        "cloudfront:GetDistribution"
      ],
      "Resource": "*"
    }
  ]
}
```

### 3. Environment Variables
- `AWS_REGION` or `AWS_DEFAULT_REGION`: AWS region (auto-detected if not provided)
- `AWS_ACCOUNT_ID`: Required only when using short role names (auto-detected if credentials allow)

### 4. AWS Authentication
The action supports multiple authentication methods:
- **Pre-configured AWS credentials** (simplest)
- **Role assumption with OIDC** (recommended for security)
- **Role assumption with existing credentials**

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="troubleshooting"></a>
## 🔧 Troubleshooting

### Common Issues

#### ❌ Invalid distribution ID format
```
Error: Invalid distribution ID format. Expected 14 alphanumeric characters
```
**Solution**: Ensure your distribution ID is exactly 14 characters and contains only letters and numbers.

#### ❌ Cannot access CloudFront distribution
```
Error: Cannot access CloudFront distribution E123EXAMPLE456. Check permissions and distribution ID.
```
**Solutions**:
- Verify the distribution ID exists and is correct
- Check that your AWS credentials have `cloudfront:GetDistribution` permission
- Ensure the distribution is in the same AWS account as your credentials

#### ❌ Invalid path format
```
Error: Invalid path format: 'index.html'. All paths must start with '/'
```
**Solution**: Ensure all paths start with `/`. Use `/index.html` instead of `index.html`.

#### ❌ Too many paths specified
```
Error: Too many paths specified (1001). CloudFront supports maximum 1000 paths per invalidation.
```
**Solution**: Reduce the number of paths or use wildcard patterns like `/*` to invalidate all objects.

#### ❌ AWS credentials not configured
```
Error: AWS credentials not configured or invalid
```
**Solutions**:
- Ensure AWS credentials are configured before this action
- If using role assumption, verify the role ARN is correct
- Check that `AWS_REGION` is set when using role assumption

### Debug Tips

1. **Enable detailed logging** by setting `ACTIONS_STEP_DEBUG=true` in your repository secrets
2. **Check AWS CloudTrail** for API calls and permission issues
3. **Verify distribution status** in the AWS CloudFront console
4. **Test AWS CLI access** in your local environment with the same credentials

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="legal"></a>
## 🧑‍⚖️ Legal
The MIT License (MIT)