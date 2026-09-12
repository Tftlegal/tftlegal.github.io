---
title: "aws secret manager cross account copy"
date: 2025-09-06T08:47:50Z
source: "sidmidru"
original_url: "https://sidmid.ru/aws-secret-manager-cross-account-copy/"
summary: "Задача — копировать секрет из AWS Secrets Manager одного аккаунта в другой.   Источником выступает аккаунт 1857432660 с секретом test-secret, а приёмником — аккаунт 4507032160.   При изменении секрета в источнике событие фиксируется CloudTrail.   EventBridge получает это событие и запускает Lambda-функцию в аккаунте-приёмнике.   Lambda обращается к источнику и забирает секрет, используя KMS-ключ."
---

# aws secret manager cross account copy

## Краткое содержание

Задача — копировать секрет из AWS Secrets Manager одного аккаунта в другой.  
Источником выступает аккаунт 1857432660 с секретом test-secret, а приёмником — аккаунт 4507032160.  
При изменении секрета в источнике событие фиксируется CloudTrail.  
EventBridge получает это событие и запускает Lambda-функцию в аккаунте-приёмнике.  
Lambda обращается к источнику и забирает секрет, используя KMS-ключ.

## Полная статья

Thank you for reading this post, don't forget to subscribe!

есть задача из одного aws account копировать secret созданный в aws secret manager в другой aws account

**1857432660**(источник, секрет`test-secret`)
**4507032160**(приёмник)

![](/news/sidmidru/article-636850d812ccd1b8/image-01.png)

общий смысл такой:
в аккаунте источнике вы меняете секрет, данное событие ловит cloudtrail, событие в cloudtrail видит eventbridge и тригерит в аккаунте (приёмнике) lamda, которая идёт в секрет в аккаунте источнике забирает используя kms ключ дешифрует секрет и далее записывает все изменения в свой секрет в аккаунте приёмнике.

- 

### **создаём kms ключ: (account 1857432660)**

KMS- > Customer managed keys → create key

Key type = Symmetric
Key usage =Encrypt and decrypt

Advanced options =KMS-*recommended и Multi-Region key*

*Alias*= alias/sm-test-secret

на шаге

Define key usage permissions - optional для OtherAWSaccounts добавляем аккаунты которым можно использовать этот ключ. в моём случае это 4507032160
без этого ключа секрет нельзя будет передать в другие аккаунты, так как секрет будет зашифрован дефолтным ключом.

![](/news/sidmidru/article-636850d812ccd1b8/image-02.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-03.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-04.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-05.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-06.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-07.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-08.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-09.png)

### **2. создаём cloudtrail (account 1857432660)**

CloudTrail → Trails → create trail если ничего не создано, то создаём по умолчанию, он и бакет сам создаст и имя дефолтное задаст management-events. Этот trail нужен чтобы события об изменениях мог отследить eventbridge

![](/news/sidmidru/article-636850d812ccd1b8/image-10.png)

### **3. создаём секрет в aws secret manager и вешаем на него созданный нами kms ключ (account 1857432660)**

AWSSecrets Manager → Secrets → store a new secret
создаём секрет test-secret
так же на этот секрет вешаем policy Resource permissions - optional

|  | { "Version" : "2012-10-17", "Statement" : [ { "Sid" : "AllowBRoleRead", "Effect" : "Allow", "Principal" : { "AWS" : "arn:aws:iam::4507032160:role/sm-mirror-exec" }, "Action" : [ "secretsmanager:GetSecretValue", "secretsmanager:DescribeSecret" ], "Resource" : "arn:aws:secretsmanager:us-east-1:1857432660:secret:test-secret-FTI7tY" } ]} |
| --- | --- |

в нём мы указываем Principal arn role (sm-mirror-exec) которую мы потом создадим в аккаунте 4507032160
и задаём какие действия можно выполнять с нашим секретом. в Resource указываем arn нашего секрета к которому мы выдаём доступ.

![](/news/sidmidru/article-636850d812ccd1b8/image-11.png)

### **4. создаём iam role для lambda. (account 4507032160)**

IAM→ Roles → create role →Custom trust policy

|  | { "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Principal": { "Service": "lambda.amazonaws.com" }, "Action": "sts:AssumeRole" } ]} |
| --- | --- |

вот это inline policy:

| 1234567891011121314151617181920212223242526272829303132333435 | { "Version": "2012-10-17", "Statement": [ { "Sid": "ReadFromA", "Effect": "Allow", "Action": [ "secretsmanager:GetSecretValue", "secretsmanager:DescribeSecret" ], "Resource": "arn:aws:secretsmanager:us-east-1:1857432660:secret:test-secret-*" }, { "Sid": "DecryptA", "Effect": "Allow", "Action": [ "kms:Decrypt", "kms:DescribeKey" ], "Resource": "arn:aws:kms:us-east-1:1857432660:key/*" }, { "Sid": "WriteToB", "Effect": "Allow", "Action": [ "secretsmanager:CreateSecret", "secretsmanager:PutSecretValue", "secretsmanager:UpdateSecret", "secretsmanager:DescribeSecret", "secretsmanager:TagResource" ], "Resource": "arn:aws:secretsmanager:us-east-1:4507032160:secret:test-secret-*" } ]} |
| --- | --- |

в этой policy мы разрешаем ходить в секрет в аккаунте 1857432660 и делать kms Decrypt а также ходить в секрет в текущем аккаунте 4507032160 и обновлять секрет

