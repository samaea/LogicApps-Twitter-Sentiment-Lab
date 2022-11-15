# Twitter Sentiment Lab using Azure Serverless technologies and PowerBI
## Introduction

In this lab, we will explore how you can use Azure Serviceless technologies to pull together a solution relatively quickly. In this scenario, the solution will be a Social Dashboard. We will use Logic Apps to monitor for specific hashtags in Twitter, obtain the overall sentiment and perform key phrase extraction using Azure Cognitive Services, and then perform some transformations using Azure Functions. Finally, we will then insert the results into PowerBI to visualise the insights on a map.


## Prerequisites

1. An Azure account and subscription.
2. A Twitter account.
3. A PowerBI account.

## Deploying the Logic App
  1. Navigate to the Azure Portal:- https://portal.azure.com
     
  1. Click on **+ Create a resource** in the top left handside of the screen.
     

  1. Under Logic App, click on **Create**. Alternatively, you can search Logic Apps in the search bar. Once the wizard opens up, click on **Create** again. Input the following values when asked:

      * **Subscription**: Select your Azure subscription.
      * **Resource Group**: TwitterDashboard-**YOURALIAS**
        * Replace **YOURALIAS** with your own alias or any word that is unique. Please do this throughout the lab wherever you see this reference.
      * **Logic App name**: TwitterDashboard-**YOURALIAS**
      * **Region**: North Europe.
      * **Plan type**: Consumption.

        ![Azure Portal Logic App Creation page](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/azure-portal-create-logic-apps.png)

  1. Click on **Review + create** and then **Create**. Once deployment is complete, click on **Go to Resource**.

## Creating your Logic App workflow
  1. Click on **Blank Logic App +**

     >Now we are in the Logic Apps Designer, we can start to pull together our workflow. Let's go ahead and add a trigger for Twitter.
### Creating your Logic App Trigger 
  1. Search for **Twitter** in the searhbox and select **When a new tweet is posted**.
     * **Connection Name**: TwitterConnection
     * **Authentication Type**: Leave as default (Use Default shared application).

  1. Click on **Sign In**.
     > A Twitter login popup should appear. Input your Twitter credentials to give your Logic Apps access into Twitter.

     ![Logic Apps Designer - Establishing connection to Twitter](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-designer-twitter-conn.png)
 
  1. Now we can define what keywords we want to search for in Twitter followed by the frequency. 
     * **Search text**: #Azure
     * **How often do you want to check for items**: 15 seconds.

     > **WARNING**: 15 seconds can be quite excessive, but the goal here is to gain as much data from Twitter in a short time. Therefore, it is advised you disable the Logic Apps or set a much higher interval to avoid unexpected Azure charges.

  1. On the top left hand side, click on **Save** to save the changes you've made so far.  
      >**NOTICE**: You may lose all the changes made in the workflow if the browser's window times out or you navigate elsewhere. Thus, it's important to frequently save your changes as you work through the workflow.

  1. When you save the workflow, it also activates the trigger. Given it's very early to be gathering data at this stage, we will temporarily disable the Logic App. **Open a new browser tab** and **navigate to** https://portal.azure.com/#view/HubsExtension/BrowseResource/resourceType/Microsoft.Logic%2Fworkflows. Select the Logic App you just deployed (i.e TwitterDashboard-**ALIAS**). Click on **Disable** in the top center of the screen.

        ![Logic Apps - Disabling the Logic App](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-disable.png)

  1. Navigate back to the browser tab that has the Logic Apps Designer opened.

### Adding Cognitive Services into the workflow

