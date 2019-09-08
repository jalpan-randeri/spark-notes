## Hive

Spark Support Hive SerDe libraries to read underlying data.
Spark uses Hadoop File Input Format API to read data.

## Input Format API
Consumer of input format api need to provide implementation of following
methods. 
```
public interface InputFormat<K, V> {

  InputSplit[] getSplits(JobConf job, 
                         int numSplits) throws IOException;

  RecordReader<K, V> getRecordReader(InputSplit split,
                                     JobConf job, 
                                     Reporter reporter) throws IOException;
}
```

### Get Splits API
This api informs spark how to logically split a file into small parts. 
How many bytes need to skip before reading and how many bytes needs to read.
By default Hadoop File Format Implementation of Input Format api splits file,
into chunks of 1 MB. 

Hadoop File Input Format implementation performs this by File System Listing. 
It performs file system listing and then calculating minimum split size. Then 
it calculates logical file split for every file.

### Record Read API
This api provides support for reading underlying file and return record to spark.
It respects the boundary for each file. This api provides a stream of records to spark, 
This allows spark to read sequence of records in asynchronous method.

Main responsibility of the RecordReader to respect record boundaries while 
processing the logical split to present a record-oriented view to the individual task.
