---
title: Securing Azure Functions with Azure Active Directory
meta-description: This week marks four years that I have been working at Microsoft. It’s been an incredibly fun & humbling journey so far! In thinking about the past four years...
excerpt: This week marks four years that I have been working at Microsoft. It’s been an incredibly fun & humbling journey so far! In thinking about the past four years...
#date: 2018-09-10T14:51:54+00:00
author: Joe
layout: post
permalink: /azure-functions-azure-active-directory/
categories:
  - Microsoft
  - Microsoft Culture
  - Working at Microsoft
  - Career Advice
  - Mentorship
tags: [microsoft, working at microsoft, career advice, culture, mentorship]
bigimg: /assets/post-content/joe-microsoft-sign-1.jpg
share-img: /assets/post-content/joe-microsoft-sign-1.jpg
---
In this example we are trying to authenticate a using a client (Postman) allowing it to access Azure Function.

The steps we are going to reproduce are:

1. Creating and configuring Azure Active Directory applications
2. Go through authentication process to accesstoken
3. Provide access toekn as Bearer authorization allowing you to access restricted function app.

Please also take a look at a [Reference Solution from Sahil Malik](https://github.com/Azure-Samples/ms-identity-dotnet-webapi-azurefunctions)

### Create a new app registration for the function/api

- Azure Active Directory -> App Registrations -> New Registration
- Enter a name
- For this example use http://localhost:7071 as the redirect URI

![Create new app registration](/assets/post-content/azaad/01-register-app.png)

- Expose an API -> Application ID URL Set
  - The actual url does not matter but I use this format: https://**tenantid**.onmicrosoft.com/functionapi
  - So for me its https://jmrcloud.onmicrosoft.com/functionapi
- Click Add Scope
- Enter in MyScope for the name, leave defaults and Add Scope.

Once completed it should look like this:

![Expose an API](/assets/post-content/azaad/02-expose-api.png)

### Create and configure client app registration

In our case, the client is Postman but will be a reactjs app in the future.

#### Create

- Azure Active Directory -> App Registrations -> New Registration
- Enter a name
- We can leave the redirect URI blank

#### Configure Authentication

- From within the App Registration page, click on **Authentication**
- Click on **Add a Platform** , Select Web
- Enter in http://localhost:7071 (again, for example purposes only)
- Check **ID tokens** under implicit grant, Click configure

#### Generate Secret

- Click on Certificates & Secret, New Client Secret
- Generate a new secret and copy it somewhere safe. We will use it later

## Set API Permissions

- Click on API Permissions -> **Add a permission** -> My APIs
- You should see your function api scope that was created earlier.
- Select it and check the box next to the scope you created
- Click **Add Permissions**
- Click **Gran admin consent for [Tentant Name]**

![API Permissions](/assets/post-content/azaad/03-api-permissions.png)

### Configure Function App 

Using the values from the **Function App API** app registration lets configure the function app. 

Open up Constants.cs and update the following:

```C#
internal static string audience = "https://<tenantname>.onmicrosoft.com/url"; // Get this value from the expose an api
internal static string clientID = "<appid>"; // this is the client id, also known as AppID. This is not the ObjectID
internal static string tenant = "<tenantname>.onmicrosoft.com"; // this is your tenant name
internal static string tenantid = "<tenantid>"; // this is your tenant id (GUID)
```

Buld and run the application and leave it running int he background.

### Authenticate via Browswer to Obtain Token

- Construct the following url and paste it into your browser. 
- Replace items noted with your own values from the **Client App Registration**

```
https://login.microsoftonline.com/{TENANT NAME}.onmicrosoft.com/oauth2/v2.0/authorize?response_type=code&client_id={CLIENT ID}&redirect_uri={REDIRECT URL}&scope=openid {APPLICATION ID URI FROM API}
```

- Paste that into a browser
- It will authenticate
