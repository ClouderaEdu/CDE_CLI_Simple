# CDE_CLI_Simple


## Objective

This is a basic introduction to the CDE CLI. It is meant to be a quickstart for anyone starting from scratch.

For a more detailed resource or a reference for converting Spark Submits to CDE Spark Submits please visit this [CDE CLI Demo](https://github.com/pdefusco/CDE_CLI_demo).


## Requirements

This demo requires 

* A CDE Virtual Cluster in CDP Private or Public Cloud (Azure or AWS)
* Installing the CDE CLI on your local machine. Please visit [these instructions](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-download-cli.html) for help.
* Basic familiarity with Spark, Python, and SQL will help. However, no modifications to the code are required. 
* Preloading data to Cloud Storage is not required as the PySpark job will create the data directly.


## Project Setup

Clone this GitHub repository to your local machine or download the files directly from here. 

To clone this GitHub repository, create and navigate to a local directory and then run

```
git clone https://github.com/pdefusco/CDE_CLI_Simple.git
```


## Using the CDE CLI

#### Step 1: Familiarize yourself with the CDE CLI options

On your local machine where you have installed the CLI, issue the following command.

```
cde --help
```

Review the output. Notice that there are eight high level command categories.

```
Available Commands:
  airflow     Airflow commands
  backup      Create and Restore CDE backups
  credential  Manage CDE credentials
  help        Help about any command
  job         Manage CDE jobs
  resource    Manage CDE resources
  run         Manage CDE runs
  spark       Spark commands
```

The *airflow* and *spark* commands will allow you to trigger a CDE Airflow DAG or CDE Spark Submit. 
These are useful for submitting a job to CDE with minimum work. For example, you may have developed a new Spark job and you would like to give it a quick test run.
You can learn more by issuing:

```
cde airflow --help
```

or 

```
cde spark --help
```

The *job* command allows you to create a CDE Job. CDE Jobs can be of two types: "Airflow" or "Spark". Why create a CDE Job if you can just use the previous commands?
A CDE Job allows you to specify CDE options, for example setting an execution schedule or listing previous runs.
You can learn more by issuing:

```
cde job --help
```

The *resource* command allows you to create a CDE Resource. CDE Resources are used to upload files, dependencies, requirements and more to the CDE Virtual Cluster.
For more on CDE Resources please visit [this link](https://github.com/pdefusco/CDE_CLI_demo#jobs-resources-python-environments)
Try:

```
cde resource --help
```

The *run* command allows you to describe, list and learn more about each execution. It can be used in the context of CI/CD, integrations with 3rd party tools, and Job Observability.

```
cde run --help
```


#### Step 2: Run a Simple Spark Submit

Issue the following command to run the local PySpark file in CDE. Notice this does not require the data being in cloud storage as a prerequisite.

```
cde spark submit sql.py
```


#### Step 3: Create a CDE Resource

Create a CDE Resource of type *file* with the following command. The resource will be named "cde_cli_simple".

```
cde resource create --name cde_cli_simple
```


#### Step 4: Upload the Spark Job File to the CDE Resource

Load the PySpark file to the CDE Resource. The command assumes the sql.py file is located in your local directory.

```
cde resource upload --name cde_cli_simple --local-path "sql.py" --resource-path "sql.py"
```


#### Step 5: Instantiate the Spark CDE Job

Once the file is located in the CDE Resource you can create a CDE Job with it. 

You can optionally add more options such as a recurrent schedule as shown below. 

```
cde job create --name "sql_job" --type "spark" --application-file "sql.py" --cron-expression "0 */1 * * *" --schedule-enabled "true" --schedule-start "2022-08-09" --schedule-end "2022-10-02" --mount-1-resource "cde_cli_simple"
```


#### Step 6: Trigger Execution of the Spark CDE Job

With the following command the Spark CDE Job will be triggered immediately in the CDE Virtual Cluster. 

```
cde job run --name "sql_job" --application-file "sql.py"
```


#### Step 7: Confirm Run and Download Logs

Issue the following commands to query the CDE Virtual Cluster and obtain the CDE Job runs. 

```
cde run list
```

#### Step 8: Customize the Spark Log Level

Let's create a new CDE Job with log level of "DEBUG". Notice you could have also used the *cde job update* command to modify the existing job.

```
cde job create --name "cde_cli_job_custom_log_level" --type "spark" --application-file "sql.py" --log-level "DEBUG" --schedule-enabled "false" --mount-1-resource "cde_cli_simple"
```

Now run the job:

```
cde job run --name "cde_cli_job_custom_log_level" --application-file "sql.py"
```

When you search for the job you can add multiple filter conditions as shown below: 

```
cde run list --filter 'status[like]%succeeded%' --filter 'spark.spec.file[eq]sql.py' --filter 'job[eq]cde_cli_job_custom_log_level'
```

The above command will print a dictionary. Take note of the id field at the top.

Finally, issue the following command to collect logs. Make sure to replace the id integer with the one corresponding to your job run.

```
cde run logs --type "driver/stdout" --id 358
```


## Conclusions & Next Steps

CDE is the Cloudera Data Engineering Service, a containerized managed service for Spark and Airflow. 
Top benefits of using CDE include:

* Ability to pick Spark versions
* Apache Airflow Scheduling
* Enhanced Observability with Tuning, Job Analysis and Troubleshooting interfaces.
* Job Management via a CLI and an API. 

If you are exploring CDE you may find the following tutorials relevant:

* [CDE CLI Demo](https://github.com/pdefusco/CDE_CLI_demo): A more advanced CDE CLI reference with additional details for the CDE user who wants to move beyond the basics shown here. 

* [Spark 3 & Iceberg](https://github.com/pdefusco/Spark3_Iceberg_CML): a quick intro of Time Travel Capabilities with Spark 3

* [GitLab2CDE](https://github.com/pdefusco/Gitlab2CDE): a CI/CD pipeline to orchestrate Cross Cluster Workflows - Hybrid/Multicloud Data Engineering

* [CML2CDE](https://github.com/pdefusco/CML2CDE): a CI/CD Pipeline to deploy Spark ETL at Scale with Python and the CDE API

* [Postman2CDE](https://github.com/pdefusco/Oozie2CDE_Migration): using the Postman API to bootstrap CDE Services



