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

### Create Access Key for kops user
1. Create Access Key and Secret Access Key
```bash
aws iam create-access-key --user-name kops >kops-creds
cat kops-creds
```
2. Export **kops** Access Key
```bash
export AWS_ACCESS_KEY_ID=$(cat kops-creds | jq -r '.AccessKey.AccessKeyId')
```
3. Export **kops** Secret Access Key
```bash
export AWS_SECRET_ACCESS_KEY=$(cat kops-creds | jq -r '.AccessKey.SecretAccessKey')
```

### Choosing Availability Zones
1. Get the list of all AZ's in our previously defined "Default Region"
```bash
aws ec2 describe-availability-zones --region $AWS_DEFAULT_REGION
```
2. Export the list of all zones, as a list:  
##### If Windows, use `'\r'` instead `'\n'`
```bash
export ZONES=$(aws ec2 describe-availability-zones --region $AWS_DEFAULT_REGION | jq -r '.AvailabilityZones[].ZoneName' | tr '\n' ',' | tr -d ' ')
ZONES=${ZONES%?}
echo $ZONES
```






