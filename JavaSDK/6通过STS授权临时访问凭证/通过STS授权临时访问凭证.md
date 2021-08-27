通常情况下，对象存储桶和对象都是私有的。如果希望第三方用户可以上传对象到存储桶或从桶中下载对象，可以通过安全令牌服务 (Security Token Service, STS) 授权临时的AccessKey、SecretKey和Token，以实现对对象存储的操作。

STS为紫光云提供的临时访问权限管理服务。其优势如下：
- 您无需将长期密钥（AccessKey和SecretKey）透漏给第三方应用，只需生成一个临时的访问令牌并将令牌交给第三方用户。
- 您可以自定义这个令牌的访问权限及有效期限，访问令牌过期后会自动失效。


# 授权流程

<img style="width:496px;height:334px;margin:0;" src="https://oss-cn-north-1.unicloudsrv.com/document/documents/1271299099928297472.png" />

> **注意：**
> 
> - 正式AK/SK只能用于业务层使用，请勿将其直接交给第三方用户，导致信息泄露。
> 
> - 正式AK/SK用户主体为业务服务器，第三方终端用户对应的信息应由业务服务器自己维护。

# 步骤1：业务服务器后端向对象存储获取临时AK/SK/Token (JAVA)
## 一、安装SDK准备工作

示例项目使用Gradle管理，执行`gradle build`即可编译。

## 二、使用样例

1. 设置AccessKey、SecretKey、Endpoint和Region

   ```
   String accessKey = "******";
        String secretKey = "******";
        AWSStaticCredentialsProvider credential = new AWSStaticCredentialsProvider(
                new BasicAWSCredentials(accessKey, secretKey));
        EndpointConfiguration endpoint = new EndpointConfiguration("http://sts-
   beijing.unicloudsrv.com", "cn-beijing-1");
   ```
 ```

 > **说明：**
 > 
 > - AccessKey和SecretKey可以从“<a href="https://www.unicloud.com/console/#/main/iam/accessKey">
紫光云控制台 > 用户中心 > 安全信息管理</a>”处获取。
 > - Region测试可直接使用"cn-beijing-1"，无需修改。
 > - Endpoint地址区分内外网，若由北京节点访问可使用内网域名“sts-beijing- 
internal.unicloudsrv.com”，否则请使用公网域名“sts-beijing.unicloudsrv.com”。


2. 创建STS Service Client

 ```
AWSSecurityTokenService stsClient = AWSSecurityTokenServiceClientBuilder.standard()
                    .withCredentials(credential)
                    .withEndpointConfiguration(endpoint)
                    .build();
 ```

3. 创建GetFederationTokenRequest
 ```
// withName: 填写您系统内的用户名，长度2-32字节
// withPolicy: 请按照样例完整填写
// withDurationSeconds: 临时Token有效时间，单位为秒，最短900秒，最长129600秒
            GetFederationTokenRequest request = new GetFederationTokenRequest().withName("Bob")
                    .withPolicy("{\"Version\":\"2012-10-17\",\"Statement\":[{\"Sid\":\"Sid1\",\"Effect\":\"Allow\",\"Action\":[\"s3:*\"],\"Resource\":[\"*\"]}]}")
                    .withDurationSeconds(3600);
            GetFederationTokenResult response = stsClient.getFederationToken(request);
```

4. 获得临时令牌，包含AccessKeyId、SecretAccessKey和SessionToken
```
Credentials tempCredential = response.getCredentials();
            System.out.println(tempCredential.getAccessKeyId());
            System.out.println(tempCredential.getSecretAccessKey());
            System.out.println(tempCredential.getSessionToken());
        }
        catch(AmazonServiceException e) {
            e.printStackTrace();
        }
        catch(SdkClientException e) {
            e.printStackTrace();
        }
    }
}
```

# 步骤2：业务服务器将获取的临时AK/SK/Token发送给第三方用户实现上传

分别介绍通过IOS系统、Andriod系统和Web页面进行上传和下载的示例。

##  IOS系统上传下载示例（提供Swift和Objective-C两种方式）

### 一、安装SDK准备工作
 安装CocoaPods ：http://cocoapods.org

 在项目目录执行 `pod install`

### 二、使用样例
1. 从APP后端获取可用的临时密钥: AccessKey、SecretKey和SessionToken

 （1） Swift方式：

```
let endpoint = AWSEndpoint.init(urlString: "https://oss-cn-north-2.unicloudsrv.com") // 紫光云北京节点
let credentialProvider = AWSBasicSessionCredentialsProvider.init(
                    accessKey: "******",
                    secretKey: "******",
                    sessionToken:  "Aet0i9YvGMPNmb0vOkAiMC3GU4V+NhkPxXyUToWk+TRE2967MpmM4HdyWygnEppIj/QfzSd8vpkFe3/TmFQukuoAxUg/CiGVynM7/aieAbEt1zbjW3A3mmfukR177VX1Ljz3EkW5smYcn9U29dOcCpwQqpwWXdC9cEYgs+/Qbv4AnsVje2TLJCPb2KguGRBEKyMCPhlJsrKyI/KeT/OS5aFSK05bXpUvdcg+")
 ```

  （2） Objective-C方式：

 ```
