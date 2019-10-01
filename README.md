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

### Generating SSH keys
1. Create a new folder for the new cluster:
```bash
mkdir -p cluster
cd cluster
```
2. Create a new Key Pair
```bash
aws ec2 create-key-pair --key-name kops | jq -r '.KeyMaterial' >kops.pem
chmod 400 kops.pem
ssh-keygen -y -f kops.pem >kops.pub
```

### Create a Store for our Kops cluster
1. Export the name for our K8s cluster:
```bash
export NAME=kops.k8s.local
```
2. Export the S3 bucket name and then create the bucket in a default region we have defined in a one of the previous steps:
```bash
export BUCKET_NAME=kops-$(date +%s)
aws s3api create-bucket --bucket $BUCKET_NAME --create-bucket-configuration LocationConstraint=$AWS_DEFAULT_REGION
```
3. Export the "KOPS_STATE_STORE" env. variable:
```bash
export KOPS_STATE_STORE=s3://$BUCKET_NAME
```
### Create a Cluster
1. Create a Kops cluster
```bash
kops create cluster --name $NAME --master-count 3 --node-count 1 \
    --node-size t2.small --master-size t2.small --zones $ZONES \
    --master-zones $ZONES --ssh-public-key kops.pub \
    --networking kubenet --kubernetes-version v1.8.4 --yes
```

### Get the new Cluster details:
1. Get the cluster details:
```bash
kops get cluster
kubectl cluster-info
kops validate cluster
```







