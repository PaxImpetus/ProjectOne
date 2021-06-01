# ProjectOne
First Official Project for the April 5th 2021 Revature Big Data Training Batch. Included is a link to my presentation, and a README file with documentation explaining the code related to my specific implementation of results
Listed in each section are the Hive queries with their corresponding results


### Initial Database Documentation
```
0: jdbc:hive2://localhost:10000> show databases;
+----------------+
| database_name  |
+----------------+
| default        |
| projectone     |
+----------------+
0: jdbc:hive2://localhost:10000> use projectone;
0: jdbc:hive2://localhost:10000> show tables;
+-----------------+
|    tab_name     |
+-----------------+
| bev_brancha     |
| bev_branchb     |
| bev_branchc     |
| bev_conscounta  |
| bev_conscountb  |
| bev_conscountc  |
+-----------------+
```
	Table properties
		Name			Row Count	Fields
		bev_brancha		100		branch_name, drink_name
		bev_branchb		200		branch_name, drink_name
		bev_branchc		300		branch_name, drink_name
		bev_conscounta		1000 		drink_name, drink_counter
		bev_conscountb		2000 		drink_name, drink_counter
		bev_conscountc		3000 		drink_name, drink_counter

	Project Preparation
	Combine Branch Tables, Consumer Count Tables, and All Tables


	//Combine Branch Tables//
```
0: jdbc:hive2://localhost:10000> create table bev_branch_all as select * from bev_brancha a union all (select * from bev_branchb b union all select * from bev_branchc c);
0: jdbc:hive2://localhost:10000> show tables;
+-----------------+
|    tab_name     |
+-----------------+
| bev_branch_all  |
| bev_brancha     |
| bev_branchb     |
| bev_branchc     |
| bev_conscounta  |
| bev_conscountb  |
| bev_conscountc  |
+-----------------+
0: jdbc:hive2://localhost:10000> select count(*) from bev_branch_all;
+------+
| _c0  |
+------+
| 600  |
+------+
```

	//Combine Consumer Count Tables//
```
0: jdbc:hive2://localhost:10000> create table cons_count_all as select * from bev_conscounta a union all (select * from bev_conscountb b union all select * from bev_conscountc c);
0: jdbc:hive2://localhost:10000> show tables;
+-----------------+
|    tab_name     |
+-----------------+
| bev_branch_all  |
| bev_brancha     |
| bev_branchb     |
| bev_branchc     |
| bev_conscounta  |
| bev_conscountb  |
| bev_conscountc  |
| cons_count_all  |
+-----------------+
0: jdbc:hive2://localhost:10000> select count(*) from cons_count_all;
+-------+
|  _c0  |
+-------+
| 6000  |
+-------+
```
	//Combine All Tables//
```
0: jdbc:hive2://localhost:10000> create table cumulative_set as (select bba.branch_name, bba.drink_name, cca.drink_counter from bev_branch_all bba left join cons_count_all cca on (bba.drink_name = cca.drink_name));
0: jdbc:hive2://localhost:10000> show tables;
+-----------------+
|    tab_name     |
+-----------------+
| bev_branch_all  |
| bev_brancha     |
| bev_branchb     |
| bev_branchc     |
| bev_conscounta  |
| bev_conscountb  |
| bev_conscountc  |
| cons_count_all  |
| cumulative_set  |
+-----------------+
0: jdbc:hive2://localhost:10000> select count(*) from cumulative_set;
+--------+
|  _c0   |
+--------+
| 73634  |
+--------+
```
	// The Cumulative set is the set of all permutations possible on the joining of consumer count and branch distribution



## Problem One

	//Total Number of Customers Everywhere
```
0: jdbc:hive2://localhost:10000> create table p6_target as select branch_name, sum(drink_counter) as totals from cumulative_set cs group by cs.branch_name;
+--------------+----------+
| branch_name  | totals   |
+--------------+----------+
| Branch1      | 1115974  |
| Branch2      | 5099141  |
| Branch3      | 2591062  |
| Branch4      | 3940862  |
| Branch5      | 3941177  |
| Branch6      | 8032799  |
| Branch7      | 6566187  |
| Branch8      | 2583583  |
| Branch9      | 5208016  |
+--------------+----------+
```
	//Total Number of Customers in Branch1
