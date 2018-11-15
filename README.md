# DB Connect Preview
## Instructions for usage
# Introduction
Databricks Connect allows you to connect your favorite IDE (IntelliJ, Eclipse, Pycharm, etc), notebooks (Zeppelin, Jupyter) and other custom applications to Databricks clusters and run Spark code. Typically, you write code in IDE, compile them as a jar, upload the jar to Databricks and then use a notebook or Databricks job UI/API to run the jar. This is a very cumbersome process during development. With Databricks Connect, you can now run the code directly from your favorite applications.

This will allow you to:
Iterate quickly when you want to test your Spark code on a medium-large dataset during development in your IDE
Step through and debug the code in your IDE even when working with a remote cluster.

Databricks Connect requires you to install a Databricks provided client on your machine and setup a cluster with a custom Databricks runtime image. The rest of the document will walk you through the steps to get started.

## Server Setup
**Step 1:** *Login and activate custom runtime versions*

Log into Databricks, and paste the following in a JS console (on Chrome on Mac, the keyboard shortcut is CMD-ALT-J) to activate custom spark versions:
```javascript
window.prefs.set("enableCustomSparkVersions", true)
```
**Step 2:** *Create cluster with custom runtime version*
1. Click on “Create Cluster” and enter the custom Spark version as (copy and paste this exact string):
```javascript
custom:custom-local__next-major-version-scala2.11__dev__master__6a23f76__27d3334__dbe0.5__fef2363__format-2.lz4
```
1. Make sure to set the cluster’s python version to be the same as the one in local environments where users will connect their IDE's from (laptop or desktop machine etc).  This is crucial, even a minor version mismatch means the client on a local workstation where an IDE is running will have environment connectivity issues trying to connect to the cluster.  
1. Add the following Spark conf:
```javascript
spark.databricks.service.server.enabled true
```
if running in Azure you will also need to add the following Spark conf:
```javascript
spark.databricks.service.port 8787
```
![alt text](https://raw.githubusercontent.com/username/projectname/branch/path/to/img.png)
![alt text](https://raw.githubusercontent.com/username/projectname/branch/path/to/img.png)

## Client Setup
**Step 1:** *Download the Client*

* The client can be downloaded from

 https://drive.google.com/file/d/1O8cH1DJqd21P1fdraDpTGMogbUcqt0Rm/view?usp=sharing
* Unpack it (unzip <filename.zip>)
On a Windows OS, make sure that the client full path does not contain any spaces.
* **Note**: The client requires the same .minor version of python that the server is running.  Example: a local client running in and environment using 3.6 will not run against a cluster running 3.5.
* **Note**: The Client does not support java 11.  We test against java 8

**Step 2:** *Configure connection properties*

To connect to a Databricks cluster, 3 things are required:
1. **Databricks URL Endpoint** (e.g.: “https://<your org name>.cloud.databricks.com”)
1. **Databricks User token:** After login to Databricks, from the top right user menu, you can click on “User Settings” and then go to “Access Tokens” tab to generate a new token
1. **Databricks Cluster ID:** You also need the cluster ID you created with the custom runtime version. You can obtain the cluster ID from the URL as shown:

**Step 3:** *Once you have the 3 piece of information, you can now pass this information to your SparkSession builder.*

* **java**

```java
import org.apache.spark.sql.SparkSession;

public class HelloWorld {

	public static void main(String[] args) {
		System.out.println("HelloWorld");
		SparkSession spark = SparkSession
			.builder()
			.master("local")
			.appName("Java Spark SQL basic example")
			.config("spark.databricks.service.client.enabled", "true")
			.config("spark.databricks.service.address", "https://xxx.cloud.databricks.com")
			.config("spark.databricks.service.token", "dapixxxx")
			.config("spark.databricks.service.clusterId", "0611-211525-xxxxx")
                 	.config("spark.databricks.service.orgId", "83127xxxxxxxx")
// Only necessary on consolidated and Azure
//.config("spark.databricks.service.port", "8787")
// Only do this on Azure
			.getOrCreate();

		System.out.println(spark.range(100).count());  
    // The Spark code will execute on the Databricks cluster.

	}
}
```
* **Scala**

```java
import org.apache.spark.sql.SparkSession

object Test {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder()
      	.master("local")
      	.config("spark.databricks.service.client.enabled", "true")
.config("spark.databricks.service.address", "https://xxx.cloud.databricks.com")
	.config("spark.databricks.service.token", "dapixxxxxxxx")
	.config("spark.databricks.service.clusterId", "0611-211525-xxxxx")
       .config("spark.databricks.service.orgId", "83127xxxxxxxx")
// Only necessary on consolidated and Azure
//.config("spark.databricks.service.port", "8787")
// Only do this on Azure
      .getOrCreate()
  println(spark.range(100).count())  
  // The Spark code will execute on the Databricks cluster.
  }
}
```
* **Python**

```python
from pyspark.sql import SparkSession
spark = SparkSession\
.builder\
.config("spark.databricks.service.client.enabled", "true")\
.config("spark.databricks.service.address", "https://xxx.cloud.databricks.com")\
	.config("spark.databricks.service.token", "dapixxxxxxxx")\
	.config("spark.databricks.service.clusterId", "0611-211525-xxxxx")\
	.config("spark.databricks.service.orgId", "83127xxxxxxxx")\ # Only necessary on consolidated and Azure
#.config("spark.databricks.service.port", "8787")\
# Only do this on Azure
.getOrCreate()

print("Testing simple count")
print(spark.range(100).count())
```
* **Configuring via environment variables**  
Alternatively, you can also configure these 3 parameters via environment variables instead of specifying them in code. This will be useful to make sure the tokens and other information doesn’t get into your revision control systems by mistake.

```shell
export DATABRICKS_API_TOKEN=dapidf1742***
export DATABRICKS_ADDRESS=https://dogfood.staging.cloud.databricks.com
export DATABRICKS_CLUSTER_ID=xxxx-xxxx-xxxxx
```
**Step 4:** *Working with dependencies*

Typically, your main class or python file will have other dependency jars and files. You can add such dependency jars and files by calling sparkContext.addJar(“path_to_the_jar”) or sparkContext.addPyFile(“path_to_the_file”). You can also add egg files and zip files with the addPyFile() interface. Everytime, when you run the code in your IDE, the dependency jars and files will be automatically pushed to the cluster.

* **Scala Example**

```java
package com.example

import org.apache.spark.sql.SparkSession

case class Foo(x: String)

object Test {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder()
      	.master("local")
      	.config("spark.databricks.service.client.enabled", "true")
.config("spark.databricks.service.address", "https://xxx.cloud.databricks.com")
	.config("spark.databricks.service.token", "dapixxxxxxxx")
	.config("spark.databricks.service.clusterId", "0611-211525-xxxxx")
       .config("spark.databricks.service.orgId", "83127xxxxxxxx")
// Only necessary on consolidated and Azure
//.config("spark.databricks.service.port", "8787")
// Only do this on Azure
      .getOrCreate();
    spark.sparkContext.setLogLevel("INFO")

    println("Running simple show query...")
    spark.read.parquet("/tmp/x").show()

    println("Running simple UDF query...")
    spark.sparkContext.addJar("./target/scala-2.11/hello-world_2.11-1.0.jar")
    spark.udf.register("f", (x: Int) => x + 1)
    spark.range(10).selectExpr("f(id)").show()

    println("Running custom objects query...")
    val objs = spark.sparkContext.parallelize(Seq(Foo("bye"), Foo("hi"))).collect()
    println(objs.toSeq)
  }
}
```
* **Python**   
*test.py*

```python
from lib import Foo
from pyspark.sql import SparkSession

spark = SparkSession
.builder
.config("spark.databricks.service.client.enabled", "true")
.config("spark.databricks.service.address", "https://xxx.cloud.databricks.com")
	.config("spark.databricks.service.token", "dapixxxxxxxx")
	.config("spark.databricks.service.clusterId", "0611-211525-xxxxx")
       .config("spark.databricks.service.orgId", "83127xxxxxxxx")\
# Only necessary on consolidated and Azure
#.config("spark.databricks.service.port", "8787")\
# Only do this on Azure
.getOrCreate()

sc = spark.sparkContext
#sc.setLogLevel("INFO")

print("Testing simple count")
print(spark.range(100).count())

print("Testing addPyFile isolation")
sc.addPyFile("lib.py")
print(sc.parallelize(range(10)).map(lambda i: Foo(2)).collect())
```
*lib.py*
lib.py

```python
class Foo(object):
  def __init__(self, x):
    self.x = x
```
