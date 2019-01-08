---
id: 167
title: Using Google Authentication with ASP.NET Core (ASP.NET 5)
date: 2016-03-04T17:30:50+00:00
author: Joe
layout: post
guid: http://joeraio.com/?p=167
permalink: /using-google-authentication-with-asp-net-core-asp-net-5/
price_table:
  - 'a:0:{}'
image: /assets/wp-content/uploads/2016/03/asp-net-core-google-authentication-mvc-ef.png
categories:
  - ASP.NET Core
  - Google Authentication
  - MVC
---
## **\*\* UPDATE &#8211; PLEASE READ \*\***

Please visit: <https://docs.microsoft.com/en-us/aspnet/core/security/authentication/social/google-logins> for updated instructions on how to use Google Authentication with ASP.NET Core.

The instructions in this article are out of date and I have not yet had a chance to update it.

* * *



While speaking with some local developers about ASP.NET Core I was asked if integrating Google Authentication was just as easy in ASP.NET Core, Core MVC (MVC6) as it was in the past. I honestly had not tested it yet so I decided to give it a shot. Thankfully, its very simple, but slightly different, so I&#8217;ve included detailed instructions below.

Note; this assumes you have Visual Studio 2015 Update 1 and ASP.NET 5 RC1 Installed. If not you can get both from the following links:

  * <a href="https://www.visualstudio.com/" target="_blank">Visual Studio Community</a>
  * <a href="https://get.asp.net/" target="_blank">ASP.net 5 Download</a>

## Create a new ASP.NET 5 MVC Project

Go to File -> New Project

<img class="alignnone size-large wp-image-168" src="https://joeraio.com/wp-content/uploads/2016/03/1-vs-2015-new-project-1024x720.png" alt="1-vs-2015-new-project" width="1024" height="720" srcset="http://localhost/wp-content/uploads/2016/03/1-vs-2015-new-project-1024x720.png 1024w, http://localhost/wp-content/uploads/2016/03/1-vs-2015-new-project-300x211.png 300w, http://localhost/wp-content/uploads/2016/03/1-vs-2015-new-project-768x540.png 768w, http://localhost/wp-content/uploads/2016/03/1-vs-2015-new-project.png 1064w" sizes="(max-width: 1024px) 100vw, 1024px" />

Select ASP.net Web Application. Name your App and hit Ok

<img class="alignnone size-full wp-image-169" src="https://joeraio.com/wp-content/uploads/2016/03/2-Web-Application-Name.png" alt="2-Web-Application-Name" width="941" height="653" srcset="http://localhost/wp-content/uploads/2016/03/2-Web-Application-Name.png 941w, http://localhost/wp-content/uploads/2016/03/2-Web-Application-Name-300x208.png 300w, http://localhost/wp-content/uploads/2016/03/2-Web-Application-Name-768x533.png 768w" sizes="(max-width: 941px) 100vw, 941px" />

Select Web Application under &#8220;ASP.NET 5&#8221; Templates

<img class="alignnone size-full wp-image-170" src="https://joeraio.com/wp-content/uploads/2016/03/3-select-asp-net-5-web-application.png" alt="3-select-asp-net-5-web-application" width="786" height="613" srcset="http://localhost/wp-content/uploads/2016/03/3-select-asp-net-5-web-application.png 786w, http://localhost/wp-content/uploads/2016/03/3-select-asp-net-5-web-application-300x234.png 300w, http://localhost/wp-content/uploads/2016/03/3-select-asp-net-5-web-application-768x599.png 768w" sizes="(max-width: 786px) 100vw, 786px" />

Let Visual studio create the project and automatically restore the packages.

Once the solution loads, hit CTRL+F5 to run your app. Note the URL it runs on and copy this URL as we will use it later.

