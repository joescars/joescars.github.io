---
id: 421
title: Azure Data Lake Extension for Azure Functions
date: 2018-09-10T14:51:54+00:00
author: Joe
layout: post
guid: https://joeraio.com/?p=421
permalink: /azure-data-lake-extension-for-azure-functions/
categories:
  - Azure Functions
  - Data Lake
  - Serverless
  - Azure
tags: [azure, azure functions, data lake, azure]
---
As part of our internal hack week a few months back, I built out an extension for Azure Functions that lets you connect directly to an Azure Data Lake Store. This extension can be used as an input or output binding.

You can install the extension from the official repo: <https://github.com/Azure/azure-functions-datalake-extension>

Example function using output binding:

<script src="https://gist.github.com/joescars/58986d82d78eb1e1f0452df842489e81.js"></script>

Example function using input binding:  

<script src="https://gist.github.com/joescars/9f8ab2293ee7d87318063a70f6d06aac.js"></script>

Happy Coding!