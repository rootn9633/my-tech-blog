---
title: Using Azure Datalake Storage with Python
date: "2019-10-28T22:12:03.284Z"
template: "post"
draft: false
slug: "/posts/Using-Azure-Datalake-Storage-with-Python/"
category: "Azure"
tags:
  - "Azure"
  - "ADLS"
  - "REST API"
  - "Python"
description: "How to authenticate and use Azure Datalake Storage REST API in Python"
---

## Reason

I explored the Azure Data Lake Storage REST API. Using the  [API](https://docs.microsoft.com/en-us/rest/api/storageservices/datalakestoragegen2/path), I have successfully created a simple python script to upload a file onto ADLS. I will detail the steps below.

## Project file description

| File          | Description        |
| ------------- |-------------|
| ADLSconnection.py| The main program. |
| ADLSconfig.py      | The config file  |
| zebra stripes | are neat      |
The main program.
ADLSconfig.py
The config file containing storage account 
To use the script, simply run `python3 ADLSconnection.py [file name]`. The file will then be uploaded to the path and storage account as specified in the config file.

## Overview

The steps to upload a file to ADLS is as below. Note that uploading one file requires three different API calls.

1. [Authentication](https://docs.microsoft.com/pl-pl/rest/api/storageservices/authorize-with-shared-key#constructing-the-canonicalized-resource-string) 
2. [Create file](https://docs.microsoft.com/en-us/rest/api/storageservices/datalakestoragegen2/path/create) : Create an empty file in ADLS first.
3. [Append file](https://docs.microsoft.com/en-us/rest/api/storageservices/datalakestoragegen2/path/update) : Append the data to the file, note that the content haven’t been inserted yet as this point. This API can be called in parallel to upload a large file.
4. [Flush data](https://docs.microsoft.com/en-us/rest/api/storageservices/datalakestoragegen2/path/update) : After all the data has been uploaded, use this API to flush the data into the file.

## Authentication

To authorize the API request, one needs the storage account name and key as well as information of the API request. This step corresponds to the getSAS function in my script.

First I generated a string to sign with the information of the API request, then encoded this string by using the HMAC-SHA256 algorithm over the UTF-8-encoded storage account key.

Any API request must include the Authorization header with the format ‘SharedKey {storage account name}:{signed string}

## Create file

To create a file, I made a request as the following:

``` http
PUT https://{accountName}.{dnsSuffix}/{filesystem}/{path}?resource=file
```

If successful, it will return 201 Created.

Note that the API will be successful even if there is already a existing file with the same name. To check if a file exists, use the  [list](https://docs.microsoft.com/en-us/rest/api/storageservices/datalakestoragegen2/path/list) API.

## Append file

After the file is created, I will make a request as the following:

``` http
PATCH https://{accountName}.{dnsSuffix}/{filesystem}/{path}?action=append&position={position}
```

The request body is the content I wish to append. If successful, it will return 202 Accepted.**The data is not yet written to the file at this point.**

Note that I have included a position query. This value indicates the position in the file the content should be inserted. This means that large files could be uploaded in batches. I have implemented a threaded batch upload in my script.

## Flush data

After all the content is appended, a request as the following:

``` http
PATCH https://{accountName}.{dnsSuffix}/{filesystem}/{path}?action={action}&position={content length}
```

This will flush the data into the file. The content length must be equal to the length of the file after all data has been written.
