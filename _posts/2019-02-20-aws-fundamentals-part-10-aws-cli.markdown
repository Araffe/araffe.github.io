---
author: Adam
comments: true
date: 2019-02-20 22:20:29+00:00
layout: post
slug: aws-fundamentals-part-10-aws-cli
title: 'AWS Fundamentals - Part 10: Using the AWS CLI'
---

Way back in part 3, I briefly discussed the AWS CLI as a way to interact with the AWS platform. In this post, I want to spend a bit more time looking at the CLI, as once you start to become more comfortable with AWS, it's likely that you'll find the console a bit limiting - mastering the CLI will ultimately make it easier and faster for you when using the platform.

Before I go any further, the first thing you need to do is actually install the CLI - the documentation explaining how to install the CLI on your local machine is available [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html). It's easy to follow so I won't duplicate it here.

Once you have the CLI installed, you'll need to configure it with credentials to talk to your AWS account, as well as specifying the region to use. If you're using your local machine with the CLI installed, have your access key ID and secret access key associated with your AWS account to hand. To configure credentials, run {% highlight shell %}aws configure{% endhighlight %} as follows:

{% highlight shell %}
aws configure
AWS Access Key ID [****************YGXQ]: 
AWS Secret Access Key [****************IJy/]: 
Default region name [eu-west-1]: 
Default output format [None]:
{% endhighlight %}

Once you've configured this, you should be able to access your AWS account via the CLI. Note that the config and credentials are stored in a couple of files in the '~/.aws' directory:

{% highlight shell %}
$ ls ~/.aws
config          credentials
{% endhighlight %}

If I take a look at my credentials file, I see the following (actual credentials removed!!):

{% highlight shell %}
8c8590cd6650:~ adaraffe$ cat ~/.aws/credentials
[default]
aws_access_key_id = ********************
aws_secret_access_key = ********************

[eks-test2]
aws_access_key_id = ********************
aws_secret_access_key = ********************

[nocreds]
aws_access_key_id =
aws_secret_access_key =
{% endhighlight %}

