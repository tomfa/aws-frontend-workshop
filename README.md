# AWS frontend with S3 and CloudFront
A small workshop on how to deploy a webpacked app on AWS infrstructure.

Presentation at [tomfa.github.io/aws-frontend-workshop/presentation](https://tomfa.github.io/aws-frontend-workshop/presentation/#/)

## Prerequisites
1. **Register for an [AWS account](https://aws.amazon.com/free/)**, and login to [the console](https://console.aws.amazon.com/console/home)
2. **Create credentials**
Click your name in the top right corner -> *My Security Credentials*, and create Access Keys.
3. **Export the keys to your environment**
```
export AWS_ACCESS_KEY_ID=AKIAIJFSBCEXEXAMPLE
export AWS_SECRET_ACCESS_KEY=oe+VyD00LDdNpWZnl02U/XYSSMZEHkJYEXAMPLE
```
4. **Make sure you have [aws cli](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) installed**.
If you do, the command ```aws``` should respond with something sensible.