```
0: jdbc:hive2://localhost:10000> select branch_name, sum(drink_counter) as total_sales from cumulative_set cs where cs.branch_name='Branch1' group by cs.branch_name;
+--------------+--------------+
| branch_name  | total_sales  |
+--------------+--------------+
| Branch1      | 1115974      |
+--------------+--------------+
```
	//Total Number of Customers in Branch2
```
0: jdbc:hive2://localhost:10000> select branch_name, sum(drink_counter) as total_sales from cumulative_set cs where cs.branch_name='Branch2' group by cs.branch_name;
+--------------+--------------+
| branch_name  | total_sales  |
+--------------+--------------+
| Branch2      | 5099141      |
+--------------+--------------+
```


## Problem Two

	//Most Consumed Beverage in Branch1
```
0: jdbc:hive2://localhost:10000> select drink_name, most from(select *, rank() over (order by most desc) as row_num from(select drink_name, sum(drink_counter) as most from cumulative_set cs where cs.branch_name='Branch1' group by cs.drink_name order by most desc)abc)xyz where xyz.row_num=1;
+---------------------+---------+
|     drink_name      |  most   |
+---------------------+---------+
| Special_cappuccino  | 108163  |
+---------------------+---------+
```
	//Least Consumed Beverage in Branch2
```
0: jdbc:hive2://localhost:10000> select drink_name, least from(select *, rank() over (order by least asc) as row_num from(select drink_name, sum(drink_counter) as least from cumulative_set cs where cs.branch_name='Branch2' group by cs.drink_name order by least asc)abc)xyz where xyz.row_num=1;
+-------------+--------+
| drink_name  |  most  |
+-------------+--------+
| Cold_MOCHA  | 47524  |
+-------------+--------+
```


##Problem Three

	//What are the beverages available on Branch10, Branch8, and Branch1?
```
0: jdbc:hive2://localhost:10000> create table problemThree_a as select distinct drink_name from (select * from bev_branch_all)i where i.branch_name = 'Branch1' or i.branch_name = 'Branch8' or i.branch_name='Branch10';
+---------------------+
|     drink_name      |
+---------------------+
| Cold_Coffee         |
| Cold_LATTE          |
| Cold_Lite           |
| Cold_cappuccino     |
| Double_Coffee       |
| Double_Espresso     |
| Double_LATTE        |
| Double_MOCHA        |
| Double_cappuccino   |
| ICY_Coffee          |
| ICY_Espresso        |
| ICY_Lite            |
| ICY_MOCHA           |
| ICY_cappuccino      |
| LARGE_Coffee        |
| LARGE_Espresso      |
| LARGE_MOCHA         |
| LARGE_cappuccino    |
| MED_Coffee          |
| MED_Espresso        |
| MED_LATTE           |
| MED_MOCHA           |
| MED_cappuccino      |
| Mild_Coffee         |
| Mild_Espresso       |
| Mild_LATTE          |
| Mild_Lite           |
| Mild_cappuccino     |
| SMALL_Espresso      |
| SMALL_LATTE         |
| SMALL_Lite          |
| SMALL_MOCHA         |
| Special_Coffee      |
| Special_Espresso    |
| Special_LATTE       |
| Special_Lite        |
| Special_MOCHA       |
| Special_cappuccino  |
| Triple_Coffee       |
| Triple_Espresso     |
| Triple_LATTE        |
| Triple_Lite         |
| Triple_MOCHA        |
| Triple_cappuccino   |
+---------------------+
[44 Rows]
```
	//What are the common beverages available in Branch4, Branch7
```
0: jdbc:hive2://localhost:10000> create table problemThree_b as select distinct drink_name from(select * from((select * from bev_branch_all) e left join (select drink_name, branch_name as shared from bev_branch_all) f on (e.drink_name=f.drink_name)) where e.branch_name='Branch4' and shared = 'Branch7')g;
+---------------------+
|     drink_name      |
+---------------------+
| Cold_Coffee         |
| Cold_Espresso       |
| Cold_LATTE          |
| Cold_Lite           |
| Cold_MOCHA          |
| Cold_cappuccino     |
| Double_Coffee       |
| Double_Espresso     |
| Double_Lite         |
| Double_MOCHA        |
| Double_cappuccino   |
| ICY_Coffee          |
| ICY_LATTE           |
| ICY_Lite            |
| ICY_MOCHA           |
| ICY_cappuccino      |
| LARGE_Coffee        |
| LARGE_Espresso      |
| LARGE_LATTE         |
| LARGE_Lite          |
| LARGE_MOCHA         |
| LARGE_cappuccino    |
| MED_Coffee          |
| MED_Espresso        |
| MED_Lite            |
| MED_MOCHA           |
| MED_cappuccino      |
| Mild_Coffee         |
| Mild_Espresso       |
| Mild_LATTE          |
| Mild_Lite           |
| Mild_MOCHA          |
| Mild_cappuccino     |
| SMALL_Coffee        |
| SMALL_Espresso      |
| SMALL_LATTE         |
| SMALL_Lite          |
| SMALL_MOCHA         |
| SMALL_cappuccino    |
| Special_Coffee      |
| Special_Espresso    |
| Special_LATTE       |
| Special_Lite        |
| Special_MOCHA       |
| Special_cappuccino  |
| Triple_Coffee       |
| Triple_Espresso     |
| Triple_LATTE        |
| Triple_Lite         |
| Triple_MOCHA        |
| Triple_cappuccino   |
+---------------------+
[51 Rows]
```

