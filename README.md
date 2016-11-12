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
By default, your bucket is not publicly accessable. In our case, we want everyone to be able to read from the bucket, but not write. We can do this under *AWS Console > FishSticksAsBait > Properties > Permissions*. Allow *Everyone* to *List*, and add the following policy to the bucket:
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
It's also possible to do this through the [aws cli](http://docs.aws.amazon.com/cli/latest/reference/s3api/put-bucket-policy.html).

This allow everyone to get any files in your bucket. Now, [https://fishsticksonarod.s3-eu-central-1.amazonaws.com/index.html]() should work (pinky-swear).

**PS!** Notice how index.html uses style.css in the **folder** static. How will this work out when we have a React router? (secret: We're getting to that soon)

## Host a */website/* on S3
1. **Checkout the master branch**
This contains a simple React-Redux app, using among other things a Router.
```
git checkout master
```

2. **Compile the app**
We want to compile this app locally first, before uploading the compiled files.
```
npm install
npm run build
```

3. **Upload and overwrite**
When we uploaded the previously, perhaps you noticed that we uploaded the ```.git``` folder, among other things. We can exclude this, by adding ```--exclude ".git/*"``` to the sync command. We can also add ```--delete```, which deletes any files at the destination which is not in the source.
```
aws s3 sync ./dist/ s3://fishsticksasbait --region=eu-central-1 --exclude ".git/*" --delete
```
You can checkout [https://fishsticksonarod.s3-eu-central-1.amazonaws.com/index.html]() to see the app.

4. **Enable website hosting**
As you may have noticed, the endpoint [https://fishsticksonarod.s3-eu-central-1.amazonaws.com]() does not work. We can make index.html our default index document, by turning on the website hosting feature.
&nbsp;
You can do this through AWS Console > FishSticksAsBait > Properties > Static Website Hosting > Enable website hosting, or through the [aws cli](http://docs.aws.amazon.com/cli/latest/reference/s3/website.html).
&nbsp;
Set ```index.html``` as the index and error document.
&nbsp;
Notice now that [HTTP://fishsticksonarod.s3-website.eu-central-1.amazonaws.com/]() works, while [HTTPS://fishsticksonarod.s3-website.eu-central-1.amazonaws.com/]() does not!

Losing HTTPS in not acceptable. There's a few other quirks with using S3 as a direct endpoint (or CNAME'ed to), but they can all be remedied with CloudFront.

## Adding Cloudfront
Cloudfront is a cache layer / CDN service from AWS. It helpes us with certain aspects. It simplifies HTTPS and the use of certificates. It optimizes latency, through distributing cache to different location. And it can gzip your content, and lower your costs.

Also, you can combine it with Cloudtrail for detailed logging, as well as generate a more condence report on the traffic to the Cloudfront distributions.

1. **Create a Cloudfront distribution**
Go to Cloudfront in the [AWS Console](https://console.aws.amazon.com/cloudfront).
Select your S3 bucket (fishsticksonarod.s3.amazonaws.com) as Origin Domain Name. Also, set *Compress Objects Automatically* to *Yes*, and *Default Root Object* to *index.html* Other than that, you can go for the defaults :)
&nbsp;
**Notable configuration options**
- *Restrict bucket access*: You can make the bucket only accessable via cloudfront
- *Viewer Protocol Policy*: You can auto redirect HTTP to HTTPS
- *Allowed HTTP Methods*: You can allow all methods, but cloudfront will never cache methods other than GET, HEAD (optionally OPTIONS)
- *Forward Headers*: Forwarding all headers disables caching. Both this and forwarding cookies should be *None* or *Whitelist* if possible.
- *Alternate Domain Names*: Which domains that can be allowed to CNAME here
- *SSL Certificate:* You can request a (free) SSL certificate from AWS for this service, and it will be automatically updated before it expires. (You can also provide your own certificate). Be aware though, that the SSL certificates must come from US East (N. Virginia) Region.
&nbsp;
The distribution will take ~15 minutes to boot up before being accessable.

2. **Handling /path in app**
In the app we created, there's an route available on the path ```/test```.
&nbsp;
By default, both S3\* and Cloudfront will detect this as a 404, as there is no file called ```test```. For us to instead return index.html, select your *Cloudfront distribution*, and click *Distribution settings*. Under *Error Pages*, create a custom response for 404, which instead should return path ```/index.html``` with a 200 status code. (Note that this change will also take ~15 min)
&nbsp;
*\*The S3 website at [HTTP://fishsticksonarod.s3-website.eu-central-1.amazonaws.com/]() will be able to return index.html, as it is it's default error object. However, due to the 404 status code that is returned with it, some strange behaviour can appear in certain versions of Internet Explorer.*

## That's it!
Your app is up and running, and it can take a beating. It's response time is great, and it can with ease support custom domains and SSL to those.

You can try benchmarking Cloudfront vs S3 via ```Developer tools``` -> ```Network``` in Chrome. As long as the S3 bucket was placed in the AWS region nearest you, you might not experience much of a difference in terms of latency. However, since Cloudfront gzips your files you can still see a gain. Here's my tests for ```bundle.js``` over 100 requests:

| Service / region | Size           | Average   |
| ---------------- |:--------------:| ---------:|
| S3 US west       | 917KB          | 1750.8 ms |
| S3 EU central    | 917KB          | 610.62 ms |
| Cloudfront       | 166KB          | 353.64 ms |
| S3 US west       | 917KB (cache)  | 609.68 ms |
| S3 EU central    | 917KB (cache)  | 198.01 ms |
| Cloudfront       | 166KB (cache)  | 139.54 ms |

**What about pricing?**
Pricing in AWS is difficult. But the bottom line is: It's cheap. For hobby usage, I'll bet you a beer it costs less than a beer. And that you'll probably pay less with Cloudfront than without. See more at:

- [amazon.com/s3/pricing/](https://aws.amazon.com/s3/pricing/)
- [amazon.com/cloudfront/pricing/](https://aws.amazon.com/cloudfront/pricing/)

## Extras
If you want something more to do, you could either:

1. Use Route53 to manage a domain of yours, and forward it to your Cloudfront distribution
2. Add SSL to your distribution, using Amazon Certificate Manager.

Or you could:

1. Change the React app, so it uses data from [the beanstalk workshop](https://github.com/helleroy/beanstalk-workshop) instead of from 500px
2. Add a Cloudfront distribution in front of your backend
