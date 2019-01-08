---
id: 340
title: Internal Server Error after Upgrading to .NET Core 1.1
date: 2017-04-05T11:38:28+00:00
author: Joe
excerpt: Learn how to fix Internal Server Error (An error occurred while starting the application.) after upgrading your Azure web app to .NET Core 1.1
layout: post
guid: http://joeraio.com/?p=340
permalink: /internal-server-error-after-upgrading-to-net-core-1-1/
price_table:
  - 'a:0:{}'
image: /assets/wp-content/uploads/2017/04/error-while-starting-1.1-web-app-azure-app-service.jpg
categories:
  - ASP.NET Core
  - Azure App Service
  - Microsoft Azure
---
![Internal Server Error after Upgrading to .NET Core 1.1](/assets/wp-content/uploads/2017/04/error-while-starting-1.1-web-app-azure-app-service.jpg)

So I decided to start upgrading all my Azure Web Apps and associated NuGet packages to to .NET Core 1.1. The process was very simple and everything worked perfectly locally.

Naturally, I pushed my update to my GitHub repo, Azure takes over and does the deployment. Then.. **<span style="color: #ff0000;">FAIL</span>**<figure id="attachment_358" style="width: 840px" class="wp-caption alignleft">

![An error occurred while starting the application](/assets/wp-content/uploads/2017/04/web-error-2.png)
```
An Error occurred while starting the application.
``` 

First thing I try is to re-sync the repo through the admin. No luck. Thankfully though, there is a simple solution.

Turns on when doing the deploy there are left over dll&#8217;s in your **wwwroot** folder that will conflict with your new dll&#8217;s. This causes the web app to fail.

To resolve this error, simply do the following.

Stop your web app

![Stop your web app](/assets/wp-content/uploads/2017/04/stop.png)

Go to your Kudu console either through advanced tools or directly from the scm url.

![Kudu advanced tools](/assets/wp-content/uploads/2017/04/kudu-advanced-tools.png)

Go to the CMD or Powershell prompt. Navigate to the /site/wwwroot folder. Delete everything in it!

![wwwroot empty](/assets/wp-content/uploads/2017/04/www-root-empty-1024x657.png)

Go back to deployment options and re-sync your Github Repo

![github deployment options](/assets/wp-content/uploads/2017/04/github-azure-web-app-sync.png)

![GitHub sync confirm](/assets/wp-content/uploads/2017/04/github-sync-confirm.png)

Once this is done start your application and you will be good to go!