## Problem Four
	// For Each Question in Problem Three, create a Partition and View, as well as an Index

	//Regarding List of Beverages Sold at Branches 10, 8 and 1
	//Partition
```
0: jdbc:hive2://localhost:10000> create external table p3a_outside (drink_name string) partitioned by (grouping_field string);
0: jdbc:hive2://localhost:10000> Alter table p3a_outside add if not exists partition (grouping_field = '1') partition (grouping_field = '2') partition (grouping_field = '3') partition (grouping_field= '4');
0: jdbc:hive2://localhost:10000> show partitions p3a_outside;
+-------------------+
|     partition     |
+-------------------+
| grouping_field=1  |
| grouping_field=2  |
| grouping_field=3  |
| grouping_field=4  |
+-------------------+
0: jdbc:hive2://localhost:10000> set hive.exec.dynamic.partition.mode=nonstrict;
0: jdbc:hive2://localhost:10000> insert into p3a_outside partition(grouping_field) select * from (select drink_name as dn, 1 as grouping_field from problemThree_a limit 11)p3aa;
0: jdbc:hive2://localhost:10000> insert into p3a_outside partition(grouping_field) select * from (select drink_name as dn, 2 as grouping_field from problemThree_a limit 11 offset 11)p3ab;
0: jdbc:hive2://localhost:10000> insert into p3a_outside partition(grouping_field) select * from (select drink_name as dn, 3 as grouping_field from problemThree_a limit 11 offset 22)p3ac;
0: jdbc:hive2://localhost:10000> insert into p3a_outside partition(grouping_field) select * from (select drink_name as dn, 4 as grouping_field from problemThree_a limit 11 offset 33)p3ad;
```

	//Index
```
0: jdbc:hive2://localhost:10000> CREATE INDEX counter ON TABLE projectone.problemThree_a (drink_name) as 'COMPACT' WITH DEFERRED REBUILD;
No rows affected (0.095 seconds)
0: jdbc:hive2://localhost:10000> show formatted index on problemThree_a;
+-----------------------+-----------------------+-----------------------+---------------------------------------+-----------------------+-----------------------+
|       idx_name        |       tab_name        |       col_names       |             idx_tab_name              |       idx_type        |        comment        |
+-----------------------+-----------------------+-----------------------+---------------------------------------+-----------------------+-----------------------+
| idx_name              | tab_name              | col_names             | idx_tab_name                          | idx_type              | comment               |
|                       | NULL                  | NULL                  | NULL                                  | NULL                  | NULL                  |
|                       | NULL                  | NULL                  | NULL                                  | NULL                  | NULL                  |
| counter               | problemthree_a        | drink_name            | projectone__problemthree_a_counter__  | compact               |                       |
+-----------------------+-----------------------+-----------------------+---------------------------------------+-----------------------+-----------------------+
```

	//View	
```
0: jdbc:hive2://localhost:10000> create view problemFour_p3a as select * from problemThree_a;
```


	//Regarding Common Beverages between Branches 4 and 7
	//Partition
