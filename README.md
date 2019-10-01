## Prerequisites:
* [git-scm](https://git-scm.com/downloads)
* [aws account](https://aws.amazon.com/)
* [aws cli](https://aws.amazon.com/cli/)
* [jq](https://stedolan.github.io/jq/download/)
* [kops](https://github.com/kubernetes/kops#installing)

Original article: https://gist.github.com/vfarcic/3c9ddff3fd412e42175a2eceab049421

## Steps:
### Export ENV for AWS CLI (of the account that have an admin access)
1. Export **Access Key**, **Secret Access Key** and **Default Region**  
```bash
open "https://console.aws.amazon.com/iam/home#/security_credential"
export AWS_ACCESS_KEY_ID=[...]
export AWS_SECRET_ACCESS_KEY=[...]
export AWS_DEFAULT_REGION=us-east-1
```

### Create a new user account for Kubernetes cluster with an appropriate permissions
1. Create Group
```bash
aws iam create-group --group-name kops
aws iam attach-group-policy --group-name kops --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
aws iam attach-group-policy --group-name kops --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam attach-group-policy --group-name kops --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess
aws iam attach-group-policy --group-name kops --policy-arn arn:aws:iam::aws:policy/IAMFullAccess
```
2. Create User
```bash
aws iam create-user --user-name kops
```
3. Add User to Group
```bash
aws iam add-user-to-group --user-name kops --group-name kops
```
4. Create Key Pair to be used to connect to the K8s instances
```bash
aws iam create-access-key --user-name kops >kops-creds
cat kops-creds
```




