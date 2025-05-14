<div style="display: flex; align-items: center;">
  <img src="logo.png" alt="Logo" width="50" height="50" style="margin-right: 10px;"/>
  <span style="font-size: 2.2em;">Invalidate AWS CloudFront Distribution</span>
</div>

<p>

This GitHub Action invalidates one or more paths in an AWS CloudFront distribution. It can be used to refresh cached content when you update files in an associated S3 bucket or other origin.

</p>

---

## 📚 Table of Contents

- [📥 Inputs](#inputs)
- [📤 Outputs](#outputs)
- [📦 Usage](#usage)
- [🚦 Requirements](#requirements)

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="inputs"></a>
## 📥 Inputs

| Name               | Description                                   | Required | Default |
| ------------------ | --------------------------------------------- | -------- | ------- |
| `distribution-id`  | The CloudFront Distribution ID                | true     | —       |
| `paths`            | Paths to invalidate (e.g., `/*`, `/index.js`) | false    | `/*`    |

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="outputs"></a>
## 📤 Outputs

| Name               | Description                         |
| ------------------ | ----------------------------------- |
| `invalidation-id`  | The ID of the CloudFront invalidation |

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="usage"></a>
## 📦 Usage

### Example – Invalidate all objects in a CloudFront distribution

```yaml
jobs:
  invalidate:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ vars.AWS_ACCOUNT_ID }}:role/cloudfront-distribution-role
          aws-region: ${{ vars.AWS_REGION }}

      - name: Invalidate CloudFront Distribution
        uses: devopspolis/invalidate-aws-cloudfront-distribution@main
        with:
          distribution-id: E123EXAMPLE456
          paths: '/*'
````

<!-- trunk-ignore(markdownlint/MD033) -->
<a id="requirements"></a>
## 🚦Requirements

1. The calling workflow must have the permissions shown below.
1. The calling workflow must have permission to assume a role with cloudfront:CreateInvalidation permissions.

    In the example below the `AWS_ACCOUNT_ID` and `AWS_REGION` are retrieved from the GitHub repository environment variables, enabling the workflow to target environment specific AWS accounts.
    OIDC authentication is recommended for GitHub → AWS access, using aws-actions/configure-aws-credentials.

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Set up AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ vars.AWS_ACCOUNT_ID }}:role/invalidate-cloudfront-distribution-role
          aws-region: ${{ vars.AWS_REGION }}
```