AWSEndpoint *endpoint = [[AWSEndpoint alloc] initWithURLString:@"https://oss-cn-north-2.unicloudsrv.com"]; // 紫光云北京节点
AWSBasicSessionCredentialsProvider *credentialProvider = [[AWSBasicSessionCredentialsProvider alloc]
                                                        initWithAccessKey:@"UP9TA5DPLG9SU20FD7OT"
                                                        secretKey:@"CjyUv6NJTGTBkT16iDRnnE0jv25ktpiV9NDE9gKD"
                                                    sessionToken:@"Aet0i9YvGMPNmb0vOkAiMC3GU4V+NhkPxXyUToWk+TRE2967MpmM4HdyWygnEppIj/QfzSd8vpkFe3/TmFQukuoAxUg/CiGVynM7/aieAbEt1zbjW3A3mmfukR177VX1Ljz3EkW5smYcn9U29dOcCpwQqpwWXdC9cEYgs+/Qbv4AnsVje2TLJCPb2KguGRBEKyMCPhlJsrKyI/KeT/OS5aFSK05bXpUvdcg+"
                                                        ];
```

2. 配置AWS Service Manager

 （1） Swift方式：
```
let configuration = AWSServiceConfiguration.init(
                                                region: AWSRegionType.CNNorth1,
                                                endpoint: endpoint,
                                                credentialsProvider: credentialProvider)
AWSServiceManager.default().defaultServiceConfiguration = configuration
```

  （2） Objective-C方式：

```
AWSServiceConfiguration *configuration = [[AWSServiceConfiguration alloc]
                                              initWithRegion:AWSRegionCNNorth1
                                              endpoint:endpoint
                                              credentialsProvider:credentialProvider];
[AWSServiceManager defaultServiceManager].defaultServiceConfiguration = configuration;
```

3. 获取Transfer Utility

 （1） Swift方式：
```
let transferUtility =  AWSS3TransferUtility.default()
```
  （2） Objective-C方式：

```
AWSS3TransferUtility *transferUtility = [AWSS3TransferUtility defaultS3TransferUtility];
```

4. 使用Transfer Utility进行上传下载

 （1） Swift方式：
```
transferUtility.uploadData(
            data,
            bucket: Bucket,
            key: S3UploadKeyName,
            contentType: "image/png",
            expression: expression,
            completionHandler: completionHandler)
```

  （2） Objective-C方式：

```
[[transferUtility downloadDataFromBucket:S3BucketName
                                     key:S3DownloadKeyName
                              expression:expression
                       completionHandler:completionHandler]
```

##  Andriod系统上传下载示例

### 一、安装SDK准备工作

示例项目使用Gradle管理，执行`gradle build`即可编译。

### 二、使用样例

1. 从APP后端获取可用的临时密钥: AccessKey、SecretKey和SessionToken
```
String accessKey = "******";
String secretKey = "******";
String sessionToken = "Aet0i9YvGMPNmb0vOkAiMC3GU4V+NhkPxXyUToWk+TRE2967MpmM4HdyWygnEppIj/QfzSd8vpkFe3/TmFQukuoAxUg/CiGVynM7/aieAbEt1zbjW3A3mmfukR177VX1Ljz3EkW5smYcn9U29dOcCpwQqpwWXdC9cEYgs+/Qbv4AnsVje2TLJCPb2KguGRBEKyMCPhlJsrKyI/KeT/OS5aFSK05bXpUvdcg+";
```

2. 初始化S3 Client
```
AWSCredentials credential = new BasicSessionCredentials(
       accessKey, secretKey, sessionToken);
sS3Client = new AmazonS3Client(credential);
sS3Client.setEndpoint("https://oss-cn-north-2.unicloudsrv.com");
sS3Client.setS3ClientOptions(S3ClientOptions.builder()
                    .setPathStyleAccess(true).build());
```

3. 初始化Transfer Utility
```
sTransferUtility = TransferUtility.builder()
                    .s3Client(sS3Client)
                    // 修改为后端指定Bucket
                    .defaultBucket("hzhz")
                    .build();
```

4. 使用Transfer Utility进行上传下载
```
TransferObserver observer = transferUtility.upload(
                file.getName(),
                file
);
```


## 通过Web页面上传下载示例
```
<script src="aws-sdk-2.664.0.min.js"></script>
<script type="text/javascript">
function openInput() {
    console.log("openInput")
    document.getElementById("file").click();
}
function upload() {
    console.log("upload")
    AWS.config.endpoint = "http://s3.test.com:9090"
    AWS.config.region = "cn-beijing-1"
    AWS.config.accessKeyId = "******"
    AWS.config.secretAccessKey = "*********"
    AWS.config.sessionToken = "AQ2hkbs76rpyvG014H3FyVOi+Kd2KbZ1MG/puM7/TUh10ycSexcsKJ3NTJ6MA7vw84qXSP52oQzWuejacuKjFkEdx5btFYuBYQeqZB/Rq/0r/bt93HipnFL+MgcMd8vNAtFyFyvaifVdj8KP1s8rsSV3W/PGD/76VIiIF5YnsdQdUbzURecvrzFaKSKFFriLF4aEj6q1TEbGv6R8Kpxp5OAxg0SkCAsfe6YR"
    AWS.config.s3ForcePathStyle = true
    s3 = new AWS.S3()

    let files = document.getElementById("file").files
    if (!files.length) {
        return alert("Please choose a file to upload first.")
    }
    let file = files[0]
    let params = {
        Bucket: 'test1',
        Key: file.name,
        Body: file
    }
    s3.putObject(params, function(err, data) { 
        if (err) {
            console.log(err, err.stack)
        } else {
            alert("upload "+ file.name + " success")
        }
    })
}
</script>
```

```