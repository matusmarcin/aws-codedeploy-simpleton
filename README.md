# My Quest to Setup AWS CodeDeploy

### Written by Matus Marcin

## Pre-requisites

I need:

* Ability to log in to AWS
* GitHub account

```
ssh -v -i testing-deployment.pem ec2-user@<INSTANCE_IP>
```

## Real work

### Some kind of server

Apache or Nginx?

There's [a tutorial](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-LAMP.html) so might go with Apache. No need for MySQL, so will try to omit that.

Note: 

* Yes, my first instance is that Amazon Linux thingie.
* I ended up doing only Amazon Linux so far. Didn't get to Ubuntu, though it should work (apart from CodeDeploy agent) similarly.

1. Add security rules to allow 80 and 443
2. Connect to your instance and follow the steps to update, install Apache and start it
3. Verify it works (IP or Public DNS of your instance)
4. Rejoice.

Commands: Installs Apache and tests it.

```
sudo yum update -y
sudo yum install -y httpd24 php56 mysql55-server php56-mysqlnd
sudo service httpd start
sudo chkconfig httpd on
chkconfig --list httpd
sudo groupadd www
sudo usermod -a -G www ec2-user
exit

groups
sudo chown -R root:www /var/www
sudo chmod 2775 /var/www
find /var/www -type d -exec sudo chmod 2775 {} +
find /var/www -type f -exec sudo chmod 0664 {} +
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
exit

```

**Repeat this for whatever number of instances you have.**

## Deployment

### CodeDeploy Agent

Amazon Linux how-to:

If you've used CloudFront to generate your instances, this is already done.

```
sudo yum info codedeploy-agent
```

```
sudo yum update
sudo yum install ruby
sudo yum install aws-cli
cd /home/ec2-user
aws s3 cp s3://aws-codedeploy-us-east-1/latest/install . --region us-east-1
chmod +x ./install
sudo ./install auto

```

### GitHub integration

[As outlined in this tutorial.](http://docs.aws.amazon.com/codedeploy/latest/userguide/github-integ-tutorial.html)

### Get a repo and push some stuff

[Done.](https://github.com/matusmarcin/aws-codedeploy-simpleton)

This includes a very simple `appspec.yml`:

```
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
```

### Now what???

[This f***ing document](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-create-service-role.html) or 250 pages in PDF.

**Steps:**

Don't miss any of these.

* [IAM User + Inline Policy](http://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-setup.html)
* [Service Role](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-create-service-role.html)
	* Note the value of ARN (e.g. arn:aws:iam::809827708923:role/CodeDeployDemo)
* [IAM Instance Profile](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-create-iam-instance-profile.html)
* Instances ([using CloudFront](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-use-cloud-formation-template.html#how-to-use-cloud-formation-template-console))
* Apache (see above)
* [Configure instances](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-configure-existing-instance.html) (mostly just checking)
* [Code Deploy Agent](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-run-agent.html) (done already if by CloudFront)

**Final push:**

* [Create application](http://docs.aws.amazon.com/codedeploy/latest/userguide/how-to-create-application.html)
* Deployment group
* Deploy from GitHub
* **Done!**

### Automatic deployment from GitHub

**Next steps:**

* [Automatically deploy from GitHub](http://blogs.aws.amazon.com/application-management/post/Tx33XKAKURCCW83/Automatically-Deploy-from-GitHub-Using-AWS-CodeDeploy)

This tutorial works quite well, just follow it.

* Create a GitHub user in IAM
* Add a policy to use CodeDeploy
* Add an inline policy as in the article but be sure to change all your stuff (region, AWS ID, application and deployment group)
* Create the GitHub part (in two steps, for some reason)

Note: In GitHub Auto-Deployment hook the Environment value is *Deployment Group*.

And that should be all.
