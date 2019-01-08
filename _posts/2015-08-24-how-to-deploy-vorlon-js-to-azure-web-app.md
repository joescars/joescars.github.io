---
id: 114
title: How to Deploy Vorlon.JS to Azure Web App
date: 2015-08-24T10:37:31+00:00
author: Joe
excerpt: This video will walk you through how to deploy Vorlon.JS to an Azure Web App using Git version control. The pre-requisites for this video are that you have an active Microsoft Azure account and node.js installed.
layout: post
guid: http://joeraio.com/?p=114
permalink: /how-to-deploy-vorlon-js-to-azure-web-app/
price_table:
  - 'a:0:{}'
image: /assets/wp-content/uploads/2015/08/deploy-vorlonjs-to-azure-web-app.png
categories:
  - JavaScript
  - Microsoft Azure
  - Web Development
---
This video will walk you through how to deploy <a href="http://www.vorlonjs.io/" target="_blank">Vorlon.JS</a> to an Azure Web App using <a href="https://git-scm.com/" target="_blank">Git</a> version control. The pre-requisites for this video are that you have an active Microsoft Azure account and node.js installed.



**Instructions** 

  1. Create new web app using custom create 
      1. From the configure section, Turn on Web Sockets – Save
  2. Go Back to Dashboard
  3. Setup deployment from source control 
      1. Select Local Git Repository 
          1. This will create the Git repository
  4. Install VorlonJS on your machine
  5. _nodejs must be installed already. If not, install from nodejs.org_
  6.   1. Open node.js command prompt
      2. Enter the following: npm i –g vorlon
  7. Test Vorlon 
      1. Type the following: vorlon 
          1. To terminate: Terminate – ctrl + c
  8. You should be in your user folder. We must now browse to the proper folder 
      1. cd appdata
      2. cd roaming
      3. cd npm
      4. cd node_modules
      5. cd vorlon
  9. Type: start . (this will launch explorer)
 10. Open another explorer 
      1. Create a new folder (ex C:\Dev\VorlonJS)
 11. Copy _node_modules_ folder from _vorlon_ folder to the new folder
 12. Copy all the files WITHIN the server folder (within vorlon folder) to the new folder you created
 13. Create new text file package.json
 14. Edit the file and add the following

<pre title="package.json" class="lang:default decode:true ">{
	"name":"vorlon",
	"version":"0.0.15",
	"description": "vorlon",
	"main": "server.js"
}</pre>

Now we want to commit this into azure website

  1. Open Git Bash 
      1. Get GIT url from Azure Deployments page 
          1. It also has instructions
      2. Browse to your vorlonjs folder you created (example: cd /c/dev/vorlonjs)
      3. Now we are going to use the commands from azure to add files to git 
          1. git init
          2. git add .
          3. git commit –m &#8220;initial commit&#8221;
      4. Again copying right from azure to add remote repository and commit 
          1. git remote add azure (this line will be in your azure account)
          2. git push azure master 
              1. If need be use reset deployment credentials
          3. Enter your password
  2. When it&#8217;s done it should now say &#8220;Active Deployment&#8221;
  3. Go to dashboard / URL
  4. First time it loads it may error out, just refresh it
  5. That&#8217;s it!

&nbsp;