# DB Connect Preview Instructions for usage
---
# Introduction
Databricks Connect allows you to connect your favorite IDE (IntelliJ, Eclipse, Pycharm, etc), notebooks (Zeppelin, Jupyter) and other custom applications to Databricks clusters and run Spark code. Typically, you write code in IDE, compile them as a jar, upload the jar to Databricks and then use a notebook or Databricks job UI/API to run the jar. This is a very cumbersome process during development. With Databricks Connect, you can now run the code directly from your favorite applications.

This will allow you to:
Iterate quickly when you want to test your Spark code on a medium-large dataset during development in your IDE
Step through and debug the code in your IDE even when working with a remote cluster.

Databricks Connect requires you to install a Databricks provided client on your machine and setup a cluster with a custom Databricks runtime image. The rest of the readme will walk you through the steps to get started.   **Databricks Connect** is still in *preview* and we look forward to any feedback you have during this process.   Many of the manual steps listed below will be automated before GA.

## Server Setup
**Step 1:** *Login and activate custom runtime versions*

Log into Databricks, and paste the following in a JS console (on Chrome on Mac, the keyboard shortcut is CMD-ALT-J) to activate custom spark versions:
```javascript
window.prefs.set("enableCustomSparkVersions", true)
```

![Activate Custom Versions](https://github.com/ToddGreenstein/dbconnect/blob/master/images/serverSetup.png)

**Step 2:** *Create cluster with custom runtime version*
* Click on “Create Cluster” and enter the custom Spark version as (copy and paste this exact string):  
```html
custom:custom-local__next-major-version-scala2.11__dev__master__6a23f76__27d3334__dbe0.5__fef2363__format-2.lz4
```
![Custom Version ](https://github.com/ToddGreenstein/dbconnect/blob/master/images/customRuntime.png)
* Make sure to set the cluster’s python version to be the same as the one in local environments where users will connect their IDE's from (laptop or desktop machine etc).  This is crucial, even a minor version mismatch means the client on a local workstation where an IDE is running will have environment connectivity issues trying to connect to the cluster.   
* Add the following Spark conf:
```javascript
spark.databricks.service.server.enabled true
```
if running in Azure you will also need to add the following Spark conf:
```javascript
spark.databricks.service.port 8787
```
![Spark Config](https://github.com/ToddGreenstein/dbconnect/blob/master/images/customRuntime2.png)

## Client Setup
**Step 1:** *Download the Client*

* The client, which is an SDK that allows you to connect to remote clusters, can be downloaded from

 https://drive.google.com/file/d/1O8cH1DJqd21P1fdraDpTGMogbUcqt0Rm/view?usp=sharing
* Unpack it (unzip <filename.zip>) On a Windows OS, make sure that the client full path does not contain any spaces.  
* Currently in the preview when unpacked, it acts as a standalone SDK that has no default integration into your system path, or environment variables.   This will change with the GA version.   
* Once unpacked, if your development is python based you will need to install pyspark.   This can be done from the SDK directory from the python folder
```bash
pip install -e . --user  
```   
* *Note on Python Virtual Environments*: The client requires the same .minor version of python that the server is running.  Example: a local client running in and environment using 3.6 will not run against a cluster running 3.5. If you're using conda on your local development environment, and your cluster is running python 3.5, you will need to create an environment locally with that identical version, like the following:
```bash
conda create --name dbconnect python=3.5
```
* Confirm you have java 8 installed.  The Client does not support Java 11.  We test against java 8.  This will change with the GA Version and will support Java 11

**Step 2:** *Configure connection properties*

To connect to a Databricks cluster, 3 things are required:
1. **Databricks URL Endpoint** (e.g.: “https://<your org name>.cloud.databricks.com”)
1. **Databricks User token:** After login to Databricks, from the top right user menu, you can click on “User Settings” and then go to “Access Tokens” tab to generate a new token.  Make sure you copy and paste this token when it's displayed on the screen, as this will be the only opportunity to capture it.  
![Token](https://github.com/ToddGreenstein/dbconnect/blob/master/images/token.png)
![Token](https://github.com/ToddGreenstein/dbconnect/blob/master/images/token2.png)
1. **Databricks Cluster ID:** You also need the cluster ID you created with the custom runtime version. You can obtain the cluster ID from the URL as shown:
![Cluster ID](https://github.com/ToddGreenstein/dbconnect/blob/master/images/clusterID.png)

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
// Only necessary on consolidated and Azure
//.config("spark.databricks.service.port", "8787")
// Only do this on Azure
//.config("spark.databricks.service.orgId", "83127xxxxxxxx")
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
// Only necessary on consolidated and Azure
// .config("spark.databricks.service.orgId", "83127xxxxxxxx")
// Only do this on Azure
// .config("spark.databricks.service.port", "8787")
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
# Only necessary on consolidated and Azure
#.config("spark.databricks.service.orgId", "83127xxxxxxxx")\
# Only do this on Azure
#.config("spark.databricks.service.port", "8787")\
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
// Only necessary on consolidated and Azure
// .config("spark.databricks.service.orgId", "83127xxxxxxxx")
// Only do this on Azure
//.config("spark.databricks.service.port", "8787")
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
.builder\
.config("spark.databricks.service.client.enabled", "true")\
.config("spark.databricks.service.address", "https://xxx.cloud.databricks.com")\
	.config("spark.databricks.service.token", "dapixxxxxxxx")\
	.config("spark.databricks.service.clusterId", "0611-211525-xxxxx")\
# Only necessary on consolidated and Azure
# .config("spark.databricks.service.orgId", "83127xxxxxxxx")\
# Only do this on Azure
# .config("spark.databricks.service.port", "8787")\
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
## Instructions for specific IDEs/development environments
### Pycharm
You will need to select the interpreter and add the SDK into the path.  This is done by navigating to Pycharm > Preferences > Project Interpreter > Show All...
![pycharm Intepreter](https://github.com/ToddGreenstein/dbconnect/blob/master/images/pycharmPath.png)
![pycharm intepreter](https://github.com/ToddGreenstein/dbconnect/blob/master/images/pycharmPath2.png)
Add the path of the downloaded Python folder, for eg:
```bash
/home/eric/Downloads/spark-2.4.0-SNAPSHOT-bin-sdk-demo/python/
```
Make sure that the directory you add is at the top/front of the PYTHONPATH. In particular, it must be ahead of any other installed version of Spark (otherwise you will either use one of those other Spark versions and run locally or throw a ClassDefNotFoundError).
![pycharm intepreter](https://github.com/ToddGreenstein/dbconnect/blob/master/images/pycharmPath3.png)

Install pyforj:
![pycharm py4j](https://github.com/ToddGreenstein/dbconnect/blob/master/images/py4j.png)

**For Python 3 clusters:**
Navigate to Run > Edit Configurations
Add PYSPARK_PYTHON=python3 as an Environment variable

![pycharm Python 3](https://github.com/ToddGreenstein/dbconnect/blob/master/images/Python3Env.png)

### IntelliJ (Scala or Java only)
We need to point the dependencies to the SDK we downloaded.  
Navigate to File -> Project Structure (On the mac, Cmd + ;) -> Modules -> Dependencies-> ‘+’ sign -> JARs or Directories

![Intellij Jars](https://github.com/ToddGreenstein/dbconnect/blob/master/images/intellijJars.png)

Make sure that the jars you add are at the top/front of the classpath. In particular, they must be ahead of any other installed version of Spark (otherwise you will either use one of those other Spark versions and run locally or throw a ClassDefNotFoundError).

Next, check the setting of the breakout option in Intellij.  It should be set to "Thread" to avoid stopping the background network threads.  "All" is the default and will cause network timeouts.  

![Intellij threads ](https://github.com/ToddGreenstein/dbconnect/blob/master/images/intellijThread.png)

# Eclipse
We need to point to the downloaded SDK.  To do that navigate to: Project menu -> Properties -> Java Build Path -> Libraries -> Add External Jars..

![Eclipse Jars ](https://github.com/ToddGreenstein/dbconnect/blob/master/images/eclipse.png)

Next, make sure that the jars you add are at the top/front of the classpath. In particular, they must be ahead of any other installed version of Spark (otherwise you will either use one of those other Spark versions and run locally or throw a ClassDefNotFoundError).

![Eclipse Jars ](https://github.com/ToddGreenstein/dbconnect/blob/master/images/eclipse2.png)

### VS Code
Verify that Python extension is installed
https://marketplace.visualstudio.com/items?itemName=ms-python.python

Select a Python interpreter from the Command Palette (Command+Shift+P on macOS and Ctrl+Shift+P on Windows/Linux)

Navigate to Code > Preferences > Settings, and choose "python settings"

Add the SDK’s python folder path to the User Settings JSON under python.venvPath
This should be added to the Python Configuration.   You will also need to disable the linter for the preview.   Once in user settings, click the ... on the right side and "edit json settings".   The modified settings are as follows:

![VS Code ](https://github.com/ToddGreenstein/dbconnect/blob/master/images/vscode.png)

If running with a virtual environment, which is the recommended way to develop for python in VS Code.   In the Command Palette (Command+Shift+P on macOS and Ctrl+Shift+P on Windows/Linux) type "select python interpreter" and point to your environment that *matches* your cluster python version.  Example: if your cluster is python 3.5, your local environment should by python 3.5.

![VS Code ](https://github.com/ToddGreenstein/dbconnect/blob/master/images/selectintepreter.png)

![VS Code ](https://github.com/ToddGreenstein/dbconnect/blob/master/images/python35.png)

Ensure PySpark is installed / in path by executing the following in the PYTHON folder.
pip install -e . --user        

# Accessing DBFS

DB connect provides access to DBFS through the standard Hadoop filesystem interface. You can use this the filesystem like so:

```scala
> import org.apache.hadoop.fs._

// get new DBFS connection
> val dbfs = FileSystem.get(spark.sparkContext.hadoopConfiguration)
dbfs: org.apache.hadoop.fs.FileSystem = com.databricks.backend.daemon.data.client.DBFS@2d036335

// list files
> dbfs.listStatus(new Path("dbfs:/"))
res1: Array[org.apache.hadoop.fs.FileStatus] = Array(FileStatus{path=dbfs:/$; isDirectory=true; ...})

// open file
> val stream = dbfs.open("dbfs:/path/to/your_file")
stream: org.apache.hadoop.fs.FSDataInputStream = org.apache.hadoop.fs.FSDataInputStream@7aa4ef24

// get file contents as string
> import org.apache.commons.io._
> println(new String(IOUtils.toByteArray(stream)))
```
