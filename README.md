* [Introduction](#introduction)
* [Getting Started](#getting-started)
* [API](#api)
  * [Assertions/Test Steps](#assertions)
  * [Custom Metrics](#custom-metrics)
  * [Logging](#logging)
  * [Read from CSV](#read-from-csv)
  * [Browser Performance Metrics](#browser-performance-metrics)

# Introduction

This library allows you to write Selenium Java tests that integrate with the Testable platform. When 

# Getting Started

When developing locally include the following artifact in your build:

```xml
<dependency>
  <groupId>io.testable</groupId>
  <artifactId>testable-java</artifactId>
  <version>0.0.1</version>
</dependency>
```


A simple example test would look as follows:

```java
import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import io.testable.java.TestableJava;

public class TestableExample {

    public static void main(String[] args) throws Exception {
      URL url = new URL("http://sample.testable.io/stocks/IBM");
      HttpURLConnection con = (HttpURLConnection) url.openConnection();
      con.setRequestMethod("GET");
      
      Reader streamReader = new InputStreamReader(con.getInputStream());
      BufferedReader in = new BufferedReader(streamReader);
      String inputLine;
      StringBuilder content = new StringBuilder();
      while ((inputLine = in.readLine()) != null) {
        content.append(inputLine);
      }
      
      in.close();
      
      con.disconnect();
      System.out.println(content);
    }
}
```

.......
# API

## Assertions/Test Steps

Capture assertions or test steps as part of the test results including test step description, 
duration, and any errors that occurred.

In the results you can see these within the Assertions widget. For example:

```java
TestableTest test = TestableJava.startTest("Google Related");
test.startStep("Open google home page");


test.finishSuccessfulStep(); // or test.finishFailedStep("My error message");
test.runStep("Open google news using Runnable", new Runnable() {
    public void run() {
        driver.get("https://news.google.com"); // any exception here gets logged as a test step failure
    }
});
test.finish();
```

## Custom Metrics

Capture a custom counter, timing, or histogram metric. When run on Testable
it is reported and aggregated into the test results. When run locally
it will simply output the metric details to the console.

See https://testable.io/documentation/scripts/custom-metrics.html for
more details.

Counter Example:

```java
TestableJava.reportMetric(TestableMetric.newCounterBuilder()
        .withName("My Request Counter")
        .withVal(1)
        .withUnits("requests")
        .build());
```

Add 1 to the "My Request Counter" metric.

Timing Example:

```java
long start = System.currentTimeMillis();

long loadTime = System.currentTimeMillis() - start;
TestableJava.reportMetric(TestableMetric.newTimingBuilder()
            .withName("Page Load Time")
            .withVal(loadTime)
            .withUnits("ms")
            .build());
```

Capture how long it takes to open https://www.google.com and capture that
as the "Page Load Time" metric.

Histogram Example:

```java
String status = "MyStatus";
TestableJava.reportMetric(TestableMetric.newHistogramBuilder()
            .withName("Status Histogram")
            .withKey(status)
            .withVal(1)
            .build());
```

Add 1 to the "Status Histogram" metric.



## Logging

Log a message or exception into the test results at the specified level.
When run locally it simply outputs to the console.

Trace logging will only be output during a smoke test.
Fatal logging will cause the test run to stop immediately.

Example:

```java
TestableJava.log(TestableLog.Level.Trace, "detailed stuff for smoke test only");
TestableJava.log(TestableLog.Level.Debug, "my debug message");
TestableJava.log(TestableLog.Level.Info, "some info");
TestableJava.log(TestableLog.Level.Error, new RuntimeException("An error occurred"));
TestableJava.log(TestableLog.Level.Fatal, new RuntimeException("Something bad happened stop everything!"));
```

## Read from CSV

Read from a CSV file that has been uploaded to your scenario. When run locally
this will load the CSV from the classpath or current working directory.
It is assumed your CSV has a header row with column names.

**Get row by index**: Return a row by index. The first row after the header row
is considered row 0.

```java
TestableCSVReader reader = TestableJava.readCsv("credentials.csv");
CSVRecord record = reader.get(2);
System.out.println(record.get("username"));
```

**Get random row**: Returns a random row from the CSV.

```java
TestableCSVReader reader = TestableJava.readCsv("credentials.csv");
CSVRecord record = reader.random();
System.out.println(record.get("username"));
```

**Get the next row**: Return the next row in the CSV **using a global iterator**.
This means that the rows in the CSV will be evenly distributed across all
virtual users that are part of your test execution.

```java
TestableCSVReader reader = TestableJava.readCsv("credentials.csv");
CSVRecord record = reader.next();
System.out.println(record.get("username"));
```

## Browser Performance Metrics

Testable provides an API for extracting a bunch of useful browser performance
metrics into your test results including: page load time, speed index,
page requests, page weight, time to first byte, time to first paint,
time to first contentful paint, and time to interactive. See our
[metrics glossary](https://docs.testable.io/guides/metrics.html) for a
precise definition of each metric.

The method also returns a Map of all metrics captured in addition to
automatically reporting it back into the test results.

```java
TestableJava.collectPerformanceMetrics(driver);
```
