## How to Deploy

### Check Your AWS Account
```sh
aws sts get-caller-identity
```

### Run Deploy Script
```sh
cd projects/environment-automation
chmod u+x ./bin/deploy
./bin/deploy
```

## Error Occur Before Deployment
I have already tried to deploy which results in failed. When deploying again it gives me this error.
```sh
An error occurred (ValidationError) when calling the CreateChangeSet operation:
Stack:arn:aws:cloudformation:ap-southeast-2:...:stack/NetworkVPC/... is in ROLLBACK_COMPLETE state and can not be updated.
```
means that the CloudFormation stack named NetworkVPC is in a failed state (ROLLBACK_COMPLETE), and cannot be updated or reused.

### Fix

Failed stack must be deleted before deploying again.
```sh
aws cloudformation delete-stack --stack-name NetworkVPC
```

> You only have to chmod the file once to make it executable

