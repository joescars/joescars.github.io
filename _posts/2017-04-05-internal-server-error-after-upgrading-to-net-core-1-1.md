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
So I decided to start upgrading all my Azure Web Apps and associated NuGet packages to to .NET Core 1.1. The process was very simple and everything worked perfectly locally.

Naturally, I pushed my update to my GitHub repo, Azure takes over and does the deployment. Then.. **<span style="color: #ff0000;">FAIL</span>**<figure id="attachment_358" style="width: 840px" class="wp-caption alignleft">

<img class="size-large wp-image-358" src="https://joeraio.com/wp-content/uploads/2017/04/web-error-2-1024x694.png" alt="" width="840" height="569" srcset="http://localhost/wp-content/uploads/2017/04/web-error-2-1024x694.png 1024w, http://localhost/wp-content/uploads/2017/04/web-error-2-300x203.png 300w, http://localhost/wp-content/uploads/2017/04/web-error-2-768x520.png 768w, http://localhost/wp-content/uploads/2017/04/web-error-2.png 1088w" sizes="(max-width: 840px) 100vw, 840px" /><figcaption class="wp-caption-text">An Error occurred while starting the application.</figcaption></figure> 

First thing I try is to re-sync the repo through the admin. No luck. Thankfully though, there is a simple solution.

Turns on when doing the deploy there are left over dll&#8217;s in your **wwwroot** folder that will conflict with your new dll&#8217;s. This causes the web app to fail.

To resolve this error, simply do the following.

Stop your web app

<img class="size-full wp-image-342 alignnone" src="https://joeraio.com/wp-content/uploads/2017/04/stop.png" alt="" width="844" height="345" srcset="http://localhost/wp-content/uploads/2017/04/stop.png 844w, http://localhost/wp-content/uploads/2017/04/stop-300x123.png 300w, http://localhost/wp-content/uploads/2017/04/stop-768x314.png 768w" sizes="(max-width: 844px) 100vw, 844px" />

Go to your Kudu console either through advanced tools or directly from the scm url.

<img class="size-full wp-image-343 alignnone" src="https://joeraio.com/wp-content/uploads/2017/04/kudu-advanced-tools.png" alt="" width="283" height="184" />

Go to the CMD or Powershell prompt. Navigate to the /site/wwwroot folder. Delete everything in it!

<img class="size-large wp-image-349 alignnone" src="https://joeraio.com/wp-content/uploads/2017/04/www-root-empty-1024x657.png" alt="" width="840" height="539" srcset="http://localhost/wp-content/uploads/2017/04/www-root-empty-1024x657.png 1024w, http://localhost/wp-content/uploads/2017/04/www-root-empty-300x192.png 300w, http://localhost/wp-content/uploads/2017/04/www-root-empty-768x493.png 768w, http://localhost/wp-content/uploads/2017/04/www-root-empty.png 1082w" sizes="(max-width: 840px) 100vw, 840px" />

Go back to deployment options and re-sync your Github Repo

<img class="alignleft size-full wp-image-350" src="https://joeraio.com/wp-content/uploads/2017/04/deployment-options.png" alt="" width="277" height="265" /><img class="size-full wp-image-351 alignnone" src="https://joeraio.com/wp-content/uploads/2017/04/github-azure-web-app-sync.png" alt="" width="592" height="215" srcset="http://localhost/wp-content/uploads/2017/04/github-azure-web-app-sync.png 592w, http://localhost/wp-content/uploads/2017/04/github-azure-web-app-sync-300x109.png 300w" sizes="(max-width: 592px) 100vw, 592px" />

<img class="alignleft size-full wp-image-353" src="https://joeraio.com/wp-content/uploads/2017/04/github-sync-confirm.png" alt="" width="940" height="182" srcset="http://localhost/wp-content/uploads/2017/04/github-sync-confirm.png 940w, http://localhost/wp-content/uploads/2017/04/github-sync-confirm-300x58.png 300w, http://localhost/wp-content/uploads/2017/04/github-sync-confirm-768x149.png 768w" sizes="(max-width: 940px) 100vw, 940px" />

Once this is done start your application and you will be good to go!