<img class="alignnone size-large wp-image-171" src="https://joeraio.com/wp-content/uploads/2016/03/4-run-project-to-get-url-1024x726.png" alt="4-run-project-to-get-url" width="1024" height="726" srcset="http://localhost/wp-content/uploads/2016/03/4-run-project-to-get-url-1024x726.png 1024w, http://localhost/wp-content/uploads/2016/03/4-run-project-to-get-url-300x213.png 300w, http://localhost/wp-content/uploads/2016/03/4-run-project-to-get-url-768x545.png 768w, http://localhost/wp-content/uploads/2016/03/4-run-project-to-get-url.png 1067w" sizes="(max-width: 1024px) 100vw, 1024px" />

## Setup and Create a new Google OAuth 2.0 Client ID and Secret

Head over to <https://console.developers.google.com> and sign in with your Google account

<img class="alignnone size-large wp-image-174" src="https://joeraio.com/wp-content/uploads/2016/03/05-google-developer-console-home-1024x726.png" alt="05-google-developer-console-home" width="1024" height="726" srcset="http://localhost/wp-content/uploads/2016/03/05-google-developer-console-home-1024x726.png 1024w, http://localhost/wp-content/uploads/2016/03/05-google-developer-console-home-300x213.png 300w, http://localhost/wp-content/uploads/2016/03/05-google-developer-console-home-768x545.png 768w, http://localhost/wp-content/uploads/2016/03/05-google-developer-console-home.png 1067w" sizes="(max-width: 1024px) 100vw, 1024px" />

In the top right click on the &#8220;Select a project&#8221; drop down and choose &#8220;Create a project&#8221;

<img class="alignnone size-full wp-image-175" src="https://joeraio.com/wp-content/uploads/2016/03/06-google-developer-console-create-project.png" alt="06-google-developer-console-create-project" width="464" height="259" srcset="http://localhost/wp-content/uploads/2016/03/06-google-developer-console-create-project.png 464w, http://localhost/wp-content/uploads/2016/03/06-google-developer-console-create-project-300x167.png 300w" sizes="(max-width: 464px) 100vw, 464px" />

In the new project dialog that pops up enter a name for your project, agree to the terms and click &#8220;create&#8221;

<img class="alignnone size-full wp-image-176" src="https://joeraio.com/wp-content/uploads/2016/03/07-google-developer-console-name-app.png" alt="07-google-developer-console-name-app" width="497" height="355" srcset="http://localhost/wp-content/uploads/2016/03/07-google-developer-console-name-app.png 497w, http://localhost/wp-content/uploads/2016/03/07-google-developer-console-name-app-300x214.png 300w" sizes="(max-width: 497px) 100vw, 497px" />

You will now be returned to your dashboard. Click on **Enable and Manage API&#8217;s**

<img class="alignnone size-full wp-image-177" src="https://joeraio.com/wp-content/uploads/2016/03/08-enable-manage-apis.png" alt="08-enable-manage-apis" width="397" height="350" srcset="http://localhost/wp-content/uploads/2016/03/08-enable-manage-apis.png 397w, http://localhost/wp-content/uploads/2016/03/08-enable-manage-apis-300x264.png 300w" sizes="(max-width: 397px) 100vw, 397px" />

In the window that pops up choose the project you just created and hit continue

<img class="alignnone size-full wp-image-178" src="https://joeraio.com/wp-content/uploads/2016/03/09-google-select-project.png" alt="09-google-select-project" width="502" height="208" srcset="http://localhost/wp-content/uploads/2016/03/09-google-select-project.png 502w, http://localhost/wp-content/uploads/2016/03/09-google-select-project-300x124.png 300w" sizes="(max-width: 502px) 100vw, 502px" />

On the next page scroll down and click on Google+ API

<img class="alignnone size-full wp-image-179" src="https://joeraio.com/wp-content/uploads/2016/03/10-select-google-plus-api.png" alt="10-select-google-plus-api" width="577" height="396" srcset="http://localhost/wp-content/uploads/2016/03/10-select-google-plus-api.png 577w, http://localhost/wp-content/uploads/2016/03/10-select-google-plus-api-300x206.png 300w" sizes="(max-width: 577px) 100vw, 577px" />

