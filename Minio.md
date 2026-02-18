Run minio docker
```
docker network create minio-net
docker run -d \
      --name minio-server \
      -p 9000:9000 \
      -p 9001:9001 \
      -e MINIO_ROOT_USER=minioadmin \
      -e MINIO_ROOT_PASSWORD=minioadmin \
      minio/minio server /data --console-address ":9001"
```

Open http://localhost:9001/

```
// create bucket
mc mb myminio/user1bucket

// create user
mc admin user add myminio user test1234

//list policy
mc admin user list myminio

//create policy
mc admin policy create myminio user-policy user1-policy2.json

//attach policy to user
mc admin policy attach myminio user-policy --user=user

//detach policy from user
mc admin policy detach myminio user-policy2 --user=user
```

 user-policy.json
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:PutBucketPolicy",
        "s3:GetBucketPolicy",
        "s3:DeleteBucketPolicy",
        "s3:ListAllMyBuckets",
        "s3:ListBucket"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::user1bucket"
      ],
      "Sid": ""
    },
    {
      "Action": [
        "s3:AbortMultipartUpload",
        "s3:DeleteObject",
        "s3:GetObject",
        "s3:ListMultipartUploadParts",
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::user1bucket/*"
      ],
      "Sid": ""
    }
  ]
}
```

Stand http://sorm-test-input01.ural.mts.ru:8002/policies/readonly