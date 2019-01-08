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
![Using Google Authentication with ASP.NET Core (ASP.NET 5)](/assets/wp-content/uploads/2016/03/asp-net-core-google-authentication-mvc-ef.png)

## **\*\* UPDATE &#8211; PLEASE READ \*\***

Please visit: <https://docs.microsoft.com/en-us/aspnet/core/security/authentication/social/google-logins> for updated instructions on how to use Google Authentication with ASP.NET Core.

The instructions in this article are out of date and I have not yet had a chance to update it.

<iframe src="https://channel9.msdn.com/Blogs/raw-tech/Google-Authentication-with-ASPNET-Core-ASPNET-5/player" width="640" height="360" allowFullScreen frameBorder="0"></iframe>

While speaking with some local developers about ASP.NET Core I was asked if integrating Google Authentication was just as easy in ASP.NET Core, Core MVC (MVC6) as it was in the past. I honestly had not tested it yet so I decided to give it a shot. Thankfully, its very simple, but slightly different, so I&#8217;ve included detailed instructions below.

Note; this assumes you have Visual Studio 2015 Update 1 and ASP.NET 5 RC1 Installed. If not you can get both from the following links:

* [Visual Studio Community](https://www.visualstudio.com/)
* [ASP.net 5 Download](https://get.asp.net/)

## Create a new ASP.NET 5 MVC Project

Go to File -> New Project

![Go to File -> New Project](/assets/wp-content/uploads/2016/03/1-vs-2015-new-project.png)

Select ASP.net Web Application. Name your App and hit Ok

![Select ASP.net Web Application](/assets/wp-content/uploads/2016/03/2-Web-Application-Name.png)

Select Web Application under &#8220;ASP.NET 5&#8221; Templates

![Select Web Application](/assets/wp-content/uploads/2016/03/3-select-asp-net-5-web-application.png)

Let Visual studio create the project and automatically restore the packages.

Once the solution loads, hit CTRL+F5 to run your app. Note the URL it runs on and copy this URL as we will use it later.

![Run the app and note URL](/assets/wp-content/uploads/2016/03/4-run-project-to-get-url.png)

## Setup and Create a new Google OAuth 2.0 Client ID and Secret

Head over to <https://console.developers.google.com> and sign in with your Google account

![Google Developer Console](/assets/wp-content/uploads/2016/03/05-google-developer-console-home.png)

In the top right click on the &#8220;Select a project&#8221; drop down and choose &#8220;Create a project&#8221;

![Select a project](/assets/wp-content/uploads/2016/03/06-google-developer-console-create-project.png)

In the new project dialog that pops up enter a name for your project, agree to the terms and click &#8220;create&#8221;

![Name your app](/assets/wp-content/uploads/2016/03/07-google-developer-console-name-app.png)

You will now be returned to your dashboard. Click on **Enable and Manage API&#8217;s**

![Enable manage apis](/assets/wp-content/uploads/2016/03/08-enable-manage-apis.png)

In the window that pops up choose the project you just created and hit continue

![Select and continue](/assets/wp-content/uploads/2016/03/09-google-select-project.png)

On the next page scroll down and click on Google+ API

![Google+ API](/assets/wp-content/uploads/2016/03/10-select-google-plus-api.png)

On the next screens click Enable and then Go to Credentials

![Enable credentials](/assets/wp-content/uploads/2016/03/11-select-enable.png)

![Go to credentials](/assets/wp-content/uploads/2016/03/12-goto-credentials.png)

On the Add Credentials screen choose Google+ API and Web server from drop downs. Choose User data under &#8220;What data will you be accessing&#8221; and click &#8220;What credentials do I need&#8221;

![Add credentials](/assets/wp-content/uploads/2016/03/13-add-credentials.png)

Remember how I had you save the url of your project? There is where we enter it. In the first box enter in the URL and make sure to remove the trailing / . In the second box add the same url but end it with /signin-google . Once you have done this click on Create client ID

![Credentials filled out](/assets/wp-content/uploads/2016/03/14-add-credentials-filled-out.png)

The next screen will let you customize the Google Consent page. This is the page the users see where they have to allow you to access their data. Just enter in a product name for now and click Continue

![Consent screen](/assets/wp-content/uploads/2016/03/15-consent-screen.png)

You are now done! Google only shows you the Client ID right away so I will show you how to go back and display your Client ID and Client Secret. First click Done on this screen

![Show client id](/assets/wp-content/uploads/2016/03/16-done-shows-clientid.png)

The next screen shows all the credentials you have setup. Simply click on the first credential name.

![Apps listed](/assets/wp-content/uploads/2016/03/17-apps-listed.png)

This screen now shows your Client ID and Client Secret. Make note of both of these because we are going to use them in your project.

![Show client id and client secret](/assets/wp-content/uploads/2016/03/18-google-dev-console-shows-clientid-clientsecret.png)

## Setup your ASP.NET Core Application with Google Authentication

Now that we have the project setup and your Google Client ID and Secret, we simply need to modify the project to use Google Authentication.

First we need to install the Google Authentication Nuget package. To do this, let&#8217;s switch back over to Visual Studio and open up your project.json file. In the dependencies section add the following piece of code to the end.

```
"Microsoft.AspNet.Authentication.Google": "1.0.0-rc1-final"
```

It should look like this

![Example project json with packages](/assets/wp-content/uploads/2016/03/19-add-google-authentication-package-project-json.png)

Notice that when you do this, Visual Studio will automatically download and install the Nuget package setting up all your reference for you.

![Auto restore packages](/assets/wp-content/uploads/2016/03/20-nuget-auto-restores-package.png)

Now lets open up Startup.cs add add the following code. NOTE: It is important you add this code after app.UseIdentity(); and before app.UseMVC()

```c#
app.UseGoogleAuthentication(options =>
 {
 options.ClientId = "[YOUR CLIENT ID]";
 options.ClientSecret = "[YOUR CLIENT SECRET]";
 });
```

It should look like the following. Of course replace the values with your Client ID and Client Secret

![Add google auth calues](/assets/wp-content/uploads/2016/03/21-edit-startup-add-google-auth.png)

Let&#8217;s now run the project, CTRL + F5, and click login in the upper right hand corner. You should now see a page similar to the one below.

![Google auth consent screen](/assets/wp-content/uploads/2016/03/23-google-auth-consent-screen.png)

Once the user hit&#8217;s allow they can authenticate using Google.

Technically we have done everything to get Google Authentication working. But, being I started from a brand new project I am going to continue and show you how to update the database use a migration so that the project will run properly.

## Applying a database migration using DNX

If you followed this tutorial from the very beginning and started with a new project, you will get an error after hitting &#8220;Allow&#8221; similar to the one below.

![Entity Framework error](/assets/wp-content/uploads/2016/03/24-ef-error.png)

How to we fix this? Easy, open up a command prompt in the project folder. Run the command &#8220;dnx ef database update&#8221;. This will apply the initial database migration.

![Run ef update](/assets/wp-content/uploads/2016/03/26-dnx-ef-update.png)

Once you have done that, refresh your page and you are good to go! It should ask you to finish setting up your account and then log you into the site.

![Google Authentication Success](/assets/wp-content/uploads/2016/03/27-authentication-success.png)

![Google auth working with ASP Core](/assets/wp-content/uploads/2016/03/28-success-google-auth-works-asp-core.png)

I hope you found this tutorial helpful! Happy Coding!