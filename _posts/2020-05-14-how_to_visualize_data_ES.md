---
layout: post
title: How To Use Elasticsearch to Visualize Data? 
date: 2020-05-14 11:12:00-0400
description:
tags: elasticsearch data-visualization
categories: sample-posts
related_posts: false
---

Elasticsearch mappings allow storing your data in formats which can be easily translated into meaningful visualizations capturing multiple complex relationships in your data. In this tutorial, we'll show how to create data visualizations with Kibana, a part of ELK stack that makes it easy to search, view, and interact with data stored in Elasticsearch indices. We'll walk you through basic data visualization types including line charts, area charts, pie charts, and time series after which you'll be ready to design a custom visualization of any complexity. 

## **Get Data**

For this tutorial, we'll be using data supplied by [Metricbeat](https://www.elastic.co/products/beats/metricbeat), a light shipper that can be installed  on your server to periodically collect metrics from the OS and various services running on the server. Metricbeat takes the metrics and sends them to the output you specify - in our case, a Qbox-hosted Elasticsearch cluster.  Metricbeat currently supports system statistics and a wide variety of metrics from the popular software like MongoDB, Apache, Redis, MySQL, and many more. Data from these services includes diverse fields and parameters which makes Metricbeat a great tool for illustrating the power of Kibana data visualization. 

To start using Metricbeat data, you'll need the following software installed and configured:

- Elasticsearch for storage and indexing of data. For this tutorial, we're using a [Qbox-hosted Elasticsearch cluster.](https://qbox.io/blog/provisioning-a-qbox-elasticsearch-cluster)
- Kibana. A Qbox-hosted Elasticsearch ships with Kibana. When provisioning your cluster, just specify that you want to install it. 

To install Metricbeat with a deb package on the Linux system,  run the following commands:

```sh
curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-6.2.3-amd64.deb
sudo dpkg -i metricbeat-6.2.3-amd64.deb
```

Before using Metricbeat, we'll need to configure the shipper in the `metricbeat.yml` file usually located in the`/etc/metricbeat/` folder on Linux distributions. In the configuration file, we at least need to specify Kibana's and Elasticsearch's hosts we want to send our data to and attach modules we want Metricbeat to collect data from. See [Metricbeat documentation](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-configuration.html) for more details about configuration. 

Once all configuration edits are made, we can start the Metricbeat service with the following command:

```sh
sudo service metricbeat start
```

Metricbeat will start periodically collecting and shipping data about your system and services to Elasticsearch. Now, we can use Kibana to display this data but before being able to do so, we have to add a `metricbeat-` index pattern to our Kibana management panel. 

After this is done, you'll see the following index template with a list of fields sent by Metricbeat to your Qbox-hosted Elasticsearch instance. 

![Image1_metricbeat_index.png]()

That's it! We can now visualize our Metricbeat data using rich Kibana's visualization features. 

## Kibana's Visualize Module

Kibana visualizations use Elasticsearch documents and their respective fields as inputs and Elasticsearch aggregations and metrics as utility functions to extract and process that data. Kibana supports numerous visualization types, including time series with Timelion and Visual Builder, various basic charts (e.g area charts, heat maps, horizontal bar charts, line charts and pie charts), tables, gauges, coordinate and region maps and tag clouds to name a few. All available visualization types can be accessed under **Visualize** section of the Kibana dashboard.

The steps needed to create a visualization might differ depending on the visualization you want to produce, however, you should know basic definitions, metrics, and aggregations applied in most visualization types.

The first step to create a standard Kibana visualization like a line- or bar chart is to select a metric which defines a value axis (usually a Y-axis). Kibana supports a number of Elasticsearch aggregations to represent your data in this axis:

- **Count**

  Returns a count of the elements in the selected index pattern 

- **Average**

  Returns the average of a numeric field selected in the drop-down.

- **Min**

  Returns the minimum value of a numeric field selected in the drop-down

- **Max**

  Returns the maximum value of a numeric field selected in the drop-down

- **Unique Count**

  This cardinality aggregation returns the number of unique values of a field

- **Standard Deviation**

  Returns the standard deviation of data in a numeric field. 

