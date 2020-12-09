# Project 2: Tracking User Activity

Scenario:
In this project, I work at an ed tech firm. I've created a service that
delivers assessments, and now lots of different customers (e.g., Pearson) want
to publish their assessments on it. I need to get ready for data scientists
who work for these customers to run queries on the data. 

# Tasks

Prepare the infrastructure to land the data in the form and structure it needs
to be to be queried by the following task:

- Publish and consume messages with Kafka
- Use Spark to transform the messages. 
- Use Spark to transform the messages so that you can land them in HDFS

In order to show the data scientists at these other companies the kinds of data
that they will have access to, decide on 1 to 3 basic business questions that
you believe they might need to answer about these data.


# Process 
I first used docker-compose to create a cluster, then created a Kafka topic and subscribed to it using zookeeper as its manager.
I then created spark dataframe by reading kafka topic into spark. The values of each topic is then casted into string into another dataframe. 
From there, I extracted and unrolled the json data into other smaller dataframes which I registered as temporary tables. I used Spark SQL and 
dataframe methods to query against them. I wrote the temporary tables to Hadoop HDFS for later access like scaling up SQL. 


# What's in the repo 

- A history file of the console 

- A report as a jupyter notebook.
  The report contains Linux commands that were used to spin up the pipeline.
  It contains queries and Spark SQL to answer business questions.
  The report also contains notes on my assumptions, my thinking.
  The data is a multi-level nested json. We are given the data and some starter codes.
  I added enhancements to get the data transformed in the format that is friendlier to query.
  There is an issue with the data in that there are some holes which exist within the key, value
  dictionary format. I found that using PySpark flatMap() helps flatten and delete holes in the final result.
  There are six parts that I will walk through in the notebook:  
  
  1) create a data frame by subscribing to the kafka topic
  2) convert the json data as a string to a new dataframe
  3) extract / unrolls the json data into new dataframes to answer you business questions
  4) register dataframes as temporary tables to allow in memory queries against them
  5) perform SQL queries against the dataframes you registered
  6) perform a write to HDFS in parquet format for each data frame you created
 
- A docker-compose.yml

- A json file 

- Misc logs and Jupyter checkpoints

Comment you code with explanations of what you do, especially the steps to spin up your pipeline.


## Data

The data is curled by using the following command 
```
curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp`
```

## Business questions

1. How many assesstments are in the dataset?
2. What are the least common courses taken?
3. What are the most common courses taken?
4. What are some courses with the most number of questions?
5. What are the summary statistics of average scores for all courses? 
6. What are courses with the highest number of perfect score?
7. What are courses with highest number of scores lower than 50?
8. What are courses with highest number of incomplete and unanswered questions?

