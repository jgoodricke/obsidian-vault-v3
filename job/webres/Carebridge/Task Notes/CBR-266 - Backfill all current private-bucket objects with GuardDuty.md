## Original Staging S3 Policy
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CBRGuardDutyStagingReserveResultTag",
            "Effect": "Deny",
            "Principal": "*",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectTagging",
                "s3:PutObjectVersionTagging"
            ],
            "Resource": "arn:aws:s3:::qip-staging-private/*",
            "Condition": {
                "ArnNotEquals": {
                    "aws:PrincipalArn": "arn:aws:iam::897787419894:role/service-role/GuardDutyS3MalwareScanRole-cb36ac6a-7d44-41e6-b8e6-f2fff4bfdde4"
                },
                "ForAnyValue:StringEquals": {
                    "s3:RequestObjectTagKeys": "GuardDutyMalwareScanStatus"
                }
            }
        },
        {
            "Sid": "CBRGuardDutyStagingPreventTagDeletion",
            "Effect": "Deny",
            "Principal": "*",
            "Action": [
                "s3:DeleteObjectTagging",
                "s3:DeleteObjectVersionTagging"
            ],
            "Resource": "arn:aws:s3:::qip-staging-private/*",
            "Condition": {
                "ArnNotEquals": {
                    "aws:PrincipalArn": "arn:aws:iam::897787419894:role/service-role/GuardDutyS3MalwareScanRole-cb36ac6a-7d44-41e6-b8e6-f2fff4bfdde4"
                }
            }
        },
        {
            "Sid": "CBRGuardDutyStagingPreventCopiedVerdicts",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::qip-staging-private/*",
            "Condition": {
                "Null": {
                    "s3:x-amz-copy-source": "false"
                }
            }
        },
        {
            "Sid": "CBRGuardDutyStagingPreventReplicatedVerdicts",
            "Effect": "Deny",
            "Principal": "*",
            "Action": [
                "s3:ReplicateObject",
                "s3:ReplicateTags"
            ],
            "Resource": "arn:aws:s3:::qip-staging-private/*",
            "Condition": {
                "ArnNotEquals": {
                    "aws:PrincipalArn": "arn:aws:iam::897787419894:role/service-role/GuardDutyS3MalwareScanRole-cb36ac6a-7d44-41e6-b8e6-f2fff4bfdde4"
                }
            }
        },
        {
            "Sid": "CBRGuardDutyStagingRequireCurrentCleanTag",
            "Effect": "Deny",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:GetObjectVersion"
            ],
            "Resource": "arn:aws:s3:::qip-staging-private/*",
            "Condition": {
                "StringNotEquals": {
                    "s3:ExistingObjectTag/GuardDutyMalwareScanStatus": "NO_THREATS_FOUND"
                },
                "ArnNotEquals": {
                    "aws:PrincipalArn": "arn:aws:iam::897787419894:role/service-role/GuardDutyS3MalwareScanRole-cb36ac6a-7d44-41e6-b8e6-f2fff4bfdde4"
                }
            }
        }
    ]
}
```