What are the different sections for (default, eks-test-2, nocreds)? These are _profiles_ that I have set up. Each of these profiles contains a different set of credentials (the 'nocreds' profile doesn't contain any credentials at all due to some testing I was doing). To use these profiles, you can set an environment variable that specifies the profile - {% highlight shell %}export AWS_DEFAULT_PROFILE=nocreds{% endhighlight %}. Alternatively, you can specify the profile on each command that you run, for example:

{% highlight shell %}
8c8590cd6650:~ adaraffe$ aws s3 ls --profile=nocreds

An error occurred (AuthorizationHeaderMalformed) when calling the ListBuckets operation: The authorization header is malformed; a non-empty Access Key (AKID) must be provided in the credential.
{% endhighlight %}

As you can see, I've specified the 'nocreds' profile in the above S3 command, which - as you would expect - returns an error as that profile doesn't contain any credentials.

As I mentioned in part 3, AWS CLI commands generally take the following form: firstly, the 'aws' command, followed by the name of the service you want to interact with (e.g. ec2, s3, etc) and then the specific action you want to take. Some examples are as follows:

{% highlight shell %}aws s3 ls{% endhighlight %} - lists all S3 buckets in your account

{% highlight shell %}aws ec2 describe-instances{% endhighlight %} - describes every EC2 instance in your account

{% highlight shell %}aws ec2 start-instances --instance-ids i-079db50f95285d315{% endhighlight %} - starts the instance with the instance ID specified

# Changing the Output Format

By default, the AWS CLI outputs everything in JSON format. Using JSON as the output makes it easy to filter using the --query parameter (desrcibed in the next section) or using other tools such as jq. However, there might be times when you want more of a 'human readable' format, such as a table or just plain text.

Specifying --output=table after the CLI command returns the output as an ASCII table, for example:

{% highlight shell %}
8c8590cd6650:~ adaraffe$ aws ec2 describe-instances --output=table

------------------------------------------------------------------------------
|                              DescribeInstances                             |
+----------------------------------------------------------------------------+
||                               Reservations                               ||
|+------------------------------+-------------------------------------------+|
||  OwnerId                     |  195002507227                             ||
||  ReservationId               |  r-0a296a0349ba97ca8                      ||
|+------------------------------+-------------------------------------------+|
|||                                Instances                               |||
||+------------------------+-----------------------------------------------+||
|||  AmiLaunchIndex        |  0                                            |||
|||  Architecture          |  x86_64                                       |||
|||  ClientToken           |                                               |||
|||  EbsOptimized          |  False                                        |||
|||  EnaSupport            |  True                                         |||
|||  Hypervisor            |  xen                                          |||
|||  ImageId               |  ami-09693313102a30b2c                        |||
|||  InstanceId            |  i-079db50f95285d315                          |||
|||  InstanceType          |  t2.micro                                     |||
|||  KeyName               |  MainKey                                      |||
|||  LaunchTime            |  2019-02-05T14:40:52.000Z                     |||
|||  PrivateDnsName        |  ip-172-31-16-225.eu-west-1.compute.internal  |||
|||  PrivateIpAddress      |  172.31.16.225                                |||
|||  PublicDnsName         |                                               |||
|||  RootDeviceName        |  /dev/xvda                                    |||
|||  RootDeviceType        |  ebs                                          |||
|||  SourceDestCheck       |  True                                         |||
|||  StateTransitionReason |  User initiated (2019-02-05 16:37:32 GMT)     |||
|||  SubnetId              |  subnet-1674155e                              |||
|||  VirtualizationType    |  hvm                                          |||
|||  VpcId                 |  vpc-ed4e6a8b                                 |||
{% endhighlight %}

Alternatively, you could use --output=text to return the information as plain text:
{% highlight shell %}
8c8590cd6650:~ adaraffe$ aws ec2 describe-instances --output=text
RESERVATIONS    195002507227    r-0a296a0349ba97ca8
INSTANCES       0       x86_64          False   True    xen     ami-09693313102a30b2c   i-079db50f95285d315     t2.micro        MainKey              2019-02-05T14:40:52.000Z        ip-172-31-16-225.eu-west-1.compute.internal     172.31.16.225           /dev/xvda  ebs     True    User initiated (2019-02-05 16:37:32 GMT)        subnet-1674155e hvm     vpc-ed4e6a8b
BLOCKDEVICEMAPPINGS     /dev/xvda
EBS     2018-12-18T13:00:43.000Z        True    attached        vol-0a95a9d67ce5cfb87
{% endhighlight %}

You might find it easier to use the text output type if using Linux tools such as grep, sed, awk and so on.

# Using --query to Filter Output

Taking the example in the section above - {% highlight shell %}aws ec2 describe-instances{% endhighlight %}, this potentially returns a lot of data, especially if you are running a lot of EC2 instances in your account. It's highly likely that you are looking for more specific information - for example, you might be looking for the private IP address associated with one specific instance. Fortunately, this is easy to achieve through the use of CLI queries.

Query strings use the [JMESpath](http://jmespath.org/) query specification. To demonstrate how it works, let's look at some examples.

Firstly, let's say I want to find out what the instance type is for each of my EC2 instances. I can get this information using the following command:

{% highlight shell %}
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,InstanceType]' --output=table

---------------------------------------
|          DescribeInstances          |
+----------------------+--------------+
|  i-079db50f95285d315 |  t2.micro    |
|  i-00536c670fdf9b720 |  t2.medium   |
|  i-0f77eeba0d2835444 |  t2.2xlarge  |
+----------------------+--------------+
{% endhighlight %}

Let's look at how that query works. Firstly, the 'Reservations[*].Instances[*]' causes the query to iterate over all the instances in the list. The second half of the query ('[InstanceId,InstanceType]') then returns only the two elements (InstanceID and InstanceType) from each instance. Finally, I specified that I wanted the output to be in table format. I can make the table even more friendly:

{% highlight shell %}
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{Instance:InstanceId,Type:InstanceType}' --output=table

---------------------------------------
|          DescribeInstances          |
+----------------------+--------------+
|       Instance       |    Type      |
+----------------------+--------------+
|  i-079db50f95285d315 |  t2.micro    |
|  i-00536c670fdf9b720 |  t2.medium   |
|  i-0f77eeba0d2835444 |  t2.2xlarge  |
+----------------------+--------------+
{% endhighlight %}

This time, I tweaked the command so that the table would have the headings 'Instance' and 'Type' to make it a bit more readable. Here's another example - what if I wanted to find the private IP address of a specific instance? Here's how I could do it:
{% highlight shell %}
aws ec2 describe-instances --query 'Reservations[*].Instances[?InstanceId==`i-0f77eeba0d2835444`].[PrivateIpAddress]' --output text
172.31.27.82
{% endhighlight %}

In this example, I've searched for a specific instance ID in the first half of the query (using ?InstanceId==`i-0f77eeba0d2835444`) and then in the second half of the query I've asked for only the 'PrivateIpAddress' element to be returned.

# Generating a CLI Skeleton

The AWS CLI has the ability to accept input from a file containing all of the parameters required, using the option --cli-input-json. However, it can be difficult to know which parameters you need and what they are called. To help with that, the CLI provides a way to generate a 'skeleton' set of parameters for a given command, which then allows you to fill it in and use it as input via the --cli-input-json option.

Here's an example: I want to create a new VPC using the CLI - to generate the CLI skeleton, I can do the following:
{% highlight shell %}
aws ec2 create-vpc --generate-cli-skeleton
{
    "CidrBlock": "", 
    "AmazonProvidedIpv6CidrBlock": true, 
    "DryRun": true, 
    "InstanceTenancy": "dedicated"
}
{% endhighlight %}

Now, all I have to do is create a file containing the information in the skeleton with the values filled in (and with 'DryRun' set to false), and then refer to that file in the command:
{% highlight shell %}
aws ec2 create-vpc --cli-input-json file://vpc.json
{% endhighlight %}

Ok, that hopefully gives you a decent overview of the AWS CLI. Thanks for reading!