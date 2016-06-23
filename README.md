# Oracle-to-Redshift-Data-Loader
    Ground to cloud data integration tool.
    Used for ad-hoc query results load from Oracle to Amazon-Redshift.
    Works from Windows CLI (command line).

Features:
 - Loads Oracle table (or query) data to Amazon-Redshift.
 - No need to create CSV extracts and S3 uploads before load to Redshift.
 - Data stream is compressed while loaded to S3 (and then to Redshift).
 - Works from your OS Windows desktop (command line).
 - It's executable (Oracle_To_Redshift_Loader.exe)  - no need for Python install.
 - It's 64 bit - it will work on any vanilla DOS for 64-bit Windows.
 - AWS Access Keys are not passed as arguments.  
 - You can modify default Python [extractor](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/extractor.py) and [loader](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/loader.py) code.
 - Written using Python/boto/psycopg2.


##Other scripts
  - [Oracle -> S3](https://github.com/alexbuz/Oracle_To_S3_Data_Uploader/blob/master/README.md) data loader
  - [PostgreSQL -> Redshift](https://github.com/alexbuz/PostgreSQL_To_Redshift_Loader/blob/master/README.md) data loader
  - [MySQL -> Redshift](https://github.com/alexbuz/MySQL_To_Redshift_Loader/blob/master/README.md) data loader
  - [CSV -> Redshift](https://github.com/alexbuz/CSV_Loader_For_Redshift/blob/master/README.md) data loader

##Purpose

- Stream/pipe/load Oracle table data to Amazon-Redshift.

## How it works
- Tool connects to source Oracle DB and opens data pipe for reading.
- Data stream is compressed and pumped to S3 using multipart upload.
- Optional upload to Reduced Redundancy storage (not RR by default).
- Optional "make it public" after upload (private by default).
- If S3 bucket doesn't exists it will be created.
- You can control the region where new bucket is created.
- Streamed data can be tee'd (dumped on disk) during load.
- If not set, S3 Key defaulted to input query file name.
- Data is loaded to Redshift from S3 using COPY command
- Target Redshift table has to exist
- It's a Python/boto/psycopg2 script
	* Boto S3 docs: http://boto.cloudhackers.com/en/latest/ref/s3.html
	* psycopg2 docs: http://initd.org/psycopg/docs/
- Executable is created using [pyInstaller] (http://www.pyinstaller.org/)

##Audience

Database/ETL developers, Data Integrators, Data Engineers, Business Analysts, AWS Developers, SysOps

##Designated Environment
Pre-Prod (UAT/QA/DEV)

##Usage

```
c:\Python35-32\PROJECTS\Ora2redshift>dist\oracle_to_Redshift_loader.exe
#############################################################################
#Oracle-to-Redshift Data Loader (v1.2, beta, 04/05/2016 15:11:53) [64bit]
#Copyright (c): 2016 Alex Buzunov, All rights reserved.
#Agreement: Use this tool at your own risk. Author is not liable for any damages
#           or losses related to the use of this software.
################################################################################
Usage:
  set AWS_ACCESS_KEY_ID=<you access key>
  set AWS_SECRET_ACCESS_KEY=<you secret key>

  set ORACLE_LOGIN=tiger/scott@orcl
  set ORACLE_CLIENT_HOME=C:\app\oracle12\product\12.1.0\dbhome_1
  
  set NLS_DATE_FORMAT="MM/DD/YYYY HH12:MI:SS"
  set NLS_TIMESTAMP_FORMAT="MM/DD/YYYY HH12:MI:SS.FF"
  set NLS_TIMESTAMP_TZ_FORMAT="MM/DD/YYYY HH12:MI:SS.FF TZH:TZM"

  set REDSHIFT_CONNECT_STRING="dbname='***' port='5439' user='***' password='***' host='mycluster.***.redshift.amazonaws.com'"


  oracle_to_redshift_loader.exe [<ora_query_file>] [<ora_col_delim>] [<ora_add_header>]
                            [<s3_bucket_name>] [<s3_key_name>] [<s3_use_rr>] [<s3_public>]

        --ora_query_file -- SQL query to execure in source Oracle db.
        --ora_col_delim  -- CSV column delimiter for downstream(,).
        --ora_quote     -- Enclose values in quotes (")
        --ora_add_header -- Add header line to CSV file (False).
        --ora_lame_duck  -- Limit rows for trial upload (1000).

        --create_data_dump -- Use it if you want to persist streamed data on your filesystem.

        --s3_bucket_name -- S3 bucket name (always set it).
        --s3_location    -- New bucket location name (us-west-2)
                                Set it if you are creating new bucket
        --s3_key_name    -- CSV file name (to store query results on S3).
                if <s3_key_name> is not specified, the oracle query filename (ora_query_file) will be used.
        --s3_use_rr -- Use reduced redundancy storage (False).
        --s3_write_chunk_size -- Chunk size for multipart upoad to S3 (10<<21, ~20MB).
        --s3_public -- Make uploaded file public (False).

        --red_to_table  -- Target Amazon-Redshit table name.
        --red_col_delim  -- CSV column delimiter for upstream(,).
        --red_quote     -- Set it if input values are quoted.
        --red_timeformat -- Timestamp format for Redshift ('MM/DD/YYYY HH12:MI:SS').
        --red_ignoreheader -- skip header in input stream

        Oracle data uploaded to S3 is always compressed (gzip).

        Boto S3 docs: http://boto.cloudhackers.com/en/latest/ref/s3.html
        psycopg2 docs: http://initd.org/psycopg/docs/

```
#Example


###Environment variables
Set the following environment variables (for all tests):

```
set AWS_ACCESS_KEY_ID=<you access key>
set AWS_SECRET_ACCESS_KEY=<you secret key>

set ORACLE_LOGIN=tiger/scott@orcl
set ORACLE_CLIENT_HOME=C:\\app\\oracle12\\product\\12.1.0\\dbhome_1

  set NLS_DATE_FORMAT="MM/DD/YYYY HH12:MI:SS"
  set NLS_TIMESTAMP_FORMAT="MM/DD/YYYY HH12:MI:SS.FF"
  set NLS_TIMESTAMP_TZ_FORMAT="MM/DD/YYYY HH12:MI:SS.FF TZH:TZM"
  
set REDSHIFT_CONNECT_STRING="dbname='***' port='5439' user='***' password='***' host='mycluster.***.redshift.amazonaws.com'"  
```

### Test load with data dump.
Oracle table `crime_test` contains data from data.gov [Crime](https://catalog.data.gov/dataset/crime) dataset.
In this example complete table `crime_test` get's uploaded to Aamzon-S3 as compressed CSV file.

Contents of the file *table_query.sql*:

```
SELECT * FROM crime_test;

```
Also temporary dump file is created for analysis (by default there are no files created)
Use `-s, --create_data_dump` to dump streamed data.

If target bucket does not exists it will be created in user controlled region.
Use argument `-t, --s3_location` to set target region name

Contents of the file *test.bat*:
```
dist-64bit\oracle_to_redshift_loader.exe ^
-q table_query.sql ^
-d "," ^
-b test_bucket ^
-k oracle_table_export ^
-r ^
-o crime_test ^
-m "DD/MM/YYYY HH12:MI:SS" ^
-s
```
Executing `test.bat`:

```
c:\Python35-32\PROJECTS\Ora2redshift>dist-64bit\oracle_to_redshift_loader.exe -q table_query.sql -d "," -b test_bucket -k oracle_table_export -r -o crime_test -m "DD/MM/YYYY HH12:MI:SS" -s
Uploading results of "table_query.sql" to existing bucket "test_bucket"
Started reading from Oracle (1.25 sec).
Dumping data to: c:\Python35-32\PROJECTS\Ora2redshift\data_dump\table_query\test_bucket\oracle_table_export.20160408_203221.gz
1 chunk 10.0 MB [11.36 sec]
2 chunk 10.0 MB [11.08 sec]
3 chunk 10.0 MB [11.14 sec]
4 chunk 10.0 MB [11.12 sec]
5 chunk 877.66 MB [0.96 sec]
Size: Uncompressed: 40.86 MB
Size: Compressed  : 8.95 MB
Elapsed: Oracle+S3    :69.12 sec.
Elapsed: S3->Redshift :3.68 sec.
--------------------------------
Total elapsed: 72.81 sec.


```

![test](https://raw.githubusercontent.com/alexbuz/Oracle-To-Redshift-Data-Loader/master/test/ora2redshift.png)

### Modifying default Oracle spooler.
You can modify default Oracle spooler fuctionality.

Open file [include\extractor.py](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/extractor.py) and modify SQL*Plus config to your liking (line 24).

```
	q="""
	set heading off line 32767 echo off feedback off  feed off pagesize 0 serveroutput off show off 
	set define off head off serveroutput off arraysize 5000
	SET LONG 50000	
	SET VERIFY OFF
	%s
	SELECT '%s'||%s||'%s' str FROM (%s) %s;
	exit;
	...
```



### Modifying default Redshift COPY command.
You can modify default Redshift COPY command this script is using.

Open file [include\loader.py](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/loader.py) and modify `sql` variable on line 24.

```
	sql="""
COPY %s FROM '%s' 
	CREDENTIALS 'aws_access_key_id=%s;aws_secret_access_key=%s' 
	DELIMITER '%s' 
	FORMAT CSV %s 
	GZIP 
	%s 
	%s; 
	COMMIT;
	...
```


###Download
* `git clone https://github.com/alexbuz/Oracle-to-Redshift-Data-Loader`
* [Master Release](https://github.com/alexbuz/Oracle-to-Redshift-Data-Loader/archive/master.zip) -- `oracle_to_redshift_loader 1.2`




#
#
#
#
#   
#FAQ
#  
#### Can it load Oracle table data to Amazon Redshift Database?
Yes, it is the main purpose of this tool.
Create query.sql file in the following format:
```
SELECT * FROM SOURCE_TABLE_NAME;
```
 - here `SOURCE_TABLE_NAME` is Oracle a table you want to copy over to Redshift.
 - Target table has to exist.
 
#### I do not want to copy the whole table. Can I just copy one Oracle table patition to Amazon Redshift Database?
Yes, use standard partition syntax.
Create query.sql file in the following format:
```
SELECT * FROM SOURCE_TABLE_NAME PARTITION(PART_NAME_1);
```
 - here `SOURCE_TABLE_NAME` is Oracle a table you want to copy over to Redshift.
 - here `PART_NAME_1` is source table partition.
 - Target table has to exist.

#### Can I stage data in any S3 Region?
No, the Amazon S3 staging folder must be in the same region as the Redshift cluster.

#### I'm joining tables in Oracle and need to copy results to Amazon Redshift database. Is it possible?
Yes, create query.sql file in the following format:
```
SELECT t1.* 
  FROM table1 t1, table2 t2 
 WHERE t1.id=t2.id;
```
 - Target table has to exist.

 
#### Can developers integrate `Oracle-to-Redshift-Data-Loader` into their ETL pipelines?
Yes. Assuming they are doing it on OS Windows.

#### How fast is data load using `Oracle-to-Redshift-Data-Loader`?
As fast as any implementation of multi-part load using Python and boto.

####How do I increase load speed?
The slowest part is S3 upload. So network speed and distance from source Oracle server and targe AWS region are important.
You can run multiple instances of this tool configured to copy individual partitions of the same table.

#### What are the other ways to move large amounts of data from Oracle to Redshift?
You can write a sqoop script that can be scheduled with Data Pipeline.

#### Can I extract data from RDS Oracle to Amazon Redshift using this tool?
Yes, but whatch where you invoke it. If you execute it outside of AWS you may get data charges.
You should spawn OS Windows EC2 instance in the same Availability Zone with your DRS Oracle and /or Redshift cluster is residing.
Login to new EC2 instance usig Remote Desktop, download `Oracle-To-Redshift-Data-Loader` and run it in CLI window.


#### Are you accessing any Oracle dictionary views or system objects?
No. I only use DBMS_SQL.PARSE to describe query you provided in input query file.
Check Python method `get_query_columns` in [extractor](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/extractor.py) module. 

#### Does it create temporary data file?
No

#### Can I log transfered data for analysis?
Yes, Use `-s, --create_data_dump` to dump streamed data.

#### Explain first step of data transfer?
The query file you provided is used to select data form target Oracle server.
Stream is compressed before load to S3.

#### Explain second step of data transfer?
Compressed data is getting uploaded to S3 using multipart upload protocol.

#### Explain third step of data load. How data is loaded to Amazon Redshift?
You Redshift cluster has to be open to the world (accessible via port 5439 from internet).
It uses PostgreSQL COPY command to load file located on S3 into Redshift table.

#### What technology was used to create this tool
I used SQL*Plus, Python, Boto to write it.
Boto is used to upload file to S3. 
SQL*Plus is used to spool data to compressor pipe.
psycopg2 is used to establish ODBC connection with Redshift clusted and execute `COPY` command.

#### Why don't you use ODBC driver for Redshift to insert data?
From my experience it's much slower that COPY command.
It's 10x faster to upload CSV file to Amazon-S3 first and then run COPY command.
You can still use ODBC for last step.
If you are a Java shop, take a look at Progress [JDBC Driver](https://www.progress.com/blogs/booyah-amazon-redshift-challenge-loaded-in-8-min-oow14).
They claim it can load 1 mil records in 6 min.


#### What would be my Oracle-to-AWS migration strategy?
 - Size the database
 - Network
 - Version of Oracle
 - Oracle clinet (SQL*Plus) availability
 - Are you doing it in one step or multiple iterations?
 
#### Was there an AWS white paper on Oracle to AWS migration strategies?
Yes, [here](https://d0.awsstatic.com/whitepapers/strategies-for-migrating-oracle-database-to-aws.pdf) it is.

#### Do you use psql to execute COPY command against Redshift?
No. I use `psycopg2` python module (ODBC).

#### Why are you uploading extracted data to S3? whould it be easier to just execute COPY command for local spool file?
As of now you cannot load from local file. You can use COPY command with Amazon Redshift, but only with files located on S3.
If you are loading CSV file from Windows command line - take a look at [CSV_Loader_For_Redshift](https://github.com/alexbuz/CSV_Loader_For_Redshift)

#### Can I modify spooler code?
Yes. Spooler code is in [include\extractor.py](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/extractor.py).

#### Can I modify default psql COPY command?
Yes. Edit [include\loader.py](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/loader.py) and add/remove COPY command options

Other options you may use:

    COMPUPDATE OFF
    EMPTYASNULL
    ACCEPTANYDATE
    ACCEPTINVCHARS AS '^'
    GZIP
    TRUNCATECOLUMNS
    FILLRECORD
    DELIMITER '$DELIM'
    REMOVEQUOTES
    STATUPDATE ON
    MAXERROR AS $MaxERROR
    
#### Can I map Redshift table columns in COPY command?
Yes. You can modify default COPY command this tool is using and map target table column.
Check [Amazon-Redshift](http://docs.aws.amazon.com/redshift/latest/dg/copy-parameters-column-mapping.html) documentation for details.

#### I'm experiencing errors in Redshift. How can I debug?
you can query stl_load_errors table for loader errors.
```
avrocluster=# select * from stl_load_errors order by starttime desc;
```
Also, you can include print statements into [include\loader.py](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/loader.py). script to see what command is actually executed.

#### Does it delete file from S3 after upload?
No

#### Does it create target Redshift table?
By default no, but using [include\loader.py](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/loader.py) you can extend default functionality and code in target table creation.

#### Can I execute multiple loaders in parallel?
Yes. Use Windows CLI Powershell to schedule multiple loaders in parallel.

Can I delete/truncate Oracle source after load?
No, but you can code it into [include\extractor.py](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/extractor.py)

#### Where are the sources?
Sources are [here](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/tree/master/sources).
Extractor code: [include\extractor.py](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/extractor.py)
Loader code: [include\loader.py](https://github.com/alexbuz/Oracle-To-Redshift-Data-Loader/blob/master/dist-64bit/include/loader.py)

#### Can you modify functionality and add features?
Yes, please, ask me for new features.

#### What other AWS tools you've created?
- [Oracle_To_S3_Data_Uploader] (https://github.com/alexbuz/Oracle_To_S3_Data_Uploader) - Stream Oracle data to Amazon- S3.
- [CSV_Loader_For_Redshift] (https://github.com/alexbuz/CSV_Loader_For_Redshift/blob/master/README.md) - Append CSV data to Amazon-Redshift from Windows.
- [S3_Sanity_Check] (https://github.com/alexbuz/S3_Sanity_Check/blob/master/README.md) - let's you `ping` Amazon-S3 bucket to see if it's publicly readable.
- [EC2_Metrics_Plotter](https://github.com/alexbuz/EC2_Metrics_Plotter/blob/master/README.md) - plots any CloudWatch EC2 instance  metric stats.
- [S3_File_Uploader](https://github.com/alexbuz/S3_File_Uploader/blob/master/README.md) - uploads file from Windows to S3.

#### Do you have any AWS Certifications?
Yes, [AWS Certified Developer (Associate)](https://raw.githubusercontent.com/alexbuz/FAQs/master/images/AWS_Ceritied_Developer_Associate.png)

#### Can you create similar/custom data tool for our business?
Yes, you can PM me here or email at `alex_buz@yahoo.com`.
I'll get back to you within hours.

###Links
 - [Employment FAQ](https://github.com/alexbuz/FAQs/blob/master/README.md)

















