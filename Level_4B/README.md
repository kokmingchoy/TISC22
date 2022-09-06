# Level 4B

![image](https://user-images.githubusercontent.com/82754379/188318980-32cbdff8-6a92-4a03-8d34-ff9fd8a4d0d7.png)

Heading over to the given URL at https://d20whnyjsgpc34.cloudfront.net/ I saw a page with (six) cat pictures:

![image](https://user-images.githubusercontent.com/82754379/188319697-22cd6b07-2594-4812-a149-74ed2375fe5d.png)


The source code for the web page revealed what could be clues:
```html
    <!-- Passcode -->
    <h1 class="mb-3">Cats rule the world</h1>
    <!-- Passcode -->
    <!-- 
      ----- Completed -----
      * Configure CloudFront to use the bucket - palindromecloudynekos as the origin
      
      ----- TODO -----
      * Configure custom header referrer and enforce S3 bucket to only accept that particular header
      * Secure all object access
    -->
```


## S3 Bucket

The clue indicated that there was an S3 bucket named "palindromecloudynekos", so I tried to access it within the browser at the URL "palindromecloudynekos.s3.amazonaws.com" and got an "Access Denied" result:

![Screenshot from 2022-09-05 00-23-26](https://user-images.githubusercontent.com/82754379/188323431-f0a06d2a-65de-45fc-9232-a8447d61cd75.png)

Reading through the write-ups for other S3-related Capture-The-Flag (CTF) challenges, I found that it may be possible to access the S3 bucket as long as I use an authenticated AWS account.

So, I signed up for an AWS account in the free tier and installed the AWS Command Line Interface (CLI), following the instructions in the online AWS documentation: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-prereqs.html .

After creating a new "Administrator" user and its access keys, I configured the AWS CLI:
```
$ aws configure

AWS Access Key ID [None]: AKIARIXBOXQJ742N6JWC
AWS Secret Access Key [None]:  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
Default region name [None]: ap-southeast-1
Default output format [None]: json
```

Next I used the authenticated account to list the contents of the given S3 bucket:
```
$ aws s3 ls s3://palindromecloudynekos

                           PRE api/
                           PRE img/
2022-08-23 21:16:20         34 error.html
2022-08-23 21:16:20       2257 index.html
```

To make things easier, I mirrored the contents of the S3 bucket to a local directory on my machine:
```
$ aws s3 sync s3://palindromecloudynekos .

download: s3://palindromecloudynekos/api/notes.txt to api/notes.txt
download: s3://palindromecloudynekos/error.html to ./error.html 
download: s3://palindromecloudynekos/index.html to ./index.html 
download: s3://palindromecloudynekos/img/photo3.jpg to img/photo3.jpg
download: s3://palindromecloudynekos/img/photo2.jpg to img/photo2.jpg
download: s3://palindromecloudynekos/img/photo5.jpg to img/photo5.jpg
download: s3://palindromecloudynekos/img/photo4.jpg to img/photo4.jpg
download: s3://palindromecloudynekos/img/photo6.jpg to img/photo6.jpg
download: s3://palindromecloudynekos/img/photo1.jpg to img/photo1.jpg
```

The file **api/notes.txt** gave some clues:
```
$ cat api/notes.txt

# Neko Access System Invocation Notes

Invoke with the passcode in the header "x-cat-header". The passcode is found on the cloudfront site, all lower caps and separated using underscore.

https://b40yqpyjb3.execute-api.ap-southeast-1.amazonaws.com/prod/agent

All EC2 computing instances should be tagged with the key: 'agent' and the value set to your username. Otherwise, the antivirus cleaner will wipe out the resources.
```

The given URL must be accessed with an HTTP GET request with the correct header in order to obtain the next clue:
```
$ curl -H "x-cat-header: cats_rule_the_world" https://b40yqpyjb3.execute-api.ap-southeast-1.amazonaws.com/prod/agent

{"Message": "Welcome there agent! Use the credentials wisely! It should be live for the next 120 minutes! Our antivirus will wipe them out and the associated resources after the expected time usage.", "Access_Key": "AKIAQYDFBGMS4XGUWZVT", "Secret_Key": "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"}
```

Each new GET request to the same URL seemed to obtain a different pair of values for **Access_Key** and **Secret_Key**.

I stored the new set of credentials under a different profile and prepared to work with it:
```
$ aws configure --profile tisc

$ aws configure --profile tisc
AWS Access Key ID [None]: AKIAQYDFBGMS4XGUWZVT
AWS Secret Access Key [None]: 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
Default region name [None]: ap-southeast-1
Default output format [None]: json

$ export AWS_PROFILE=tisc
```

I installed [enumerate-iam.py](https://github.com/andresriancho/enumerate-iam) from Github and used it to identify the permissions that the given credentials possess:
```
$ ./enumerate-py.iam --access-key AKIAQYDFBGMS4XGUWZVT --secret-key 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
```

One of the things I could do was to list the IAM roles:
```
$ aws iam list-roles
```

A few roles stood out:

- ec2_agent_role
- lambda_agent_development_role
- lambda_agent_webservice_role
- lambda_cleaner_service_role

The existence of these roles suggested that I may be required to spin up an EC2 instance (and tag it with the key 'agent' with value set to my username), and also write some AWS lambda functions.



