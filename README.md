## Prerequisites:
* [git-scm](https://git-scm.com/downloads)
* [aws account](https://aws.amazon.com/)
* [aws cli](https://aws.amazon.com/cli/)
* [jq](https://stedolan.github.io/jq/download/)
* [kops](https://github.com/kubernetes/kops#installing)

Original article: https://gist.github.com/vfarcic/3c9ddff3fd412e42175a2eceab049421

## Steps:
### Create a new IAM user account and export ENV for AWS CLI
1. Create a new user account **kops** as a **programatic user account**.
2. Export **Access Key**, **Secret Access Key** and **Default Region**  
``` bash
open "https://console.aws.amazon.com/iam/home#/security_credential"
export AWS_ACCESS_KEY_ID=[...]
export AWS_SECRET_ACCESS_KEY=[...]
export AWS_DEFAULT_REGION=us-east-2
```