On the next screens click Enable and then Go to Credentials

<img class="alignnone size-full wp-image-180" src="https://joeraio.com/wp-content/uploads/2016/03/11-select-enable.png" alt="11-select-enable" width="478" height="218" srcset="http://localhost/wp-content/uploads/2016/03/11-select-enable.png 478w, http://localhost/wp-content/uploads/2016/03/11-select-enable-300x137.png 300w" sizes="(max-width: 478px) 100vw, 478px" />

<img class="alignnone size-full wp-image-181" src="https://joeraio.com/wp-content/uploads/2016/03/12-goto-credentials.png" alt="12-goto-credentials" width="791" height="257" srcset="http://localhost/wp-content/uploads/2016/03/12-goto-credentials.png 791w, http://localhost/wp-content/uploads/2016/03/12-goto-credentials-300x97.png 300w, http://localhost/wp-content/uploads/2016/03/12-goto-credentials-768x250.png 768w" sizes="(max-width: 791px) 100vw, 791px" />

On the Add Credentials screen choose Google+ API and Web server from drop downs. Choose User data under &#8220;What data will you be accessing&#8221; and click &#8220;What credentials do I need&#8221;

<img class="alignnone size-full wp-image-182" src="https://joeraio.com/wp-content/uploads/2016/03/13-add-credentials.png" alt="13-add-credentials" width="594" height="526" srcset="http://localhost/wp-content/uploads/2016/03/13-add-credentials.png 594w, http://localhost/wp-content/uploads/2016/03/13-add-credentials-300x266.png 300w" sizes="(max-width: 594px) 100vw, 594px" />

Remember how I had you save the url of your project? There is where we enter it. In the first box enter in the URL and make sure to remove the trailing / . In the second box add the same url but end it with /signin-google . Once you have done this click on Create client ID

<img class="alignnone size-full wp-image-183" src="https://joeraio.com/wp-content/uploads/2016/03/14-add-credentials-filled-out.png" alt="14-add-credentials-filled-out" width="633" height="620" srcset="http://localhost/wp-content/uploads/2016/03/14-add-credentials-filled-out.png 633w, http://localhost/wp-content/uploads/2016/03/14-add-credentials-filled-out-300x294.png 300w, http://localhost/wp-content/uploads/2016/03/14-add-credentials-filled-out-70x70.png 70w" sizes="(max-width: 633px) 100vw, 633px" />

The next screen will let you customize the Google Consent page. This is the page the users see where they have to allow you to access their data. Just enter in a product name for now and click Continue

<img class="alignnone size-full wp-image-184" src="https://joeraio.com/wp-content/uploads/2016/03/15-consent-screen.png" alt="15-consent-screen" width="642" height="472" srcset="http://localhost/wp-content/uploads/2016/03/15-consent-screen.png 642w, http://localhost/wp-content/uploads/2016/03/15-consent-screen-300x221.png 300w" sizes="(max-width: 642px) 100vw, 642px" />

You are now done! Google only shows you the Client ID right away so I will show you how to go back and display your Client ID and Client Secret. First click Done on this screen

<img class="alignnone size-full wp-image-185" src="https://joeraio.com/wp-content/uploads/2016/03/16-done-shows-clientid.png" alt="16-done-shows-clientid" width="736" height="492" srcset="http://localhost/wp-content/uploads/2016/03/16-done-shows-clientid.png 736w, http://localhost/wp-content/uploads/2016/03/16-done-shows-clientid-300x201.png 300w" sizes="(max-width: 736px) 100vw, 736px" />

The next screen shows all the credentials you have setup. Simply click on the first credential name.

<img class="alignnone size-full wp-image-186" src="https://joeraio.com/wp-content/uploads/2016/03/17-apps-listed.png" alt="17-apps-listed" width="793" height="340" srcset="http://localhost/wp-content/uploads/2016/03/17-apps-listed.png 793w, http://localhost/wp-content/uploads/2016/03/17-apps-listed-300x129.png 300w, http://localhost/wp-content/uploads/2016/03/17-apps-listed-768x329.png 768w" sizes="(max-width: 793px) 100vw, 793px" />

