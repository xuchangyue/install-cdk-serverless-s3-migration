
1. 安装并配置AWS CLI2，可在笔记本或ec2中安装环境，这里新起一台amazon linux 2来安装，保证环境是干净的，Mac/Windows安装参考文档https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/install-cliv2.html

```
`curl ``"https://awscli.amazonaws.com/``awscli-exe-linux-x86_64-2.0.30.zip``"`` ``-``o ``"awscliv2.zip"`
`unzip awscliv2``.``zip`
`sudo ``./``aws``/``install`
`aws ``--``version
# 配置aws cli2 ，
# 选择国内区域来部署cdk stack，配置国内IAM admin user的Access Key ID 和 Secret Access Key
# Region选择需同步的S3 bucket region，比如cn-north-1`
`aws configure `
`AWS ``Access`` ``Key`` ID ``[``None``]:`` ``AKIATDxxxxxxxx`
`AWS ``Secret`` ``Access`` ``Key`` ``[``None``]:``  N6Q6wxxxxxxxxxxxxxxx`
`Default`` region name ``[``None``]:`` cn``-``north``-``1`
`Default`` output format ``[``None``]:`` json`
```

1.  安装CDK

```
# 安装cdk所需的python3和nodejs
sudo yum install python3 -y
curl --silent --location https://rpm.nodesource.com/setup_14.x | sudo bash -
sudo yum install -y nodejs
# 查看nodejs版本
`node ``-``e ``"console.log('Running Node.js ' + process.version)"`
`sudo npm install ``-``g aws``-``cdk
cdk --version`
```

1. 下载cdk-serverless

```
sudo yum install git -y
git clone https://github.com/aws-samples/amazon-s3-resumable-upload.git
cd ./amazon-s3-resumable-upload/serverless/cdk-serverless
`sudo python3 ``-``m pip install ``-``r requirements``.``txt`
# 查看已有的cdk stack
`cdk ls`
`s3``-``migrate``-``serverless`
`# 修改此stack的配置。`
`vi app``.``py
`
```

```
# 修改以下部分,src_bucket是国内的源bucket，des_bucket是global的目标bucket

`bucket_para ``=`` ``[{`
`    ``"src_bucket"``:`` ``"xucybjstosg"``,`
`    ``"src_prefix"``:`` ``""``,`
`    ``"des_bucket"``:`` ``"xucysing"``,`
`    ``"des_prefix"``:`` ``""`
`}]

Des_bucket_default = 'xucysing'
Des_prefix_default = ''
`
```

1. 在system manager中的Parameter Store中创建parameter，名称为s3_migration_credentials，如下图所示，Value为global IAM user的Access Key ID 和 Secret Access Key，这对AK/SK需要有权限写入S3.

```
`{`
`"aws_access_key_id": "AKIAxxxxxxxxxxxTV",`
`"aws_secret_access_key": "Qhogx0xxxxxxxxxxxxxAUZ",`
`"region": "ap-southeast-1"`
`}`
```

[Image: image.png]
1. 部署cdk stack

```
# 修改cn-aws-account-id为国内aws账户的id
cdk bootstrap aws://cn-aws-account-id/cn-north-1
cdk deploy # 输入y表示确认，等待部署完成
```

1. 测试，在国内S3 bucket 中上传文件，观察是否能够自动同步到global s3 bucket

