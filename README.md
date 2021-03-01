[![Build Status](https://travis-ci.com/seahen/maven-s3-wagon.svg?branch=master)](https://travis-ci.com/seahen/maven-s3-wagon)
[![Maven Central](https://img.shields.io/maven-central/v/com.github.seahen/maven-s3-wagon.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.seahen/maven-s3-wagon/)
[![javadoc.io](https://javadocio-badges.herokuapp.com/com.github.seahen/maven-s3-wagon/badge.svg)](https://www.javadoc.io/doc/com.github.seahen/maven-s3-wagon)
[![license](https://img.shields.io/github/license/seahen/maven-s3-wagon.svg)]()
    

# Maven S3 Wagon

This wagon enables communication between Maven and Amazon S3.

pom's with a reference to this wagon can publish build artifacts (.jar's, .war's, etc) to S3.

When uploading the contents of a directory, API calls to S3 are multi-threaded.

This allows directories with a lot of content (eg when invoking mvn site-deploy) to be published very quickly

Check [Maven Central](http://search.maven.org/#search|gav|1|g%3A%22com.github.seahen%22%20AND%20a%3A%22maven-s3-wagon%22) for the latest version

This project is based on https://github.com/jcaddel/maven-s3-wagon/

# Documentation

## Usage
Configuring Maven to use the wagon is very simple.

You will need 3 things from Amazon: 

1. S3 bucket name - For example, the [Kuali](http://www.kuali.org/) S3 bucket is called - `maven.kuali.org`
2. AWS Access Key ID - eg - `JKCAIZQRXJVCMWNYAZ2Q` - for reference only, not a real id.
3. AWS Secret Access Key - eg - `aIKJN9sL9cu3GsHoti0mqcbH4NNLDCthsn0lms1x` - for reference only,  not a real secret key.

Once you have that information follow these 3 steps to configure Maven to use the wagon.

Add this to the build section of a pom:

    <build>
      <extensions>
        <extension>
          <groupId>com.github.seahen</groupId>
          <artifactId>maven-s3-wagon</artifactId>
          <version>[S3 Wagon Version]</version>
       </extension>
      </extensions>
    </build>

Add this to the distribution management section:

    <distributionManagement>
      <site>
        <id>s3.site</id>
        <url>s3://[S3 Bucket Name]/site</url>
      </site>
      <repository>
        <id>s3.release</id>
        <url>s3://[S3 Bucket Name]/release</url>
      </repository>
      <snapshotRepository>
        <id>s3.snapshot</id>
        <url>s3://[S3 Bucket Name]/snapshot</url>
      </snapshotRepository>
    </distributionManagement>
  
And setup one of the supported authentication techniques (see below)

If things are setup correctly, `$ mvn deploy` will produce output similar to this:

    [INFO] --- maven-deploy-plugin:2.7:deploy (default-deploy) @ kuali-example ---
    [INFO] Logged in - maven.kuali.org
    Uploading: s3://maven.kuali.org/release/org/kuali/common/kuali-example/1.0.0/kuali-example-1.0.0.jar
    [INFO] Logged off - maven.kuali.org
    [INFO] Transfers: 1 Time: 2.921s Amount: 7.6M Throughput: 2.6 MB/s
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------


## Authentication
The wagon supports multiple methods for authenticating with Amazon S3.

1 - System Properties: `aws.accessKeyId` + `aws.secretKey`

2 - Environment Variables: `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`

3 - IAM roles for service accounts (IRSA): see [AWS EKS User Guide - IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) for more information

4 - Entries in `~/.m2/settings.xml`

    <servers>
      <server>
        <id>[repository id]</id>
        <username>[AWS Access Key ID]</username>
        <password>[AWS Secret Access Key]</password>
      </server>
    </servers>

5 - The default AWS credential profiles file: typically located at `~/.aws/credentials` (location can vary per platform), and shared by many of the AWS SDKs and by the AWS CLI.

6 - Amazon ECS container credentials: loaded from the Amazon ECS if the environment variable AWS_CONTAINER_CREDENTIALS_RELATIVE_URI is set.

7 - Instance profile credentials: used on EC2 instances, and delivered through the Amazon EC2 metadata service.

The priority used when searching for credentials is the same as the order listed on this page.  For example, if a set of credentials is found in the system properties, the next three areas are never checked.  If no credentials can be found, an exception is thrown.l


## Permissions
Files are uploaded with no particular permission set by default (so they will take on the bucket default configuration), but that can be overridden through configuration.

Add `<filePermissions>` to the configuration for a server in `~/.m2/settings.xml` to change the default. For example:

    <server>
      <id>s3.snapshot</id>
      <filePermissions>AuthenticatedRead</filePermissions>
    </server>

The allowed values for `<filePermissions>` are:

     Private
     PublicRead
     PublicReadWrite 
     AuthenticatedRead 
     LogDeliveryWrite
     BucketOwnerRead
     BucketOwnerFullControl

The javadoc for [`CannedAccessControlList`](http://docs.amazonwebservices.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/model/CannedAccessControlList.html) has details on what each value means.