In this stage, we will add Cognitive Services as part of our Logic Apps workflow to analyse the sentiment of the incoming tweets (from the Twitter Trigger) and perform keyphrase extraction on them.

  1. Click on **+ New step**
  1. Search for the connector **Cognitive Services** and click on it.

     ![Logic Apps Designer - Adding Cognitive Services](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-cog-services.png)

  1. Under **Actions**, select **Sentiment (V3.0)** 
  
       ![Logic Apps Designer - Adding Cognitive Services](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-cog-services-sentiment.png)


  1. Notice how it is asking for an Account Key and Site URL for our Cognitive Services resource. Given we haven't deployed one yet, let's go ahead and do this now. In a new browser tab, navigate to https://portal.azure.com/#create/Microsoft.CognitiveServicesAllInOne and use the below inputs:-
     * **Subscription**: Use the same subscription you used earlier.
     * **Resource group**: Use the same resource group used earlier (i.e TwitterDashboard-**YOURALIAS**).
     * **Region**: Use the same region as earlier (in our case that was **northeurope**).
     * **Name**: TwitterDashboard-**YOURALIAS**.
     * **Pricing tier**: Standard S0.


       ![Azure Portal - Creating Cognitive Services resource](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/azure-portal-create-cognitive-services.png)

  1. Click on **Review + create** followed by **Create**. Once it has been deployed, click on **Go to resource**

  1. Select **Keys and Endpoint** and copy **Key 1** and **Endpoint** into Notepad.

     ![Azure Portal - Obtaining Cognitive Services key and url](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/cognitive-services-keys.png)

  1. Now, let's switch back to the browser tab that has the Logic Apps Designer opened.
     * **Connection name**: Can be anything you can want e.g. CongitiveServicesConnection.
     *  **Account Key**: Input the value of **Key 1** that you obtained earlier.
     *  **Site URL**: Input the value of **Endpoint** from earlier.

        ![Azure Portal - Creating Cognitive Services resource](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-cog-services-creds.png)


  1. Click **Create**
     * **documents id -1**: When you click on this field, a Dynamic content window will popup. Select **Tweet id**
     * **documents text - 1**: Similary, select **Tweet text**
       ![Logic Apps - Selecting parameters from documents id and text](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-cog-services-params.png)

  1. Click on **+ New Step**

  1. Search for the connector **Cognitive Services** and click on it.

  1.  Under **Actions**, select **Keyphrases (V3.0)** 

  1. Simiarly to what we did with configuring Sentiment (V3.0):
     * **documents id -1**: Tweet Id
     * **documents text - 1**: Tweet text
  1. Click **Save** on the top left corner.

     ![Logic Apps - Selecting Keyphrases parameters from documents id and text](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-cog-services-keyphrases-params.png)

  ### Adding an Azure Functions Services into the workflow

  > NOTE:- The below is just informational, avoid running the workflow at this stage as we want to pipe as much information from Twitter to PowerBI in the short amount of time we will be running it.

  If we ran our workflow, we would realise the Keyphrases Connector will output something similar to the below. Notice how under documents it has returned a JSON array called keyPhrases with the value **["new enhancements","RT","dotnet"]**:-

  ![Logic Apps - Selecting Keyphrases parameters from documents id and text](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/cognitive-services-keyphrases-output.png)

  We want to make the above output look prettier by converting the JSON array into a more readible string using Azure Functions.

  1. Let's create and deploy an Azure function. Open a new browser tab and navigate to https://portal.azure.com/#create/Microsoft.FunctionApp.
     * **Subscription**: Use the same subscription you used earlier.
     * **Resource group**: Use the same resource group used earlier (i.e TwitterDashboard-**YOURALIAS**).
     * **Function App Name**: TwitterDashboard-**YOURALIAS**.
     * **Runtime stack**: .NET
     * **Version**: 6.
     * **Region**: Use the same region as earlier (in our case that was **northeurope**).
     * **Plan Type**: Consumption (Serverless).

       ![Azure Portal - Creating a Function App](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/azure-portal-create-function-app.png)

  1. Click **Review + Create** and then **Create**. Once deployment is complete, click on **Go to resoure**. This should now open the Function App.

  1. Select **Functions** and then click on **Create**.
  1. Select **HTTP Trigger** and then click on **Create**.
  1. Click on **Code + Test**.
  1. You will notice sample code in there which Azure Functions generated by default. Replace all the code with the following:-

      ``` csharp
      #r "Newtonsoft.Json"
      using System.Net;
      using Microsoft.AspNetCore.Mvc;
      using Microsoft.Extensions.Primitives;
      using Newtonsoft.Json;
      using Newtonsoft.Json.Linq;
      public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
      {
          log.LogInformation("C# HTTP trigger function processed a request.");
          // Get the phrases array
          var phrases = await new StreamReader(req.Body).ReadToEndAsync();
          dynamic jsonPhrases = JsonConvert.DeserializeObject(phrases);
          // Join the array with a comma
          var joinedPhrases = string.Join(", ", jsonPhrases);
          // Return the response
          string responseMessage = string.IsNullOrEmpty(phrases)
              ? "This HTTP triggered function executed successfully. Pass an array of words in the query string or in the request body in order to sanity the results."
              : joinedPhrases;
          return new OkObjectResult(responseMessage);
        }
        ```

  1. Click **Save**. Now it's time to test whether our piece of code works. As mentioned earlier, we will use Azure Functions to transform a JSON array into a more readable string.

  1. Click on **Test/Run**. Under **Body** replace all the text with 

     ```
     ["new enhancements","RT","dotnet"]
     ```

  1. Click **Run**. The Function should transform and output it to:-

     ```
     new enhancements, RT, dotnet
     ```
       ![Azure Portal - Azure Functions output](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/functions-output.png)

  1. If you received a similar output to the above, it is now time to switch back to the browser tab that has Logic Apps Designer opened.

  1. Now, let's add Azure Functions in the Logic Apps Designer, click on **+New Step**.

  1. Search for the connector **Azure Functions** and select **Choose an Azure Function**. Select the Azure Function we created earlier called TwitterDashboard-**YOURALIAS**

     ![Logic Apps - Selecting an Azure Function](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-select-function.png)

  1. Select **HttpTrigger1**
  1. Select the field box next to **Request Body**. The Dynamic content popup should appear. Click on **Expression** and input the following:-
     ```
     body('Key_Phrases_(V3.0)')['documents'][0]['keyPhrases']
     ```
     >NOTE: To understand the above expression, it is basically referencing the output of Key Phrases (V3.0). The value we are interested (the key phrases output) resides in this path. You can figure out the path yourself through the example earlier, which you can find here https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/cognitive-services-keyphrases-output.png
    
  1. Select **OK** to save the expression.

     ![Logic Apps - Selecting an Azure Function](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-function-expression.png)
 
   1. Click on **Save** on the top left corner.

  ### Adding PowerBI into the workflow

  >Now it's time to send the data into PowerBI to visual the sentiment of our tweets. Before we can do this, we will need to create a PowerBI dataset.
 
  1. In a new browser tab, navigate to https://www.powerbi.com/groups/me/create.
  1. Click on **Create** located on the top left corner.
  1. Select **Streaming dataset** and then **API**

     ![PowerBI - Creating a streaming dataset](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/powerbi-create-streaming-dataset.png)

  1. Input the following values in their respective fields:
     * **Dataset name**: TwitterDashboard-**YOURALIAS**
     * Values to input under **Values from stream**
       Value | Type
       --------- | -----------
       `Twitter Text` | text
       `Sentiment` | text
       `Key Phrases` | text
       `Location` | text
      * **Turn on** the checkbox for **Historic data analysis**
  1. Click **Create**

       ![PowerBI - Creating a streaming dataset](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/powerbi-create-streaming-dataset-2.png)

  1. Now click on **Create**

  1. With our streaming dataset now created, let's navigate back to the browser tab that has the Logic Apps Designer opened.
  1. Click on **+ New step**
  1. Search for the connector **PowerBI** and then select the connector **PowerBI**. 
  1. Select the action **Add rows to a dataset**. Click on **Sign in** and use the same credentials that you used to create the Power BI streaming dataset with.
  1. Next to **Workspace** select **My Workspace**
       * For **Dataset** select TwitterDashboard-**YOURALIAS**
       * For **Table** select **RealTimeData**
  1. Select **Add new parameter** and **tick the checkbox** for all the fields we created (i.e Twitter Text, Sentiment, Key Phrases and Location). This provide additional fields to fill in.
     * **Twitter Text**: Select the field box and select **Tweet Text**
     * **Sentiment**: Select the field box, select **Expression** 
       ```
       body('Sentiment_(V3.0)')['documents'][0]['sentiment']
       ```
     * **Key Phrases**: Select the field box, search for **Body** and select **Body** under HttpTrigger1 (i.e the output of our Azure Functions)
     * **Location**: Select the field box and select **Location**

     * It should now look like this:
       ![PowerBI - Creating a streaming dataset](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-powerbi.png)

     * Click **Save** on the top left corner.

  1. **Open a new browser tab** and navigate to https://portal.azure.com/#view/HubsExtension/BrowseResource/resourceType/Microsoft.Logic%2Fworkflows. Select the your Logic App (i.e TwitterDashboard-**ALIAS**). Click on **Enable** in the top center of the screen. Feel free to also click **Run Trigger** to manually start the Logic App.

  1. If everything has been configured correctly, you should start to see successful executions of your Logic App workflow as shown below:

     ![Logic Apps - Successful executions of the workflow](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/logic-apps-enable.png)

  ## Creating a report in PowerBI to visual the data on a map

  1. **Navigate to** https://www.powerbi.com/groups/me/list.
  1. Click on **My workspace**.
  1. Locate the PowerBI dataset you created in earlier steps (
TwitterDashboard-**ALIAS**), click on the **ellipsis (\*\*\*)** icon and then on **Create Report**
     ![PowerBI- Create Report from dataset](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/powerbi-myworkspace.png)
  1. In the report, select the **Map icon**. Feel free to expand the size of the map.
       ![PowerBI - Map Icon](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/powerbi-map-icon.png)
  1. On the top right hand side, **expand out** the RealTimeData table.
  1. **Drag and drop** the **Location** field over to **Location** under the parameters of the map.
  1. **Drag and drop** the **Sentiment** field over to **Bubble size**.
  1. Click on the **paint brush icon** under **Build visual**

     ![PowerBI - Map Configuration](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/powerbi-map.png)
  1. Expand **Colors** and for **negative**, select the colour red, **neutral** yellow and **positive** green.

     ![PowerBI - Map Configuration](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/powerbi-map-2.png)

  1. And if you are interested in viewing the tweets and key phrases: Select a blank area next to the map. Then, select the **two dotted squares** under **Format visual**, click on the **Table icon** and select all the fields under **RealTimeData**.

       ![PowerBI - Table Configuration](https://raw.githubusercontent.com/samaea/LogicApps-Twitter-Sentiment-Lab/main/images/powerbi-table.png)


  ## Summary
  
  > **Warning**
  > Do not forget to **disable** or **delete** the Logic App by the end of the lab to avoid unexpected Azure charges!

  Congratulations, you've now reached the end of the lab! In summary, you've built a Logic Apps that pulls messages from Twitter by searching for hastag #Azure. Afterwards, you performed sentiment analysis and key phrase extraction with the help of Azure Cognitive Services. You then used Azure Functions to transform the output of key phrase extraction to be more readable. Lastly, you past Azure Function's output along with the Twitter's and sentiment analysis output into a PowerBI dataset where then created a report to visualise the insights of the data. Hopefully this lab showed you the power of low code solutions to build a solution, which otherwise would have taken many days to achieve if you built this from the ground up using a pro-code approach.
