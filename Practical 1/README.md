# DSBDA Practical – MapReduce Log File Processing (Java + Hadoop)

## 🔧 Objective

Design a distributed application using **MapReduce (Java)** to process a **log file** of a system and list users who have logged in for the **maximum period**.

---

## 🖥️ Prerequisites

* Cloudera VM (pseudo-distributed Hadoop setup)
* Eclipse IDE
* Basic knowledge of Java & Hadoop
* Sample system log file (can be downloaded from [mapreduceprogram.blogspot.com](https://mapreduceprogram.blogspot.com/))

---

## 📁 Step-by-Step Instructions

### 1. Open Eclipse and Create Java Project

* Go to `File > New > Java Project`
* **Project Name:** `Process` *(recommended to keep name same for consistency)*

### 2. Create Package and Class

* Right-click on `src > New > Package` → Name it `Process`
* Right-click on the package → `New > Class` → Name it `Process`
* Paste the following code into `Process.java`:

```java
package Process;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class Process {

  public static class IPMapper extends Mapper<Object, Text, Text, IntWritable> {
    private final static IntWritable one = new IntWritable(1);
    private Text ip = new Text();

    public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      if (itr.hasMoreTokens()) {
        ip.set(itr.nextToken());
        context.write(ip, one);
      }
    }
  }

  public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values, Context context)
        throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "IP address count");
    job.setJarByClass(Process.class);
    job.setMapperClass(IPMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```

---

### 3. Add Required External JARs

1. Right-click `Project > Build Path > Configure Build Path > File System `
2. Add:

   * `/usr/lib/hadoop/hadoop-common-2.6.0-cdh5.13.0.jar`
   * `/usr/lib/hadoop-0.20-mapreduce/hadoop-core-2.6.0-mr1-cdh5.13.0.jar`

✅ Errors in your code should now be resolved.

---

### 4. Prepare the Log File

* Download or copy sample log from:
  🔗 [https://mapreduceprogram.blogspot.com/](https://mapreduceprogram.blogspot.com/)
* Save the file as: `Process.txt` on the Cloudera Desktop

---

### 5. Upload File to HDFS

Open **Cloudera Terminal** and run:

```bash
hadoop fs -put Desktop/Process.txt Process.txt
```

---

### 6. Export Project as JAR

* Right-click `Process.java` → `Export > Java > JAR file`
* Name it: `Process.jar`
* Save it to a known Cloudera directory (e.g., `/home/cloudera`)

---

### 7. Run MapReduce Job

Run in Cloudera terminal:

```bash
hadoop jar Process.jar Process.Process Process.txt dir16
```

---

### 8. View Output

After successful execution:

```bash
hadoop fs -cat dir16/part-r-00000
```

✅ You will now see the **list of users (IP addresses)** and the **number of times** they logged into the system. The user with the highest count stayed the longest.

---

## 🏁 Conclusion

You have successfully:

* Designed and executed a **MapReduce** job in **pseudo-distributed Hadoop**
* Processed a **log file**
* Identified users with **maximum login time**

---
✅ DONE