These are  just several parent aggregations available. For more metrics and aggregations consult [Kibana](https://www.elastic.co/guide/en/kibana/current/xy-chart.html) documentation.

Kibana also supports the bucket aggregations which determine what information to retrieve from your Elasticsearch index. This information is usually displayed above the X-axis of your chart which is normally the *buckets* axis. The X-axis supports the following aggregations for which you may find additional information in the main [Elasticsearch documentation](https://www.elastic.co/guide/en/kibana/current/xy-chart.html):

- **Date Histogram**

  This aggregation is built from a numeric field and is organized by date. It allows specifying intervals for your historical data or design custom intervals.

- **Histogram**

  Builds a standard histogram from a numeric field. You need to specify an integer interval for this field. 

- **Range**

  Range aggregation allows specifying ranges of values for a numeric field.

- **IPv4 Range**

  With this aggregation, you can specify ranges of IPv4 addresses. 

- **Terms**

  With a terms aggregation, you can specify the top or bottom *n* elements of a field to display ordered by count or any other custom metric.

- **Filters**

  Kibana supports filters to specify rules for querying your Elasticsearch documents.

After we've specified aggregations for the X-axis, we can add sub-aggregations that refine the visualization. With this option, you can create charts with multiple buckets and aggregations of data. After all metrics and aggregations are defined,  we can also customize the chart using custom labels, colors, and other useful features. 

# Time Series with Timelion

[Timelion](https://www.elastic.co/blog/timelion-timeline) is the time series composer for Kibana that enables to combine totally independent data sources in a single visualization using chainable functions. Timelion uses a simple expression language that allows retrieving time series data, making complex calculations and chaining additional visualizations. 

In the example below, we are combining a time series of the average CPU time spent in kernel space (`system.cpu.system.pct`) during the specified period of time with the same metric taken with a 20-minute offset.  The expression below chains two `.es()` functions which define the ES index to retrieve data from, a time field to use for our time series, a field to apply our metric to (`system.cpu.system.pct`) and an offset value. Chaining these two functions allows visualizing dynamics of the CPU usage over time.

```sh
.es(index=metricbeat-*, timefield='@timestamp', metric='avg:system.cpu.system.pct'), .es(offset=-20m,index=metricbeat-*, timefield='@timestamp', metric='avg:system.cpu.system.pct')
```

![Image2_Timelion-Time-Series.png]()

You can experiment with Timelion by doing similar comparisons for the percentage of the CPU time spent in user space, for low-priority processes, being idle and using numerous other metrics shipped by your Metricbeat instance. 

## **Visual Builder**

A powerful alternative to Timelion for building time series visualization is the [Visual Builder](https://www.elastic.co/guide/en/kibana/current/time-series-visual-builder.html) recently added to Kibana as a native module. Similarly to Timelion, Time Series Visual Builder enables to combine multiple aggregations and pipeline them to display complex data in a meaningful way. However, with Visual Builder, you can use simple UI to define metrics and aggregations instead of chaining functions manually as in Timelion. 

In the example below, we combine six time series that display the CPU usage in various spaces including user space, kernel space, CPU time spent on low-priority processes, time spent on handling hardware and software interrupts, and percentage of time spent in wait (on disc). To produce time series for each parameter, we define a metric that includes an aggregation type (e.g average) and the field name (e.g `system.cpu.user.pct`) for that parameter. For each metric, we can also specify a label to make our time series visualization more readable. With the Visual Builder, we can even create annotations that will attach additional data sources like system messages emitted at specific intervals to our Time Series visualization. It's as easy as that!

![Image3_Visualbuilder-time-series.png]()

In addition to time series visualizations, Visual Builder supports other visualization types such as Metric, Top N, Gauge and Markdown which automatically convert our data into their respective visualization formats. For example, in the image below we've created a Top N simple visualization that displays top spaces where our CPU is used.

![Image4_visual-builder-top-n.png]()

In sum, Visual Builder is a great sandbox for experimentation with your data with which you can produce wonderful time series, gauges, metrics, and Top N lists. 

### Creating a Line Chart

A line chart is a basic type of chart that represents data as a series of data points connected by straight line segments. In the image below, you can see a line chart of the system load over the past 15 minutes. To create this chart, in the Y-axis, we used an average aggregation for the `system.load.1` field that calculates the system load average. You are not limited to the average aggregation, however, since Kibana supports a number of other Elasticsearch aggregations including median, standard deviation, min, max, and percentiles to name a few. You can play with them to figure out whether they work fine with the data you want to visualize. 

After defining the metric for the Y-axis, we specify parameters for our X-axis. In this example, we use data histogram for aggregation and the default `@timestamp` field to take timestamps  from. After our parameters are entered, we can click on the 'play' button which will generate the line chart visualization with all axes and labels automatically added. That's it! Now we can save our line chart to the dashboard by clicking 'Save' link in the top menu. 

![Image5_System-load-line-chart.png]()



### **Pie Chart**

A pie chart or a circle chart is a visualization type which is divided into different slices to illustrate numerical proportion. 

In this example, we'll be using a split slice chart to visualize the CPU time usage by the processes running on our system. The first step to create our pie chart is to select a metric that defines how a slice's size is determined. Kibana pie chart visualizations provide with three options for this metric: *count, sum, and unique count aggregations* discussed above. For our goal, we are interested in the *sum* aggregation for the`system.process.cpu.total.pct` field that describes the percentage of CPU time spent by the process since the last update. Once the metric is specified, we can also create a custom label for this value (e.g Total CPU usage by the process).

The next step is to define our *buckets*. We've decided to use split slices chart which is a convenient way to visualize how parts make up the meaningful whole. For our buckets, we need to select a *Terms* aggregation that specifies the top or bottom *n* elements of a given field to display ordered by some metric. In our case, we'll display 7 top processes running on our system ( `system.process.name` field)  in terms of CPU time usage. The metric used to display our *Terms* aggregation will be the sum of the total CPU time usage by an individual process defined above. That's it! Now, as always, let's click play to see our pie chart. 

![Image6_pie-chart-processes.png]()

As you see, Kibana automatically produced seven slices for the top seven processes in terms of CPU time usage. The size of each slice represents this value which is the highest for `supergiant` and `chrome` processes in our case. We can now save the created pie chart to the dashboard visualizations for later access. 

**Notice**: when creating pie charts, remember that pie slices should sum up to a meaningful whole. In our case, this rule is followed -  the whole is a sum of the CPU time usage by top seven processes running our system. 

### Making an Area Chart

Area charts are just like line charts in that they represent the change in one or more quantities over time. The difference is, however, that area charts have the area between the X-axis and the line filled with color or shading.

In Kibana, the area chart's Y-axis is the *metrics* axis. It supports a number of aggregation types like count, average, sum, min, max, percentile, and more. In the example below,  we drew an area chart that displays the percentage of CPU time usage by individual processes running on our system. 

![Image7_Area-chart-processes]()



For the Y-Axis the metrics defined is the average for the field `system.process.cpu.total.pct` which can be higher than 100 percent if your computer has a multi-core processor. The next step is to specify the X-Axis metric and create individual buckets. In the X-Axis, we are using Date Histogram aggregation for the `@timestamp` field  with the auto interval which defaults to 30 seconds. As an option, you can also select intervals ranging from milliseconds to years or even design your own interval. 

Once we've specified the Y-axis and X-axis aggregations, we can now define sub-aggregations to refine the visualization. For this example, we've selected *split series*  which is a convenient way to represent the quantity change over time. Now, in order to represent the individual process, we define the 'Terms' sub-aggregation on the field `system.process.name` ordered by the previously defined CPU usage metric. In this bucket, we can also select the number of processes to display. That's it!  Now we can save our area chart visualization of the CPU usage by an individual process to the dashboard.

### Conclusion 

Elasticsearch powered by Kibana makes data visualizations an extremely fun thing to do. Recent Kibana versions ship with a number of convenient templates and visualization types as well as a native Visualization Builder. With these features, you can construct anything ranging from a line chart to tag clouds leveraging Elasticsearch's rich aggregation types and metrics. In the next tutorials, we will discuss more visualization options in Kibana such as coordinate and region maps and tag clouds. 

