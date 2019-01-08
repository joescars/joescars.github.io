---
id: 295
title: Azure Functions with Multiple Output Bindings
date: 2017-03-19T21:28:45+00:00
author: Joe
excerpt: Step by step guide on creating an Azure Function with multiple output bindings.
layout: post
guid: http://joeraio.com/?p=295
permalink: /azure-functions-with-multiple-output-bindings/
price_table:
  - 'a:0:{}'
image: /assets/wp-content/uploads/2017/03/azure-functions-muleiple-output.png
categories:
  - Azure Functions
  - Microsoft Azure
  - Serverless
---
Recently while working on a partner project, it was required that we build an Azure Function that not only outputs to an Azure Storage Queue, but also archives that same message in Azure Storage. While the concept itself is simple, I wanted to use the built in Azure Functions bindings to do this as simply and cleanly as possible. After a little research, it turns out this is possible and very easy to implement.

The following walks you through how to create an Azure Function with a manual trigger that both sends a message to Azure Table Storage and Azure Queue Storage through built in Azure Function Bindings.

As with anything, there are multiple ways you can do this. I will walk you through from the very beginning starting with the easiest method, through the Azure Portal.

The following assumes you have an Azure Account. If not you [can sign up for a free trial here](https://azure.microsoft.com/en-us/free/) or you can try the Azure Functions for free by [clicking here](https://functions.azure.com/try).

_If you would like to skip the tutorial and go right to the code, you can view the repo here: [Azure Functions Multiple Output Bindings](https://github.com/joescars/AzureFunctionMultipleOutputBinding)_.

**Create a new Function App _(you can skip this step if you are using the try Azure Functions demo)_**

![Create a new Function App](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-1.png)

This will create your Azure Function App Service and a corresponding blob storage account. For the demo we will use this blob storage account but technically you could use any by utilizing the connection string.

**Browse to the new function app you created. Click on new function, choose C#, then _ManualTrigger-CSharp and click create_.**

![Browse to the function app](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-3.png)

 _Note; We are doing a manual trigger for demo purposes but you could use any of the triggers available._

Great! Now we have an Azure Function created. Lets add some output bindings. **Click on “_Integrate_”**

![Click on IntegrateC](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-4.png)

From here, we can easily add multiple output bindings through the interface.

**Click on _New Output_, choose _Azure Queue Storag_e and click _Select_**

![Click on new output](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-5.png)

**Leave all the default values and click S_ave_. We will come back to these later.**

![Leave all the default values](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-6.png)

**Again, click on _New Output_, this time choose _Azure Table Storage_, and hit _Select_.**

![Click on new output](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-7.png)

**Again, let’s leave all the default values and click _Save_.**

![Leave all the default values](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-8.png)

Behind the scenes, the portal interface has been modifying your _function.json_ file which contains all the trigger and binding information for your Azure Function. Your _function.json_ should now look like this.

![Updated function.json](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-9.png)

You can get to this by click on “_view files_” and selecting function.json.

_**function.json**_ tells the Azure Function that you now have two output binding and holds their configuration values. In this case we want to output to a table named **_outputTable_** and a storage queue names _**outqueue**_. All this using the default connection to the storage account we created.

_Note; you do not have to create the Azure Storage Queue or Table. It will automatically be created if it does not exist._ 

All we need to do now is add some code to the function to send data to those bindings.

**Let’s go back to our _run.csx_ and replace all the code with the following:**

```c#
using System;

public static void Run(string input, TraceWriter log, out string outputQueueItem, ICollector<FaceResult> outputTable)
{
    log.Info($"C# manually triggered function called with input: {input}");

    // DO SOMETHING
    // Your custom function code goes here
    // i.e. cognitive services face detection

    // Create message   
    string msg = $"Notification! 25 Faces Detected";
    log.Info(msg);

    // Store message in Azure Table Storage    
    outputTable.Add(
        new FaceResult() { 
            PartitionKey = "FaceTracking", 
            RowKey = Guid.NewGuid().ToString(), 
            Message = msg }
        );
    // Store message in Azure Storage Queue    
    outputQueueItem = msg; 

}

// Example class to storage messages regarding face detection
public class FaceResult
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Message { get; set; }
}
```

Click save and if you did everything correctly you should see _Compilation Succeeded_

![Complication Succeeded](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-10.png)

That’s it! You now have an Azure Function that can easily write to Azure Table Storage and Azure Storage Queue using built in Output bindings and very little code.

If you go ahead and run it you should get the following message. In addition, if you open the storage account for the function you will see the new table and queue created with the messages stored.

![New table and queue messages](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-11.png)

![New table and queue messages](/assets/wp-content/uploads/2017/03/azure-function-multiple-output-bindings-12.png)