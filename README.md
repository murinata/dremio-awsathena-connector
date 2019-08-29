# Dremio ARP AWS Athena Example Connector


## Overview

The Advanced Relational Pushdown (ARP) Framework allows for the creation of Dremio plugins for any data source which has a JDBC driver and accepts SQL 
as a query language. It allows for a mostly code-free creation of a plugin, allowing for modification of queries issued 
by Dremio using a configuration file.

There are two files that are necessary for creation of an ARP-based plugin: the storage plugin configuration, which 
is code, and the plugin ARP file, which is a YAML (https://yaml.org/) file.

The storage plugin configuration file tells Dremio what the name of the plugin should be, what connection options 
should be displayed in the source UI, what the name of the ARP file is, which JDBC driver to use and how to make a 
connection to the JDBC driver.

The ARP YAML file is what is used to modify the SQL queries that are sent to the JDBC driver, allowing you to specify 
support for different data types and functions, as well as rewrite them if tweaks need to be made for your specific 
data source. For this Example , SNOWFLAKE DREMIO Connector YAML is cloned with one change as AWS ATHENA doesn't support OFFSET as of now.

dremio snowflake connector is here https://github.com/narendrans/dremio-snowflake, without this work , i wouldn't able to cook up ATHENA this easily. It literally took 1/2 day to code and test for a day.

## ARP File Format

The ARP file is broken down into several sections:

**metadata**
- This section outlines some high level metadata about the plugin.

**syntax**
- This section allows for specifying some general syntax items like the identifier quote character.

**data_types**
- This section outlines which data types are supported by the plugin, their names as they appear in the JDBC driver, and how they map to Dremio types.

**relational_algebra** - This section is divided up into a number of other subsections:

- **aggregation**
  - Specify what aggregate functions, such as SUM, MAX, etc, are supported and what signatures they have. You can also specify a rewrite to alter the SQL for how this is issued.
- **except/project/join/sort/union/union_all/values**
  - These sections indicate if the specific operation is supported or not.
- **expressions**
  - This section outlines general operations that are supported. The main sections are:
- **operators**
  - Outlines which scalar functions, such as SIN, SUBSTR, LOWER, etc, are supported, along with the signatures of these functions which are supported. Finally, you can also specify a rewrite to alter the SQL for how this is issued.
- **variable_length_operators**
  - The same as operators, but allows specification of functions which may have a variable number of arguments, such as AND and OR.

If an operation or function is not specified in the ARP file, then Dremio will handle the operation itself. Any operations which are indicated as supported but need to be stacked on operations which are not will not be pushed down to the SQL query.

## Building and Installation
*NOTE - Build in Linux system , Project built in windows might throw runtime exceptions*
1. In root directory with the pom.xml file run `mvn clean install`
2. Take the resulting .jar file in the target folder and put it in the \dremio\jars folder in Dremio
3. Take the AWS Athena JDBC driver from (https://s3.amazonaws.com/athena-downloads/drivers/JDBC/SimbaAthenaJDBC_2.0.7/AthenaJDBC42_2.0.7.jar) and put in in the \dremio\jars\3rdparty folder
4. Make sure AWS Credentials are available eitehr in ENVIRONMENT VARIABLES, Profile file ( ~/.aws/credentails) or the Sever has AWSInstanceMetadata Service enabled

## Usage
1. Select Source and Select AWS Athena ( should appear first)
3. Give a Name, Make sure your enter the Region ( e.g;- ap-southeast-2, us-west-2 etc), Leave AWSCredentialProdiver as DefaultChain ( Unless you want to change and know how the various credential provider works) and S3 Path (S3://<bucketname>/<path>) where the crendentials has full write acess ( AWS Athena works by first committing the result set in S3 Path and then to the JDBC / Dremo)
4. Restart Dremio

AWS Region info ( https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)

AWS Credential Provider - DefaultCredentialProviderChaininfo - Shows the order in which it looks for Credentials. (https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)

S3 Ouput Locaion to have the S3 Path where the credentials as write access.



