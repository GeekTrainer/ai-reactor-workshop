# Examining the user interface

[Prerequisite: Detecting people with Face API](./detect-face-api.md)

With our model built, we're now ready to turn our attention to the UI. We'll explore the templates, and then see everything in action.

## Walking through the templates

The templates for both **detect.html** and **train.html** are already built for you, and are available inside the **templates** folder. Because you've already created HTML using Jinja, we wanted to focus on the new code for Face API. Let's see the important parts of each template.

### train.html

#### Displaying the result

``` html
{% if message %}
<section class="alert alert-success" role="alert">
    <h4 class="alert-header">{{message}}</h4>
</section>
{% endif %}
```

A [Bootstrap alert](https://getbootstrap.com/docs/4.0/components/alerts/) is displayed if a message is passed to our template, which will show if we've updated or created a new name.

#### Managing people

``` html
<form method=post enctype=multipart/form-data> <section class="form-group row">
    <label class="col-sm-2 col-form-label">Who is this?</label>
    <section class="col-sm-10">
        <input type="text" class="form-control" name="name" />
    </section>
    </section>
    <section class="form-group row">
        <label class="col-sm-2 col-form-label">Upload an image</label>
        <section class="col-sm-10">
            <input type="file" class="form-control" name="file" />
        </section>
    </section>
    <section class="form-group row">
        <section class="col-sm-12">
            <button type="submit" class="btn btn-success">Upload</button>
        </section>
    </section>
</form>
```

We have a form for specifying a name for the individual, and a file input for uploading images.

### detect.html

#### Displaying names

``` html
{% if names %}
<section class="alert alert-success" role="alert">
    <h4 class="alert-heading">I recognize people!</h4>
    {% for name in names %}
    <div>{{name}}</div>
    {% endfor %}
</section>
{% endif %}
```

If a list of names is sent to the template, a Bootstrap alert is created to display the messages. We loop through all of the names, and then display them inside the alert.

#### Displaying error messages

``` html
{% if message %}
<section class="alert alert-warning" role="alert">
    <h4 class="alert-heading">{{message}}</h4>
</section>
{% endif %}
```

If an error arises, it's passed in `message`. If that's the case, it'll be displayed in a warning Bootstrap alert.

#### Uploading photos

``` html
<form method=post enctype=multipart/form-data> <section class="form-group row">
    <label class="col-sm-2 col-form-label">Upload picture</label>
    <section class="col-sm-10">
        <input name="file" class="form-control" type="file" />
    </section>
    </section>
    <section class="form-group row">
        <section class="col-sm-12">
            <button type="submit" class="btn btn-success">Upload</button>
        </section>
    </section>
</form>
```

Finally, we have the form which will allow the user to upload a photo.

## Testing our application

After ensuring all the files you modified are saved you should be ready to test the page. If you stopped your website, you can restart it by opening a command or terminal window and executing `flask run`.

### Testing the training

Navigate to **http://localhost:5000/train**. You can then specify a name and upload a file. There is no dropdown list (you could add it!), so make sure the name is the same for each photo you upload of the same person. Also, ensure the photo only has one face.

### Testing the detection

After you've trained the model, you can test the detection by navigating to **http://localhost:5000/detect**. Upload a new picture and see if Face API can find the person!

## Deploy the application to Azure

Now that you've seen everything in action, you can deploy your updates to Azure. By using `az webapp up` we can seamlessly redeploy our application by reissuing the command. Because we've updated the **requirements.txt** file, Azure will automatically install the newly required package.

### Performing the deployment

In a command or terminal window, execute the following command, **making sure you replace the placeholder of APP_NAME with a unique name**.

> **NOTE:** This command **must** be executed in the same directory as your code. You must also use the same name you used previously.

``` bash
az webapp up -n APP_NAME --resource-group contoso-travel-rg --location northcentralus
```

### Add new application settings

We created two new environmental variables for Face API. We need to update our App Service with these two values.

1. Run the following CLI command to create an application setting named **FACE_API_KEY**, replacing **APP_NAME** with the name assigned to your App Service and **face_api_key** with your Face API key:

``` bash
az webapp config appsettings set -g contoso-travel-rg -n APP_NAME --settings FACE_API_KEY=face_api_key
```

2. Now use this command to create an application setting named **FACE_API_ENDPOINT** replacing **endpoint** with your Computer Vision API endpoint:

``` bash
az webapp config appsettings set -g contoso-travel-rg -n APP_NAME --settings FACE_API_ENDPOINT=endpoint
```

## Run the website

Now it's time to see your site in the real world! Open your browser and navigate to **https://APP_NAME.azurewebsites.net/detect**, replacing **APP_NAME** with the name of your application. Because you were training the model on Azure, you'll be able to run the detection feature without having to retrain! And, if you open the site on your mobile device you could even take a selfie to test your site.

## Summary and next steps

We've seen how to create a facial recognition application. You could enhance the site quite a bit if you'd like. You could add links to the pages (of course). You could also list the names available, and offer someone the option to either add to an existing name or create new. Maybe allow for multiple files to be uploaded? There's a lot of room to explore!
