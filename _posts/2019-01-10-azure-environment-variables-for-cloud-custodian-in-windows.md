---
title: Azure environment variables for Cloud Custodian in Windows
meta-description: Learn how to set up Azure service principal authentication environment variables for use with Cloud Custodian control plane.
excerpt: Learn how to set up Azure service principal authentication environment variables for use with Cloud Custodian control plane.
#date: 2018-09-10T14:51:54+00:00
author: Joe
layout: post
permalink: /azure-environment-variables-for-cloud-custodian-in-windows/
categories:
  - Control Plane
  - Cloud Custodian
  - Python
  - Powershell
  - Open Source
tags: [cloud custodian, control plane, python, azure, powershell, open source, windows, wsl, capital one]
---
This week I had the opportunity to hack with my colleagues on [Cloud Custodian](https://cloudcustodian.io).

If you are not familiar with Cloud Custodian, it is a control plane that lets you write and easily enforce policies across all your cloud resources. It works with AWS, Azure and GCP.

As part of the setup process you need create a new [virtualenv](https://virtualenv.pypa.io/en/stable/), then [create a service principal](https://cloudcustodian.io/docs/azure/authentication.html) and finally authenticate your local instance by setting enviornment variables.

The easiest way to do this (IMO) is to add these variables to the activate script for your custodian virtualenv. I've included directions below dependong on which console you use.

## Command Line

Open *custodian/scripts/activate.bat* and add the following to the end of the file.

```bat
REM Azure Service Principal Authentication
set "AZURE_TENANT_ID=<tentant id>"
set "AZURE_SUBSCRIPTION_ID=<subscription id>"
set "AZURE_CLIENT_ID=<client id>"
set "AZURE_CLIENT_SECRET=<secret>"
```

## PowerShell

Open *custodian/scripts/activate.ps1* and add the following to the end of the file.

```powershell
# Azure Service Principal Authentication
$env:AZURE_TENANT_ID="<tentant id>"
$env:AZURE_SUBSCRIPTION_ID="<subscription id>"
$env:AZURE_CLIENT_ID="<client id>"
$env:AZURE_CLIENT_SECRET="<secret>"
```

## Windows Subsystem for Linux (WSL)

Open custodian/bin/activate and add the following to the end of the file. 

```python
# Azure Service Principal Authentication
AZURE_TENANT_ID="<tentant id>"
export AZURE_TENANT_ID
AZURE_SUBSCRIPTION_ID="<subscription id>"
export AZURE_SUBSCRIPTION_ID
AZURE_CLIENT_ID="<client id>"
export AZURE_CLIENT_ID
AZURE_CLIENT_SECRET="<secret>"
export AZURE_CLIENT_SECRET
```

Everytime you activate your custodian virtualenv going forward your Service Prinicipal Authentication will be set!