```
0: jdbc:hive2://localhost:10000> create external table p3b_outside (drink_name string) partitioned by (grouping_field string);
0: jdbc:hive2://localhost:10000> Alter table p3b_outside add if not exists partition (grouping_field = '1') partition (grouping_field = '2') partition (grouping_field = '3');
0: jdbc:hive2://localhost:10000> show partitions p3b_outside;
+-------------------+
|     partition     |
+-------------------+
| grouping_field=1  |
| grouping_field=2  |
| grouping_field=3  |
+-------------------+
0: jdbc:hive2://localhost:10000> set hive.exec.dynamic.partition.mode=nonstrict;
0: jdbc:hive2://localhost:10000> insert into p3b_outside partition(grouping_field) select * from (select drink_name as dn, 1 as grouping_field from problemThree_a limit 17)p3ba;
0: jdbc:hive2://localhost:10000> insert into p3b_outside partition(grouping_field) select * from (select drink_name as dn, 2 as grouping_field from problemThree_a limit 17 offset 17)p3bb;
0: jdbc:hive2://localhost:10000> insert into p3b_outside partition(grouping_field) select * from (select drink_name as dn, 3 as grouping_field from problemThree_a limit 14 offset 34)p3bc;
```
	//Index
```
0: jdbc:hive2://localhost:10000> CREATE INDEX counter ON TABLE projectone.problemThree_b (drink_name) as 'COMPACT' WITH DEFERRED REBUILD;
0: jdbc:hive2://localhost:10000> show formatted index on problemThree_b;
+-----------------------+-----------------------+-----------------------+---------------------------------------+-----------------------+-----------------------+
|       idx_name        |       tab_name        |       col_names       |             idx_tab_name              |       idx_type        |        comment        |
+-----------------------+-----------------------+-----------------------+---------------------------------------+-----------------------+-----------------------+
| idx_name              | tab_name              | col_names             | idx_tab_name                          | idx_type              | comment               |
|                       | NULL                  | NULL                  | NULL                                  | NULL                  | NULL                  |
|                       | NULL                  | NULL                  | NULL                                  | NULL                  | NULL                  |
| counter               | problemthree_b        | drink_name            | projectone__problemthree_b_counter__  | compact               |                       |
+-----------------------+-----------------------+-----------------------+---------------------------------------+-----------------------+-----------------------+
```
	//View	
```
0: jdbc:hive2://localhost:10000> create view problemFour_p3b as select * from problemThree_b;
```




## Problem Five

	//Demonstrate adding a note or comment to a table
```
ALTER TABLE bev_branch_all SET TBLPROPERTIES('notes'='these are all the drinks sold at each branch');
show tblproperties bev_branch_all;
+------------------------+-----------------------------------------------+
|       prpt_name        |                  prpt_value                   |
+------------------------+-----------------------------------------------+
| COLUMN_STATS_ACCURATE  | {"BASIC_STATS":"true"}                        |
| last_modified_by       | georgefk                                      |
| last_modified_time     | 1620850826                                    |
| notes                  | these are all the drinks sold at each branch  |
| numFiles               | 1                                             |
| numRows                | 600                                           |
| rawDataSize            | 12397                                         |
| totalSize              | 12997                                         |
| transient_lastDdlTime  | 1620850826                                    |
+------------------------+-----------------------------------------------


ALTER TABLE cons_count_all SET TBLPROPERTIES('comment'='these are all the number of customers for each drink');
show tblproperties cons_count_all;
+------------------------+----------------------------------------------------+
|       prpt_name        |                     prpt_value                     |
+------------------------+----------------------------------------------------+
| COLUMN_STATS_ACCURATE  | {"BASIC_STATS":"true"}                             |
| comment                | these are all the number of customers for each drink |
| last_modified_by       | georgefk                                           |
| last_modified_time     | 1620850846                                         |
| numFiles               | 1                                                  |
| numRows                | 6000                                               |
| rawDataSize            | 99610                                              |
| totalSize              | 105610                                             |
| transient_lastDdlTime  | 1620850846                                         |
+------------------------+----------------------------------------------------+
```


## Problem Six

	// Remove Branch 5 from the Full Table Set in Problem One 
	// Recall in Problem One we create a table called p6_target	
```
0: jdbc:hive2://localhost:10000> insert overwrite table p6_target select bn, most from(select *, rank() over (order by bn asc) as row_num from(select branch_name as bn, sum(drink_counter) as most from cumulative_set cs group by cs.branch_name)abc)xyz where xyz.row_num!=5;
0: jdbc:hive2://localhost:10000> select * from p6_target;
+------------------------+------------------+
| p6_target.branch_name  | p6_target.totals |
+------------------------+------------------+
| Branch1                | 1115974          |
| Branch2                | 5099141          |
| Branch3                | 2591062          |
| Branch4                | 3940862          |
| Branch6                | 8032799          |
| Branch7                | 6566187          |
| Branch8                | 2583583          |
| Branch9                | 5208016          |
+------------------------+------------------+
```



