# dataPARC.DataSeries SDK 

A .NET library for creating clients for the dataPARC.DataSeries services

## Documentation

See the [docs](https://dataparc.github.io/dataPARC.DataSeries.SDK/v1.0.0/) for full API reference.

## Prerequisites

Access to a nuget feed containing the dataPARC packages distributed with dataPARC.DataSeries. To add a feed to a project, add the following nuget.config

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="dataparc" value="[path to nuget feed]" />
  </packageSources>
</configuration>
```

With the feed configured, add a nuget reference to dataPARC.DataSeries.SDK

`dotnet add package dataPARC.DataSeries.SDK`

## Examples

- [Clients](#clients)
- [Authorization](#authorization)
- [Reading Data](#reading-data)
- [Reading Current Values](#reading-current-values)
- [Reading Up To Current Values](#reading-up-to-current-values)
- [Reading Specific Timestamps](#reading-specific-timestamps)
- [Reading Source Info](#reading-source-info)
- [Getting Quick Statistics](#getting-quick-statistics)
- [Exporting data](#exporting-data)
- [Reading Data By Aggregate Type](#reading-data-by-aggregate-type)

### Clients

```cs
// Create client that have methods for calling DataSeries service functions. 

// Create a client with a hostname, port number, and a certificate validator.
string host = "hostname";
int port = 12340;

await using var dataSeriesClient = new DataSeriesClient(host, port, CertificateValidation.AcceptAllCertificates);

```

### Authorization

```cs
// When creating clients, you are required to pass in an ICertificateValidation that is used to validate the server's certificate. This can be a custom validator as long as it implements ICertificateValidation.
ICertificateValidation certVal = CertificateValidation.DefaultHttpClientHandlerValidator;
await using var readClient = new DataSeriesClient(hostname, port, certVal);

// The other auth related parameter the clients have is an optional IAuthProvider that is used to get access tokens from a token provider. This can also be a custom provider as long as it implements IAuthProvider. 
var token = AuthProviderFactory.StartDeviceCodeFlow("authority", "client_id", "scope", res =>
{
    // Pass info from res to user/caller to run device code flow.
};
IAuthProvider provider = AuthProviderFactory.CreateInMemoryTokenProvider(token);
await using var writeClient = new DataSeriesClient(hostname, port, certVal, provider);
```

### Reading Data

```cs
await using var client = new DataSeriesClient(hostname, port, CertificateValidation.AcceptAllCertificates);

// Read datapoints for a given time period for a single tag.

// Arrange parameters

var tagName = new FullyQualifiedTagName
{
    Tag = "Full Utag",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag = new TagQuery(tagName);
var startTime = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var endTime = DateTime.SpecifyKind(DateTime.Today.AddHours(3), DateTimeKind.Utc);

// Optional parameters ReturnStartBounds and ReturnEndBounds, default values are true
var returnStartBounds = false;
var returnEndBounds = false;
var readParams1 = new ReadRawParameters(tag, startTime, endTime, returnStartBounds, returnEndBounds);
var readRes1 = await client.ReadRaw(readParams1);

var points = readRes1.DataPoints;

  // Read datapoints for given time periods for a collection of tags.

var tagName1 = new FullyQualifiedTagName
{
    Tag = "Full Utag1",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag1 = new TagQuery(tagName1);
var startTime1 = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var endTime1 = DateTime.SpecifyKind(DateTime.Today.AddHours(3), DateTimeKind.Utc);
var tagParam1 = new ReadRawTagParameters(tag1, startTime1, endTime1);

var tagName2 = new FullyQualifiedTagName
{
    Tag = "Full Utag2",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag2 = new TagQuery(tagName2);
var startTime2 = DateTime.SpecifyKind(DateTime.Today.AddHours(1), DateTimeKind.Utc);
var endTime2 = DateTime.SpecifyKind(DateTime.Today.AddHours(4), DateTimeKind.Utc);
var tagParam2 = new ReadRawTagParameters(tag2, startTime2, endTime2);

// Optional parameters ReturnStartBounds and ReturnEndBounds, default values are true
var returnStartBounds = false;
var returnEndBounds = false;
var readParams2 = new ReadRawBulkParameters(new ReadRawTagParameters[] {tagParam1, tagParam2} , returnStartBounds, returnEndBounds);
var readRes2 = await client.ReadRawBulk(readParams2);
foreach (var res in readRes2) {
    // Each tag requested is returned in the same object as the singular ReadRawAsync call.
}


// Stream raw data for a collection of tags. Streaming is equivalent to reading in the data it can return. It provides more flexibility
// around client memory constraints.

// Optional parameter PageSize to define how much data to expect in each chunk of the stream. Default value is 512.
var pageSize = 1024;
var streamParams1 = new StreamRawBulkParameters(new ReadRawTagParameters[] {tagParam1, tagParam2} , returnStartBounds, returnEndBounds, pageSize);
var streamRes1 = client.StreamRawBulk(streamParams1);

await foreach (var result in streamRes1)
{
    // You can do further processing with the result
}

```

### Reading Current Values

```cs
await using var client = new DataSeriesClient(hostname, port, CertificateValidation.AcceptAllCertificates);

  // Read current values for a collection of tags.

  var tagName1 = new FullyQualifiedTagName
{
    Tag = "Full Utag1",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag1 = new TagQuery(tagName1);
var tagParam1 = new ReadCurrentValueParameters(tag1);

var tagName2 = new FullyQualifiedTagName
{
    Tag = "Full Utag2",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag2 = new TagQuery(tagName2);
var tagParam2 = new ReadCurrentValueParameters(tag2);
var readRes1 = await client.ReadCurrentValues(new ReadCurrentValueParameters[] {tagParam1, tagParam2});

foreach (var res in readRes1) {
    var point = res.DataPoint; 
}

```

### Reading Up To Current Values

```cs
await using var client = new DataSeriesClient(hostname, port, CertificateValidation.AcceptAllCertificates);

  // Read up to current values for a collection of tags from a start time.

var tagName1 = new FullyQualifiedTagName
{
    Tag = "Full Utag1",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag1 = new TagQuery(tagName1);
var startTime1 = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var tagParam1 = new ReadRawToCurrentValueParameters(tag1, startTime1);

var tagName2 = new FullyQualifiedTagName
{
    Tag = "Full Utag2",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag2 = new TagQuery(tagName2);
var startTime2 = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var tagParam2 = new ReadRawToCurrentValueParameters(tag2, startTime2);
var readRes1 = await client.ReadRawToCurrentValues(new ReadRawToCurrentValueParameters[] {tagParam1, tagParam2});

foreach (var res in readRes1) {
    var point = res.DataPoints; 
}

```

### Reading Specific Timestamps

```cs
await using var client = new DataSeriesClient(hostname, port, CertificateValidation.AcceptAllCertificates);

 // Get tag datapoint for given timestamp for a collection of tags.
 var tagName1 = new FullyQualifiedTagName
{
    Tag = "Full Utag1",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag1 = new TagQuery(tagName1);
var startTime1 = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var tagParam1 = new ReadAtTimeParameters(tag1, startTime1);

var tagName2 = new FullyQualifiedTagName
{
    Tag = "Full Utag2",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag2 = new TagQuery(tagName2);
var startTime2 = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var tagParam2 = new ReadAtTimeParameters(tag2, startTime2);
var readRes1 = await client.ReadAtTimeBulk(new ReadAtTimeParameters[] {tag1, tag2});

foreach (var res in readRes1) {
    var point = res.DataPoints[0];
}
```

### Reading Source Info

```cs
await using var client = new DataSeriesClient(hostname, port, CertificateValidation.AcceptAllCertificates);

// Read source info for a single tag.

var tagName = new FullyQualifiedTagName
{
    Tag = "Full Utag",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag = new TagQuery(tagName);
var readRes = await client.ReadSourceInfo(new ReadSourceInfoParameters(tag));

var sourceInfo = readRes.Value;

// Read source info for a collection of tags.

var tagName1 = new FullyQualifiedTagName
{
    Tag = "Full Utag1",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag1 = new TagQuery(tagName1);
var tagParam1 = new ReadSourceInfoParameters(tag1);

var tagName2 = new FullyQualifiedTagName
{
    Tag = "Full Utag2",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag2 = new TagQuery(tagName2);
var tagParam2 = new ReadSourceInfoParameters(tag2);
var readRes = await client.ReadSourceInfoBulk(new ReadSourceInfoParameters[] {tagParam1, tagParam2});

foreach (var res in readRes)
{
    var sourceInfo = res.Value;
}

```

### Getting Quick Statistics

```cs
await using var client = new DataSeriesClient(hostname, port, CertificateValidation.AcceptAllCertificates);

 // Read quick statistics for a collection of tags.

var tagName1 = new FullyQualifiedTagName
{
    Tag = "Full Utag1",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag1 = new TagQuery(tagName1);

var startTime1 = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var endTime1 = DateTime.SpecifyKind(DateTime.Today.AddHours(3), DateTimeKind.Utc);
var isTextSeries = false;
var interpolationType = InterpolationType.Continuous;
var dataType = TagDataType.Double;
var isDrawToNow = true;
var tagQuickStatisticsParameters1 = new TagQuickStatisticsParameters(tag1, startTime1, endTime1, isTextSeries,, interpolationType dataType, isDrawToNow);

var tagName2 = new FullyQualifiedTagName
{
    Tag = "Full Utag2",
    Interface = "",
    InterfaceGroup = "",
    InterfaceSet = ""
};
var tag2 = new TagQuery(tagName2);

var startTime2 = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var endTime2 = DateTime.SpecifyKind(DateTime.Today.AddHours(3), DateTimeKind.Utc);
var tagQuickStatisticsParameters2 = new TagQuickStatisticsParameters(tag2, startTime2, endTime2, isTextSeries,, interpolationType dataType, isDrawToNow);

var aggregates = new AggregateType?[]
{
    AggregateType.Maximum,
    AggregateType.Minimum,
    AggregateType.Count
};
var getQuickStatisticsRequest = new QuickStatisticsParameters
(
    new TagQuickStatisticsParameters[]
    {
        tagQuickStatisticsParameters1, tagQuickStatisticsParameters2
    }, 
    aggregates
);

var getQuickStatisticsReply = await client.GetQuickStatistics(getQuickStatisticsRequest);
foreach (var res in getQuickStatisticsReply)
{
    var quickStatistics = res;
}

// Read latest quick statistics for a collection of tags.

var timeAvg = 1.1;
var rawAvg = 1.5;
var stdDev = 1.2;
var min = 0.5;
var max = 2.1;
var count = 100;
var sum = 50;
var duration = 20;
var lastGoodQualityTime = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var timeAvgToLastPoint = 20;
var currentStatistics1 = new QuickStatisticsResult("Full Utag1", timeAvg, rawAvg, stdDev, min, max, count, sum, duration, lastGoodQualityTime, timeAvgToLastPoint);
var latestQuickStatisticsParameters1 = new TagLatestQuickStatisticsParameters(tagQuickStatisticsParameters1, currentStatistics1);
var getLatestQuickStatisticsRequest = new LatestQuickStatisticsParameters
(
    new TagLatestQuickStatisticsParameters[]
    {
        latestQuickStatisticsParameters1
    }, 
    aggregates
);
var getQuickStatisticsReply = await client.GetLatestQuickStatistics(getLatestQuickStatisticsRequest);
foreach (var res in getQuickStatisticsReply)
{
    var quickStatistics = res;
}

```

### Exporting data

```cs
await using var client = new DataSeriesClient(hostname, port, CertificateValidation.AcceptAllCertificates);

// Stream exporting raw data without interpolation for a collection of tags.

var tagName1 = "Full UTag1";
var deltaTime1 = 0;
var interpolationType1 = InterpolationType.Continuous;
var forceQualityCheck1 = true;
var exportParameters1 = new ExportParameters(tagName1, deltaTime1, interpolationType1, forceQualityCheck1);

var tagName2 = "Full UTag2";
var deltaTime2 = 5;
var interpolationType2 = InterpolationType.Continuous;
var forceQualityCheck2 = false;
var exportParameters2 = new ExportParameters(tagName2, deltaTime2, interpolationType2, forceQualityCheck2);

var startTime = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var endTime = DateTime.SpecifyKind(DateTime.Today.AddHours(3), DateTimeKind.Utc);
var maximumExportSize = 1,073,741,824;
// Optional parameters IncludeBounds, default values are true 
var includeBounds = false;

var bulkExportWithoutInterpolationRequest = new BulkExportWithoutInterpolationParameters(new ExportParameters[] {exportParameters1, exportParameters2}, startTime, endTime, maximumExportSize, includeBounds);

var exportRes = client.ExportRawWithoutInterpolationBulk(bulkExportWithoutInterpolationRequest);

 await foreach (var result in exportRes)
 {
     // You can do further processing with the result
 }

// Stream exporting datapoints with to step size interpolation for a collection of tags.

var runNormalizationMode = RunNormalizationMode.StepSize;
var stepSize = 5;
var aggregateType = AggregateType.Interpolative;
var bulkExportWithToStepSizeInterpolationRequest = new BulkExportWithInterpolationParameters(new ExportParameters[] {exportParameters1, exportParameters2}, startTime, endTime, stepSize, aggregateType, maximumExportSize, runNormalizationMode, false, includeBounds);

var exportRes = client.ExportRawWithInterpolationBulk(bulkExportWithToStepSizeInterpolationRequest);

 await foreach (var result in exportRes)
 {
     // You can do further processing with the result
 }

 // Stream exporting datapoints with to all times interpolation for a collection of tags.

var runNormalizationMode = RunNormalizationMode.AllTimes;
var stepSize = 0;
var aggregateType = AggregateType.Interpolative;
var isInterpolateStateTag = true;
var bulkExportWithToStepSizeInterpolationRequest = new BulkExportWithInterpolationParameters(new ExportParameters[] {exportParameters1, exportParameters2}, startTime, endTime, stepSize, aggregateType, maximumExportSize, runNormalizationMode, isInterpolateStateTag, includeBounds);

var exportRes = client.ExportRawWithInterpolationBulk(bulkExportWithToStepSizeInterpolationRequest);

 await foreach (var result in exportRes)
 {
     // You can do further processing with the result
 }

```

### Reading Data By Aggregate Type

```cs
await using var client = new DataSeriesClient(hostname, port, CertificateValidation.AcceptAllCertificates);

// Stream reading datapoints by aggregate type for a collection of tags.

var tagName1 = "Full UTag1";
var deltaTime1 = 0;
var interpolationType1 = InterpolationType.Continuous;
var forceQualityCheck1 = true;
var exportParameters1 = new ExportParameters(tagName1, deltaTime1, interpolationType1, forceQualityCheck1);

var tagName2 = "Full UTag2";
var deltaTime2 = 5;
var interpolationType2 = InterpolationType.Continuous;
var forceQualityCheck2 = false;
var exportParameters2 = new ExportParameters(tagName2, deltaTime2, interpolationType2, forceQualityCheck2);

var startTime = DateTime.SpecifyKind(DateTime.Today.AddHours(0), DateTimeKind.Utc);
var endTime = DateTime.SpecifyKind(DateTime.Today.AddHours(3), DateTimeKind.Utc);
var maximumExportSize = 1,073,741,824;
// Optional parameters IncludeBounds, default values are true 
var includeBounds = false;
var runNormalizationMode = RunNormalizationMode.AllTimes;
var stepSize = 0;
var aggregateType = AggregateType.Interpolative;
var isInterpolateStateTag = true;
var bulkExportWithToStepSizeInterpolationRequest = new BulkExportWithInterpolationParameters(new ExportParameters[] {exportParameters1, exportParameters2}, startTime, endTime, stepSize, aggregateType, maximumExportSize, runNormalizationMode, isInterpolateStateTag, includeBounds);

var exportRes = client.ReadDataByAggregateBulk(bulkExportWithToStepSizeInterpolationRequest);

 await foreach (var result in exportRes)
 {
     // You can do further processing with the result
 }


```

