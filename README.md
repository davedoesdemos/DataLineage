# Data Lineage

**Produced by Dave Lusty**

## Introduction

This is a demo showing you a simple way to create and store data lineage information. While a product such as Apache Atlas can allow you to centralise this information it does by necessity add some complexity and infrastructure as well as the associated operating costs of running the solution. The technique shown here does have some limitations, but it can offer a very simple way to get lineage information into your data, it doesn't rely on any specific tools or products, and the information is accessible and readable in any tool you can access your data from. Because it doesn't require connectors or APIs this can be applied directly to every row of your data right where it's processed.

There is a [video of this demo (no video yet)](https://youtu.be/S5kxhPDQaxs)

The demo will begin with a simple CSV file with a couple of rows of data. We will copy this in an "ingest" pipeline and add source information in a lineage column. Next we will copy the data in a "processing" pipeline which will convert to Parquet format and add some information to the lineage column.

### Resources

Needless to say, if you store a large text column on every row of data your data will become larger and processing will take longer. This trade off will be different in different scenarios. In uncompressed CSV you'll see a large increase in size, while compressed Parquet won't show much increase since every row will have almost identical data meaning it compresses very well with columnar compression techniques. While adding in a row does add processing time, if you're already processing the data then the increase should only be slight. If you're currently doing a binary file copy then this may slow the copy but the trade off of recording the source might be worth it in your use-case. 
In any event, this technique gives you another option in your toolkit.

### Data Structure

In this demo we will be using a JSON formatting in the lineage data column. Unfortunately due to the various technologies and formats I cannot give you full guidance on what to use and where because not all strings will be supported in all file formats. The JSON used here is an array of objects, allowing a new row to be added every time the row of data is processed. With the CSV format we start with we cannot add a new line to the JSON, and so if you wish to add multiple records to a CSV you might need to contrive some other format, or use character escaping to work around the limitations. Once in a more structured binary format we are more free to support full JSON.

### Schema

You are completely free to form a structure to suit your lineage requirements. My advice here is to spend as little time as you can developing a schema. JSON is exceptionally flexible and will allow you to add in new fields at any time in the future without causing failure with existing information. You can add different columns within different entries if needed and these can be interpreted at the time of reading back the information. Treat this as a NoSQL store, not as a 3NF database.

As a good start I recommend the following information be recorded:

processingDate: the date on which the pipeline started. Avoid trying to record datetimes on a per row basis as this is very compute heavy.
dataSource: the source of the data from the perspective of this pipeline. The ultimate source will be at the top of the lineage, and we want to see where it goes along the way. This could be a JSON array if multiple sources are being merged.
pipeline: this is a unique identifier for your pipeline. This could be its name or some other way to see what process touched the data.
pipelineRunID: if you have a unique ID for the pipeline run this might be useful to record.
pipelineVersion: if you have good versioning this can be useful but is not essential.

## Infrastructure

For this demo we will be using Azure Data Factory and a storage account. 

Deploy the infrastructure required for this workshop by clicking the button below:

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdavedoesdemos%2FDataLakeInADay%2Fmaster%2Finfrastructure%2FAzureResourceGroup1%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
    </a>