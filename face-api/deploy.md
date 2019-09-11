# Examining the user interface

[Prerequisite: Detecting people with Face API](./detect-face-api.md)

With our model built, we're now ready to turn our attention to deploying our site.

## Deploy the application to Azure

Now that you've seen everything in action, you can deploy your updates to Azure. By using `az webapp up` we can seamlessly redeploy our application by reissuing the command. Because we've updated the **requirements.txt** file, Azure will automatically install the newly required package.

### Performing the deployment

In a command or terminal window, execute the following command, **making sure you replace the placeholder of APP_NAME with a unique name**.

> **NOTE:** This command **must** be executed in the same directory as your code. You must also use the same name you used previously.

``` bash
az webapp up -n APP_NAME
```

> **NOTE:** We don't need to specify the resource group or location as the app is already created. The tool will redeploy to the same location as before.

### Add new application settings

We created two new environmental variables for Face API. We need to update our App Service with these two values.

1. Run the following CLI command to create an application setting named **FACE_API_KEY**, replacing **APP_NAME** with the name assigned to your App Service and **face_api_key** with your Face API key:

``` bash
az webapp config appsettings set -g contoso-travel-rg -n APP_NAME --settings FACE_API_KEY=face_api_key
```

## Run the website

Now it's time to see your site in the real world! Open your browser and navigate to **https://APP_NAME.azurewebsites.net/detect**, replacing **APP_NAME** with the name of your application. Because you were training the model on Azure, you'll be able to run the detection feature without having to retrain! And, if you open the site on your mobile device you could even take a selfie to test your site.

## Summary and next steps

We've seen how to create a facial recognition application. You could enhance the site quite a bit if you'd like. You could add links to the pages (of course). You could also list the names available, and offer someone the option to either add to an existing name or create new. Maybe allow for multiple files to be uploaded? There's a lot of room to explore!