This screen now shows your Client ID and Client Secret. Make note of both of these because we are going to use them in your project.

<img class="alignnone size-full wp-image-187" src="https://joeraio.com/wp-content/uploads/2016/03/18-google-dev-console-shows-clientid-clientsecret.png" alt="18-google-dev-console-shows-clientid-clientsecret" width="651" height="410" srcset="http://localhost/wp-content/uploads/2016/03/18-google-dev-console-shows-clientid-clientsecret.png 651w, http://localhost/wp-content/uploads/2016/03/18-google-dev-console-shows-clientid-clientsecret-300x189.png 300w" sizes="(max-width: 651px) 100vw, 651px" />

## Setup your ASP.NET Core Application with Google Authentication

Now that we have the project setup and your Google Client ID and Secret, we simply need to modify the project to use Google Authentication.

First we need to install the Google Authentication Nuget package. To do this, let&#8217;s switch back over to Visual Studio and open up your project.json file. In the dependencies section add the following piece of code to the end.

<pre style="padding-left: 30px;" class="">"Microsoft.AspNet.Authentication.Google": "1.0.0-rc1-final"</pre>

It should look like this

<img class="alignnone size-large wp-image-188" src="https://joeraio.com/wp-content/uploads/2016/03/19-add-google-authentication-package-project-json-1024x720.png" alt="19-add-google-authentication-package-project-json" width="1024" height="720" srcset="http://localhost/wp-content/uploads/2016/03/19-add-google-authentication-package-project-json-1024x720.png 1024w, http://localhost/wp-content/uploads/2016/03/19-add-google-authentication-package-project-json-300x211.png 300w, http://localhost/wp-content/uploads/2016/03/19-add-google-authentication-package-project-json-768x540.png 768w, http://localhost/wp-content/uploads/2016/03/19-add-google-authentication-package-project-json.png 1064w" sizes="(max-width: 1024px) 100vw, 1024px" />

Notice that when you do this, Visual Studio will automatically download and install the Nuget package setting up all your reference for you.

<img class="alignnone size-full wp-image-189" src="https://joeraio.com/wp-content/uploads/2016/03/20-nuget-auto-restores-package.png" alt="20-nuget-auto-restores-package" width="986" height="228" srcset="http://localhost/wp-content/uploads/2016/03/20-nuget-auto-restores-package.png 986w, http://localhost/wp-content/uploads/2016/03/20-nuget-auto-restores-package-300x69.png 300w, http://localhost/wp-content/uploads/2016/03/20-nuget-auto-restores-package-768x178.png 768w" sizes="(max-width: 986px) 100vw, 986px" />

Now lets open up Startup.cs add add the following code. NOTE: It is important you add this code after app.UseIdentity(); and before app.UseMVC()

<pre class="" style="padding-left: 60px;">app.UseGoogleAuthentication(options =&gt;
 {
 options.ClientId = "[YOUR CLIENT ID]";
 options.ClientSecret = "[YOUR CLIENT SECRET]";
 });</pre>

It should look like the following. Of course replace the values with your Client ID and Client Secret

<img class="alignnone size-full wp-image-190" src="https://joeraio.com/wp-content/uploads/2016/03/21-edit-startup-add-google-auth.png" alt="21-edit-startup-add-google-auth" width="808" height="283" srcset="http://localhost/wp-content/uploads/2016/03/21-edit-startup-add-google-auth.png 808w, http://localhost/wp-content/uploads/2016/03/21-edit-startup-add-google-auth-300x105.png 300w, http://localhost/wp-content/uploads/2016/03/21-edit-startup-add-google-auth-768x269.png 768w" sizes="(max-width: 808px) 100vw, 808px" />

Let&#8217;s now run the project, CTRL + F5, and click login in the upper right hand corner. You should now see a page similar to the one below.

