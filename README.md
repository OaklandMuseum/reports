## Jaspersoft Reports for CollectionSpace
This is a collection of Jaspersoft reports for Oakland Museum's CollectionSpace instance. 

N.B. Many of these reports use SQL queries specifically created to call from custom postgres tables and will generate errors when added to a default CollectionSpace system.

## Subreports
A number of subreports were created that are referenced within a parent report. They are named with a **subreport_** prefix. Their corresponding .jasper files are also included as the compiled version of the subreport are called from within the parent report.

## Parameters
There are a few parameters used in these reports that will need to be updated for other custom CollectionSpace instances.
* $P{tenantid} - the custom tenant ID. The default value is the OMCA tenant ID, "35".
* $P{csid} -  useful for local testing using Jaspersoft Studio (or older iReports). The default value is ignored when run within CollectionSpace
* $P{cspace_server} - this is used for creating links to a specific server. This parameter is used in a few ways within these reports
  * adding a link to the record CSID that opens the full record view in a browser (not applicable in the PDF report, unfortunately.)
  * linking to a script that pulls in media blobs (see **Media blobs** section below.)


## Media blobs
Many of these reports pull in an image for each Cataloging record that returned. It is important to note that CollectionSpace doesn't  provide a means to request images from the database but rather through the backend API. For this, a small custom script was created that when given a media blob CSID value will return the respective image. 

For these reports the custom script tag is wrapped within an \<imageExpression/\> tag
```xml
<imageExpression><![CDATA["http://" + $P{cspace_server} +  "/fetchimage/index.php?csid=" + $F{blobcsid}]]></imageExpression>
```

This specific custom script is not provided for this repository but is easy enough to create using a scripting language of your choice and a cURL-like method. Care must be placed to avoid opening up this access point to the world, of course!

## Adding a report to CollectionSpace

Comprehensive instructions are provided from the [CollectionSpace configuration documentation](https://wiki.collectionspace.org/display/DOC/How+to+add+and+run+reports).

Simplified instructions follow.

### Report location
All reports including subreports should live in the following server directory:
```
/usr/local/share/apache-tomcat-7.0.57/cspace/reports
```

### Add record to the Reports procedure
To add a report to the system it first needs to be added to the Reports procedure using the backend API.

Here is sample cURL command to add the report to the CollectionSpace system 
```
curl -X POST http://localhost:8180/cspace-services/reports -i -u admin@cspace-instance.url:password -H "Content-Type: application/xml" -T payload.xml
```
### Create payload file
The payload.xml file is constructed like this

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<document name="report">
    <ns2:reports_common
    xmlns:ns2="http://collectionspace.org/services/report">
        <name>{{NAME OF REPORT}}</name>
        <notes>{{REPORT NOTES}}</notes>
        <forDocTypes>
            <forDocType>{{DOC TYPE}}</forDocType>
        </forDocTypes>
        <supportsSingleDoc>true</supportsSingleDoc>
        <supportsDocList>false</supportsDocList>
        <supportsGroup>false</supportsGroup>
        <supportsNoContext>true</supportsNoContext>
        <filename>{{REPORT FILE NAME}}.jrxml</filename>
        <outputMIME>application/pdf</outputMIME>
    </ns2:reports_common>
</document>
```

* {{NAME OF REPORT}} is the name of the report displayed in the UI. Must be unique.
* {{REPORT NOTES}} is added to the database row. Not displayed in the UI.
* {{DOC TYPE}} is the singluar form of the document type. Examples include
  * "CollectionObject"
  * "Conditioncheck"
  * "Conservation"
  * "Group"
* {{REPORT FILE NAME}} is the exact jrxml file name located in the default report location (see **Report location** section above).

