<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>

<title></title>

<script type="text/javascript">
window.onload = function() {
  var imgs = document.getElementsByTagName('img'), i, img;
  for (i = 0; i < imgs.length; i++) {
    img = imgs[i];
    // center an image if it is the only element of its parent
    if (img.parentElement.childElementCount === 1)
      img.parentElement.style.textAlign = 'center';
  }
};
</script>





<style type="text/css">
body, td {
   font-family: sans-serif;
   background-color: white;
   font-size: 13px;
}

body {
  max-width: 800px;
  margin: auto;
  padding: 1em;
  line-height: 20px;
}

tt, code, pre {
   font-family: 'DejaVu Sans Mono', 'Droid Sans Mono', 'Lucida Console', Consolas, Monaco, monospace;
}

h1 {
   font-size:2.2em;
}

h2 {
   font-size:1.8em;
}

h3 {
   font-size:1.4em;
}

h4 {
   font-size:1.0em;
}

h5 {
   font-size:0.9em;
}

h6 {
   font-size:0.8em;
}

a:visited {
   color: rgb(50%, 0%, 50%);
}

pre, img {
  max-width: 100%;
}
pre {
  overflow-x: auto;
}
pre code {
   display: block; padding: 0.5em;
}

code {
  font-size: 92%;
  border: 1px solid #ccc;
}

code[class] {
  background-color: #F8F8F8;
}

table, td, th {
  border: none;
}

blockquote {
   color:#666666;
   margin:0;
   padding-left: 1em;
   border-left: 0.5em #EEE solid;
}

hr {
   height: 0px;
   border-bottom: none;
   border-top-width: thin;
   border-top-style: dotted;
   border-top-color: #999999;
}

@media print {
   * {
      background: transparent !important;
      color: black !important;
      filter:none !important;
      -ms-filter: none !important;
   }

   body {
      font-size:12pt;
      max-width:100%;
   }

   a, a:visited {
      text-decoration: underline;
   }

   hr {
      visibility: hidden;
      page-break-before: always;
   }

   pre, blockquote {
      padding-right: 1em;
      page-break-inside: avoid;
   }

   tr, img {
      page-break-inside: avoid;
   }

   img {
      max-width: 100% !important;
   }

   @page :left {
      margin: 15mm 20mm 15mm 10mm;
   }

   @page :right {
      margin: 15mm 10mm 15mm 20mm;
   }

   p, h2, h3 {
      orphans: 3; widows: 3;
   }

   h2, h3 {
      page-break-after: avoid;
   }
}
</style>



</head>

<body>
<pre><code class="r echo=TRUE">library(ggplot2)
library(scales)
library(Hmisc)
</code></pre>

<p>Loading and preprocessing the data</p>

<ol>
<li>Load the data (i.e. read.csv())</li>
</ol>

<pre><code class="r, echo=TRUE">if(!file.exists(&#39;activity.csv&#39;)){
    unzip(&#39;activity.zip&#39;)
}
activityData &lt;- read.csv(&#39;activity.csv&#39;)
</code></pre>

<ol>
<li>Process/transform the data (if necessary) into a format suitable for your analysis
What is mean total number of steps taken per day?
<code>{r, echo=TRUE}
stepsByDay &lt;- tapply(activityData$steps, activityData$date, sum, na.rm=TRUE)
</code></li>
<li>Make a histogram of the total number of steps taken each day
<code>{r, echo=TRUE}
qplot(stepsByDay, xlab=&#39;Total steps per day&#39;, ylab=&#39;Frequency using binwith 500&#39;, binwidth=500)
</code></li>
<li>Calculate and report the mean and median total number of steps taken per day
<code>{r, echo=TRUE}
stepsByDayMean &lt;- mean(stepsByDay)
stepsByDayMedian &lt;- median(stepsByDay)
</code>
What is the average daily activity pattern?
<code>{r, echo=TRUE}
averageStepsPerTimeBlock &lt;- aggregate(x=list(meanSteps=activityData$steps), by=list(interval=activityData$interval), FUN=mean, na.rm=TRUE)
</code></li>
<li><p>Make a time series plot</p>

<pre><code class="r, echo=TRUE">ggplot(data=averageStepsPerTimeBlock, aes(x=interval, y=meanSteps)) +
geom_line() +
xlab(&quot;5-minute interval&quot;) +
ylab(&quot;average number of steps taken&quot;) 
</code></pre></li>
<li><p>Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?</p>

<pre><code class="r, echo=TRUE">mostSteps &lt;- which.max(averageStepsPerTimeBlock$meanSteps)
timeMostSteps &lt;-  gsub(&quot;([0-9]{1,2})([0-9]{2})&quot;, &quot;\\1:\\2&quot;, averageStepsPerTimeBlock[mostSteps,&#39;interval&#39;])
</code></pre></li>
</ol>

<p>Imputing missing values</p>

<ol>
<li>Calculate and report the total number of missing values in the dataset
<code>{r, echo=TRUE}
numMissingValues &lt;- length(which(is.na(activityData$steps)))
</code></li>
<li><p>Devise a strategy for filling in all of the missing values in the dataset.</p></li>
<li><p>Create a new dataset that is equal to the original dataset but with the missing data filled in.</p>

<pre><code class="r, echo=TRUE">activityDataImputed &lt;- activityData
activityDataImputed$steps &lt;- impute(activityData$steps, fun=mean)
</code></pre></li>
<li><p>Make a histogram of the total number of steps taken each day</p>

<pre><code class="r, echo=TRUE">stepsByDayImputed &lt;- tapply(activityDataImputed$steps, activityDataImputed$date, sum)
qplot(stepsByDayImputed, xlab=&#39;Total steps per day (Imputed)&#39;, ylab=&#39;Frequency using binwith 500&#39;, binwidth=500)
</code></pre>

<p>... and Calculate and report the mean and median total number of steps taken per day.</p>

<pre><code class="r, echo=TRUE">stepsByDayMeanImputed &lt;- mean(stepsByDayImputed)
stepsByDayMedianImputed &lt;- median(stepsByDayImputed)
</code></pre>

<p>Are there differences in activity patterns between weekdays and weekends?</p></li>
<li><p>Create a new factor variable in the dataset with two levels - &quot;weekday&quot; and &quot;weekend&quot; indicating whether a given date is a weekday or weekend day.</p>

<pre><code class="r, echo=TRUE">activityDataImputed$dateType &lt;-  ifelse(as.POSIXlt(activityDataImputed$date)$wday %in% c(0,6), &#39;weekend&#39;, &#39;weekday&#39;)
</code></pre></li>
<li><p>Make a panel plot containing a time series plot</p>

<pre><code class="r, echo=TRUE">averagedActivityDataImputed &lt;- aggregate(steps ~ interval + dateType, data=activityDataImputed, mean)
ggplot(averagedActivityDataImputed, aes(interval, steps)) + 
geom_line() + 
facet_grid(dateType ~ .) +
xlab(&quot;5-minute interval&quot;) + 
ylab(&quot;avarage number of steps&quot;)
</code></pre></li>
</ol>

</body>

</html>