<img class="alignnone size-large wp-image-191" src="https://joeraio.com/wp-content/uploads/2016/03/23-google-auth-consent-screen-1024x726.png" alt="23-google-auth-consent-screen" width="1024" height="726" srcset="http://localhost/wp-content/uploads/2016/03/23-google-auth-consent-screen-1024x726.png 1024w, http://localhost/wp-content/uploads/2016/03/23-google-auth-consent-screen-300x213.png 300w, http://localhost/wp-content/uploads/2016/03/23-google-auth-consent-screen-768x545.png 768w, http://localhost/wp-content/uploads/2016/03/23-google-auth-consent-screen.png 1067w" sizes="(max-width: 1024px) 100vw, 1024px" />

Once the user hit&#8217;s allow they can authenticate using Google.

Technically we have done everything to get Google Authentication working. But, being I started from a brand new project I am going to continue and show you how to update the database use a migration so that the project will run properly.

## Applying a database migration using DNX

If you followed this tutorial from the very beginning and started with a new project, you will get an error after hitting &#8220;Allow&#8221; similar to the one below.

<img class="alignnone size-large wp-image-192" src="https://joeraio.com/wp-content/uploads/2016/03/24-ef-error-1024x726.png" alt="24-ef-error" width="1024" height="726" srcset="http://localhost/wp-content/uploads/2016/03/24-ef-error-1024x726.png 1024w, http://localhost/wp-content/uploads/2016/03/24-ef-error-300x213.png 300w, http://localhost/wp-content/uploads/2016/03/24-ef-error-768x545.png 768w, http://localhost/wp-content/uploads/2016/03/24-ef-error.png 1067w" sizes="(max-width: 1024px) 100vw, 1024px" />

How to we fix this? Easy, open up a command prompt in the project folder. Run the command &#8220;dnx ef database update&#8221;. This will apply the initial database migration.

<img class="alignnone size-full wp-image-193" src="https://joeraio.com/wp-content/uploads/2016/03/26-dnx-ef-update.png" alt="26-dnx-ef-update" width="967" height="183" srcset="http://localhost/wp-content/uploads/2016/03/26-dnx-ef-update.png 967w, http://localhost/wp-content/uploads/2016/03/26-dnx-ef-update-300x57.png 300w, http://localhost/wp-content/uploads/2016/03/26-dnx-ef-update-768x145.png 768w" sizes="(max-width: 967px) 100vw, 967px" />

Once you have done that, refresh your page and you are good to go! It should ask you to finish setting up your account and then log you into the site.

<img class="alignnone size-large wp-image-194" src="https://joeraio.com/wp-content/uploads/2016/03/27-authentication-success-1024x726.png" alt="27-authentication-success" width="1024" height="726" srcset="http://localhost/wp-content/uploads/2016/03/27-authentication-success-1024x726.png 1024w, http://localhost/wp-content/uploads/2016/03/27-authentication-success-300x213.png 300w, http://localhost/wp-content/uploads/2016/03/27-authentication-success-768x545.png 768w, http://localhost/wp-content/uploads/2016/03/27-authentication-success.png 1067w" sizes="(max-width: 1024px) 100vw, 1024px" />

<img class="alignnone size-large wp-image-195" src="https://joeraio.com/wp-content/uploads/2016/03/28-success-google-auth-works-asp-core-1024x726.png" alt="28-success-google-auth-works-asp-core" width="1024" height="726" srcset="http://localhost/wp-content/uploads/2016/03/28-success-google-auth-works-asp-core-1024x726.png 1024w, http://localhost/wp-content/uploads/2016/03/28-success-google-auth-works-asp-core-300x213.png 300w, http://localhost/wp-content/uploads/2016/03/28-success-google-auth-works-asp-core-768x545.png 768w, http://localhost/wp-content/uploads/2016/03/28-success-google-auth-works-asp-core.png 1067w" sizes="(max-width: 1024px) 100vw, 1024px" />

I hope you found this tutorial helpful! Happy Coding!