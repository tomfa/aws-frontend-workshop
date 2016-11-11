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

## Host files using AWS S3
S3 stands for Simple Storage Service. It can be used to store logs, backups, or files of any kind. One of its main uses is storing static web files, and that's just what a frontend app is. So let's do it!

1. **In the [AWS Console](https://console.aws.amazon.com/console/home), go to [S3](https://console.aws.amazon.com/s3/home), and create a bucket.**
You can call it what ever you wish, but it must (actually) be globally unique. (We'll get to why later.) I'll call mine *fishsticksasbait* throughout the workshop.

2. **Clone the [webapp-repo](https://github.com/tomfa/redux-example)**
```
git clone git@github.com:tomfa/redux-example.git
```
We got a few branches here, for different sort of apps.

3. **Checkout the branch simple**
```
git checkout simple
```
This is a simple static webapp, without any libraries or other fancy stuff. Let's start with uploading this to S3.

4. **Upload files to S3**
Use [aws s3 sync](http://docs.aws.amazon.com/cli/latest/reference/s3/sync.html) to syncronize your folder with your bucket. If you did not create your S3 bucket in Frankfurt (eu-central-1), you can lookup your region code in [AWS Regions and Endpoints](http://docs.aws.amazon.com/general/latest/gr/rande.html#apigateway_region).
```
aws s3 sync . s3://fishsticksasbait --region=eu-central-1
```
Voilà! You should immediately be able to see the glories website at [https://fishsticksonarod.s3-eu-central-1.amazonaws.com/index.html]().
*(Nope, I lied, sorry)*

5. **Make your bucket publicly accessable**
By default, your bucket is not publicly accessable. In our case, we want everyone to be able to read from the bucket, but not write. We can do this by adding the following policy to the bucket:
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "PublicReadForGetBucketObjects",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::fishsticksasbait/*"
		}
	]
}
```
You can either do this through AWS Console > FishSticksAsBait > Properties > Permissions > Edit bucket policy, or through the [aws cli](http://docs.aws.amazon.com/cli/latest/reference/s3api/put-bucket-policy.html).

This allow everyone to get any files in your bucket. Now, [https://fishsticksonarod.s3-eu-central-1.amazonaws.com/index.html]() should work (pinky-swear).

**PS!** Notice how index.html uses style.css in the **folder** static. How will this work out when we have a React router? (secret: We're getting to that soon)

