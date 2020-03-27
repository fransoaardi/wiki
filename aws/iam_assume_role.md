# introduction

로컬환경에서 `aws sdk` 를 이용해서 session 을 생성하는데, 계정의 개인키를 발급받아서 `~/.aws/credentials`, `~/.aws/config` 에 이미 세팅을 해놓은 상태다.
deploy 할 EC2 instance 에 적용중인 IAM role 을 이용하는 방식으로 개발/배포 일관성을 맞추려고 했고, `sts:assumeRole` 을 이용하기로 했다.

# how-to
 - 로컬에서 identity 를 확인해보면 user 설정(개인계정)이다.
```
$ aws sts get-caller-identity
{
    "UserId": "AAA",
    "Account": "BBB",
    "Arn": "arn:aws:iam:BBB::user/CCC@gmail.com"
}
```

- config 에 Assume 할 IAM 정보를 추가했다.
```
$ cat config
[default]
region = ap-northeast-2
output = json

[profile platform]
role_arn = arn:aws:iam::BBB:role/DDD
source_profile = default
```

- 하지만 Assume 실패했다, user:CCC 에서 role:DDD 를 assume 할 수 없다고한다.
```
$ aws sts get-caller-identity --profile platform

An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:iam::BBB:user/CCC@gmail.com is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::BBB:role/DDD
```

- user:CCC 는 모든 Resource 에 대한 Allow 를 가진 super user 이기 때문에, 상관이 없긴 하지만, AssumeRole 하기 위해서는 아래와 같은 정책 설정이 필요하다.
> `iam:ListRoles` 는 필요없을것도 같지만, 제거해보진 않았다.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:ListRoles",
                "sts:AssumeRole"
            ],
            "Resource": "*"
        }
    ]
}
```

- **[중요] role:DDD 에게 policy 추가가 필요한것이 아니라, 다른 계정으로부터 AssumeRole 을 허용하려면, `신뢰 관계`를 추가해야된다.**
> BBB 계정에 속한 모든 사용자와 신뢰관계가 추가된다.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::BBB:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

- 이후, user:CCC 에서 role:DDD (platform 으로 profile alias 한)를 assumeRole 할 수 있다.
```
$ aws sts get-caller-identity --profile platform
{
    "UserId": "AAA:botocore-session-1585272320",
    "Account": "BBB",
    "Arn": "arn:aws:sts::BBB:assumed-role/DDD/botocore-session-1585272320"
}
```

- assume-role 하고 role-session 생성하고 이때 생성된 `AccessKeyId` 와 `SecretAccessKey` 값을 `aws configure` 해서 적용하면, temporary IAM role 이 적용될것이다. 
> 솔직히 귀찮아서 해보진 않았다. 다만 `role-session` 만 생성한다고 `aws configure` 된 secret, accessKey 값이 자동으로 바뀌진 않는다.
```
$ aws sts assume-role --role-arn arn:aws:iam::BBB:role/DDD --role-session-name test
{
    "Credentials": {
        "AccessKeyId": "AAA",
        "SecretAccessKey": "it's_secret",
        "SessionToken": "it's_secret_token",
        "Expiration": "2020-03-27T14:26:50Z"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "AAA:test",
        "Arn": "arn:aws:sts::BBB:assumed-role/DDD/test"
    }
}
```

- 간편하게 `--profile platform` 과 같이 일회성으로 사용하는게 더 편하다.
> `--profile platform` 있을때와 없을때 identity 값이 바뀌는것을 확인할 수 있다.
```
$ aws sts get-caller-identity
{
    "UserId": "AAA",
    "Account": "BBB",
    "Arn": "arn:aws:iam::BBB:user/CCC@gmail.com"
}
```

```
$ aws sts get-caller-identity --profile platform
{
    "UserId": "AAA:botocore-session-1585272320",
    "Account": "BBB",
    "Arn": "arn:aws:sts::BBB:assumed-role/DDD/botocore-session-1585272320"
}
```

- `aws configure list` 해보면 assumeRole 이 잘 실행됐음을 확인할 수 있다.
```
$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************HQEK shared-credentials-file
secret_key     ****************Qaom shared-credentials-file
    region           ap-northeast-2      config-file    ~/.aws/config
```

```
$ aws configure list --profile platform
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                 platform           manual    --profile
access_key     ****************257J      assume-role
secret_key     ****************buoD      assume-role
    region                <not set>             None    None
```
