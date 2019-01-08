---
id: 248
title: Deploy Azure Web App or Azure Function from Repository Folder
date: 2017-01-23T11:00:33+00:00
author: Joe
excerpt: 'Learn how easy it is to deploy an Azure Web App or Azure function from a specific folder inside of a GitHub repository. This lets you keep all your related projects organized into one main folder. '
layout: post
guid: http://joeraio.com/?p=248
permalink: /deploy-azure-web-app-or-azure-function-from-repository-folder/
price_table:
  - 'a:0:{}'
image: /assets/wp-content/uploads/2017/01/azure-functions-deployment.jpg
categories:
  - ASP.NET Core
  - Azure Functions
  - Microsoft Azure
---
I participate in a lot of hackathons. Whether its an internal up-skilling event, university hackathon, or on site with a partner. These are often some of the most fun and frustrating things I get to work on.

At just about every hackathon, we use GitHub to setup a central repo with all the parts of the project we are working on. This could for example contain a web app, azure functions, Xamarin app, console app etc. We normally will just throw everything into its own folder and work from the one repo to keep it easy for everyone.

Often, I want to deploy my web app (and only the web app) right from this repo as I am pushing changes to GitHub. Thankfully through Azure, not only is it very easy to setup continuous deployment using GitHub, but you can also specific what folder within the repo to deploy from.

To deploy an ASP.NET Core web app to Azure from a specific folder in your repository, you simply have to add a new application setting called &#8216;project&#8217; and point it to the source folder of the project.

For example; I have a project consisting of a UWP slideshow application and my web app to manage the slides. The repository can be found here: <https://github.com/joescars/MICMediaSlideshow>

In this instance, I only want to deploy the folder &#8216;MicMediaManager&#8217; to my web app. To do this I [connected my Azure Web App to GitHub for continuous deployment](https://docs.microsoft.com/en-us/azure/app-service-web/app-service-continuous-deployment) and then added the following settings to my app settings.

<img class="alignnone size-large wp-image-252" src="https://joeraio.com/wp-content/uploads/2017/01/azure-web-app-deploy-repo-folder-1024x393.png" alt="" width="840" height="322" srcset="http://localhost/wp-content/uploads/2017/01/azure-web-app-deploy-repo-folder-1024x393.png 1024w, http://localhost/wp-content/uploads/2017/01/azure-web-app-deploy-repo-folder-300x115.png 300w, http://localhost/wp-content/uploads/2017/01/azure-web-app-deploy-repo-folder-768x295.png 768w, http://localhost/wp-content/uploads/2017/01/azure-web-app-deploy-repo-folder.png 1039w" sizes="(max-width: 840px) 100vw, 840px" />

That&#8217;s it! Now I can update each folder independently and my web app will only publish changes within the source folder specified.

**So&#8230; what about Azure Functions?**

This same method works with **Azure Functions** as well. I&#8217;ve setup a sample repository here: <https://github.com/joescars/AzureFunctionsCustomDeployment> in which I have a folder with my Functions inside their own sub-folder. I then go to the function app settings, add a project setting and it will deploy all my Functions from that folder.

<img class="alignnone size-large wp-image-254" src="https://joeraio.com/wp-content/uploads/2017/01/azure-function-app-settings-1024x185.png" alt="" width="840" height="152" srcset="http://localhost/wp-content/uploads/2017/01/azure-function-app-settings-1024x185.png 1024w, http://localhost/wp-content/uploads/2017/01/azure-function-app-settings-300x54.png 300w, http://localhost/wp-content/uploads/2017/01/azure-function-app-settings-768x139.png 768w, http://localhost/wp-content/uploads/2017/01/azure-function-app-settings-1200x216.png 1200w, http://localhost/wp-content/uploads/2017/01/azure-function-app-settings.png 1214w" sizes="(max-width: 840px) 100vw, 840px" />

<img class="alignnone size-large wp-image-255" src="https://joeraio.com/wp-content/uploads/2017/01/azure-function-custom-folder-app-setting.png" alt="" width="683" height="391" srcset="http://localhost/wp-content/uploads/2017/01/azure-function-custom-folder-app-setting.png 683w, http://localhost/wp-content/uploads/2017/01/azure-function-custom-folder-app-setting-300x172.png 300w" sizes="(max-width: 683px) 100vw, 683px" />

Happy Coding!