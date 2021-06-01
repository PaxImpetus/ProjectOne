# Project One - Complex Data Querying in Beeline Hive

## Project Description

Our objective is to perform six sets of queries on the six data sets found in this GitHub repository.

	https://github.com/Haripriya6/Sample-HIVE-Project/blob/master/Sample%20HIVE%20Project.pdf

The link provided above includes six sets of data
  * Three sets of data contain an ordered pair of branch names and drink names
    * bev_branch_a
    * bev_branch_b
    * bev_branch_c
  * Three sets of data contain an ordered pair of drink names and customers purchased
    * cons_count_a
    * cons_count_b
    * cons_count_c

Some query prompts require more than one solution set.
The following bullets describe each query problem
1. Determine the number of customers for Branches 1 _and_ 2 
2. Determine the most consumed beverage in Branch 1 _and_ least in 2
3. Determine all beverages available in Branches 10, 8, 1 _and_ common beverages between 4 and 7
4. Create a **partition**, **index** and **view** for both components of the previous query
5. Alter the Table Properties of any pertinent table to include a note or comment
6. Remove an entity from the first query


## Features

### Completion
1. Generated a physical table listing the total customers for all branches
2. Generated two tables, each respecting their corresponding drink-count pair for a given branch
3. Generated two tables
  a. The table related to branches 10, 8, and 1 iterate all beverages sold across the branches - duplicates are ignored
  b. The table related to branches 4 and 7 iterate all beverages sold at both branches - unique beverages are ignored
4. For 3a and 3b, created a partition, view, and index for each
5. Altered two tables to include descriptions in the form of a note or string
6. Remove row 5 from the table generated in query 1
  * Of note, the original query was to remove the 5th element from the first query result set, but the first query only asks for two elements, not five.
  * To supplement, I created a table that represents all branch results, then removed the fifth elemnt, Branch 5

### TODO
* Wrap the code in a Begin-End Pair
* Organize the data such that outputs are sequential with respect to the ordered query list


## Getting Started

### About the Data
Tables with like names _can_ be united, as each row is independent of its neighbors.
Additionally, some rows be be duplicates, and we do not want to neglect those duplicates.
However, the two schemas do **not** include keys that directly link data.
Rather, consider the following:
  * a set of branches {n | n<sub>1</sub>, n<sub>2</sub>, ... n<sub>x</sub>} that each sell a drink {m}, where x is the branch number
  * drink {m} has multiple values (i.e.) { m | 234, 33, 0, 674}, where each drink-value is its own row
Then, every drink in the set of n gets the values found in m.
When you joun drink-branch on drink-count, all branches get the set of counts if that branch sells the drink.
So, when you compare branches with like drinks, you will notice that these branches sell the same quantities of drinks for the given drink

### Initial Replication Requirements
Uploading Data into Hadoop Distributive File System
1. Download documents from Dataset_for_the_project.zip
2. Unzip documents from download package
3. Move the unzipped folder closer to the C:/ drive (or whatever your drive is named)
4. Move the files into your HDFS using ```hdfs dfs -put /mnt/driveChar/User/Username/.../unzippedDocument /user/userName/.../folderName```
5. Launch Hive one of two ways
  a. type ```hive``` into the command line
  b. in two new windows, launch ```hiveserver2``` in the first and ```beeline -u jdbc:hive2://host:port``` in the second
6. Create and Use a database with the following commands
  * FORM: ```create database dbName;```
  * FORM: ```show dbName;```
  * FORM: ```use dbName;```
7. Create an internal table for each document downloaded, label the tables identically to the documents
  * FORM: ```CREATE TABLE IF NOT EXISTS tblName (branch_name String, drink_name String) ROW FORMAT DELIMITED FIELDS TERMINATED BY ‘,’ STORED AS TEXTFILE;```
8. Load data into the internal table
  * FORM: ```LOAD DATA INPATH '/user/userName/.../folderName/file.type' OVERWRITE INTO TABLE tblName;```


### Supplementary Setup Proposition
Create Three Internal Tables, to make query typing more efficient
  * bev_branch_all	three-table union between bev_branch_a, \_b, and \_c
  * cons_count_all	three-table union between cons_count_a, \_b, and \_c
  * cumulative_set	Left Join on bev_branch_all and cons_count_all

## Usage

A Simple Demonstration of Complex Queries on Unstructured, Unrelated Data.

## Contributors

George Feitosa Kotzabassis - programmer
GitHub User Haripriya6 - data provider

## Liscence

None - data source project is unliscenced