роль сохраняем с именем sm-mirror-exec,

arn это роли arn:aws:iam::4507032160:role/sm-mirror-exec он как раз был нужен на 3ем шаге этой инструкции.

![](/news/sidmidru/article-636850d812ccd1b8/image-12.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-13.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-14.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-15.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-16.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-17.png)

### **5. создаём lambda (account 4507032160)**

Lambda → Functions → create function

Author from scratch
Function name = sm-mirror-test-secret
Runtime = Python 3.12
Architecture = x86_64
Change default execution role →Use an existing role = sm-mirror-exec

create function

далее переходим в эту lambda и в Code добавляем:

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748 | import jsondef lambda_handler(event, context): # TODO implement return { 'statusCode': 200, 'body': json.dumps('Hello from Lambda!') }import os, boto3, botocoreSRC_ARN = os.environ["SOURCE_SECRET_ARN"]SRC_REGION = os.environ["SOURCE_REGION"]DST_NAME = os.environ["TARGET_SECRET_NAME"]DST_REGION = os.environ.get("TARGET_REGION", SRC_ARN.split(":")[3])DST_KMS = os.environ.get("TARGET_KMS_KEY_ID")sm_src = boto3.client("secretsmanager", region_name=SRC_REGION)sm_dst = boto3.client("secretsmanager", region_name=DST_REGION)def lambda_handler(event, context): v = sm_src.get_secret_value(SecretId=SRC_ARN) s = v.get("SecretString"); b = v.get("SecretBinary") try: sm_dst.describe_secret(SecretId=DST_NAME) exists = True except botocore.exceptions.ClientError as e: if e.response["Error"]["Code"] == "ResourceNotFoundException": exists = False else: raise if not exists: args = {"Name": DST_NAME} if s is not None: args["SecretString"] = s if b is not None: args["SecretBinary"] = b if DST_KMS: args["KmsKeyId"] = DST_KMS args["Tags"] = [{"Key": "SyncedFrom", "Value": SRC_ARN}] sm_dst.create_secret(**args) else: args = {"SecretId": DST_NAME} if s is not None: args["SecretString"] = s if b is not None: args["SecretBinary"] = b sm_dst.put_secret_value(**args) return {"status": "ok"} |
| --- | --- |

сохраняем и переходим
Configuration → Environment variables
и добавляем следующие переменные:

| SOURCE_REGION | us-east-1 |
| --- | --- |
| SOURCE_SECRET_ARN | arn:aws:secretsmanager:us-east-1:1857432660:secret:test-secret-FTI7tY |
| TARGET_REGION | us-east-1 |
| TARGET_SECRET_NAME | test-secret |

далее переходим в
Configuration → Permissions → Resource-based policy statements → Add permissions

там заполняем:

StatementID= AllowEventBridgeInvoke
Principal = 1857432660
Action = lambda:InvokeFunction

Save

![](/news/sidmidru/article-636850d812ccd1b8/image-18.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-19.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-20.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-21.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-22.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-23.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-24.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-25.png)

### **6. создаёмIAMrole для eventbridge (account 1857432660)**

чтобы разрешить EventBridge вызывать Lambda в target account 4507032160 нужно создать execution-роль для EventBridge

IAM→ Roles → create role → Custom trust policy

|  | { "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Principal": { "Service": "events.amazonaws.com" }, "Action": "sts:AssumeRole" } ]} |
| --- | --- |

Role name = evb-to-lambda-b

далее заходим в рольevb-to-lambda-b→ Permissions → Add permisstions → Create inline policy

|  | { "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": "lambda:InvokeFunction", "Resource": "arn:aws:lambda:us-east-1:4507032160:function:sm-mirror-test-secret" } ]} |
| --- | --- |

Policy name = evb-to-lambda-b

![](/news/sidmidru/article-636850d812ccd1b8/image-26.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-27.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-28.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-29.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-30.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-31.png)

### **7. создаём eventbridge (account 1857432660)**

Event buses можем использовать дефолтный, создадим rule

Amazon EventBridge → Rules → Create rule
Name = SecretChangeInvokeLambda
Rule type = Rule with an event pattern
Event source = Other
Event pattern = Custom pattern (JSONeditor)

|  | { "source": ["aws.secretsmanager"], "detail-type": ["AWS API Call via CloudTrail"], "detail": { "eventSource": ["secretsmanager.amazonaws.com"], "eventName": ["CreateSecret", "PutSecretValue", "UpdateSecret", "DeleteSecret"], "requestParameters": { "secretId": [{ "prefix": "arn:aws:secretsmanager:us-east-1:1857432660:secret:test-secret" }] } }} |
| --- | --- |

Target types =AWSservice
Select a target = Lambda function
Target location = Target in anotherAWSaccount
Function = arn:aws:lambda:us-east-1:4507032160:function:sm-mirror-test-secret
Execution role =evb-to-lambda-b
create

![](/news/sidmidru/article-636850d812ccd1b8/image-32.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-33.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-34.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-35.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-36.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-37.png)

### проверяем:

добавим секрет в**1857432660**он обновится в**4507032160**

![](/news/sidmidru/article-636850d812ccd1b8/image-38.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-39.png)

![](/news/sidmidru/article-636850d812ccd1b8/image-40.png)

смотрим как отработала lambda в мониторинге всё ок

![](/news/sidmidru/article-636850d812ccd1b8/image-41.png)

## Оригинал

https://sidmid.ru/aws-secret-manager-cross-account-copy/
