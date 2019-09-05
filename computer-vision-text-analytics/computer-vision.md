# Adding Computer Vision to our application

[Prerequisites: Create Azure resources](./create-azure-resources.md)

Like all Cognitive Services APIs, the Computer Vision API is invoked by placing calls over the Internet to a REST endpoint. Rather than placing those calls directly, you can use the [Python SDK for the Computer Vision API](https://pypi.org/project/azure-cognitiveservices-vision-computervision/) to simplify your code. The SDK abstracts calls to the Computer Vision API and handles JSON payloads for you so that you don’t have to write code to generate and consume JSON yourself.

It is the OCR feature of the Computer Vision API that will enable the Contoso Travel site to extract text from images. In this unit, you will modify Contoso Travel to use the Computer Vision API to extract text from photos uploaded to the site.

## Add configuration code to app.py

You have now subscribed to the Computer Vision API and obtained an endpoint and an API key for calling it. The next step is to modify the Contoso Travel site to use these values to call the Computer Vision API and extract text from photos. We'll start by adding the code to load the appropriate SDKs and system variables.

Open **app.py**. **Replace** everything **above** the line which says `@app.route("/", methods=["GET", "POST"])` with the following:

``` python
import os, base64
from flask import Flask, render_template, request, flash

# Import Cognitive Services SDK components
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import ComputerVisionErrorException
from msrest.authentication import CognitiveServicesCredentials

# Configure system variables with dotenv
from dotenv import load_dotenv
load_dotenv()

# Create a ComputerVisionClient for calling the Computer Vision API
endpoint = os.environ["ENDPOINT"]
vision_key = os.environ["VISION_KEY"]
vision_credentials = CognitiveServicesCredentials(vision_key)
vision_client = ComputerVisionClient(endpoint, vision_credentials)

app = Flask(__name__)
app.secret_key = os.urandom(24)
```

### Breaking down the code

#### Importing libraries

``` python
import os, base64
from flask import Flask, render_template, request, flash
```

The first two lines of code import the libraries we'll be using for Flask and working with images.

``` python
# Import Cognitive Services SDK components
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import ComputerVisionErrorException
from msrest.authentication import CognitiveServicesCredentials
```

We then import the classes needed for calling the Computer Vision SDK. Specifically, `ComputerVisionClient`, which will be used to call the REST endpoints, `ComputerVisionErrorException`, which will be used to detect errors, and `CognitiveServicesCredentials`, which will be used to send our API key and endpoint name.

#### Using dotenv

``` python
# Configure system variables with dotenv
from dotenv import load_dotenv
load_dotenv()
```

[python-dotenv](https://github.com/theskumar/python-dotenv) is used to load environmental variables from a text file as opposed to setting them in the local OS or in code. To use **dotenv**, you add a file named **.env** to the root of your application. You then store the keys and values in this file. `load_dotenv()` then opens this file and sets the environmental variables for the application. When the code runs on production it will instead read the environmental variables.

> **NOTE:** Make sure you add **.env** to your **.gitignore** file to avoid publishing keys to GitHub or other public repositories.

> **NOTE:** dotenv is perfect for scenarios such as this where security isn't tight but you still want to avoid publishing keys publicly. For applications where keys are sensitive you may need to investigate other solutions.

#### Creating the client

``` python
# Create a ComputerVisionClient for calling the Computer Vision API
endpoint = os.environ["ENDPOINT"]
vision_key = os.environ["VISION_KEY"]
vision_credentials = CognitiveServicesCredentials(vision_key)
vision_client = ComputerVisionClient(endpoint, vision_credentials)
```

We then load `VISION_KEY` and `ENDPOINT` from our environmental variables. We then create an instance of `CognitiveServicesCredentials`, which we will use to tell the client where to locate the REST endpoint and the key to use. Finally, we create `vision_client`, an instance of `ComputerVisionClient`, which will be used to make the calls to detect characters in the image.

#### Enabling FlashMessage

``` python
app = Flask(__name__)
app.secret_key = os.urandom(24)
```

We used this code previously to setup FlashMessage.

## Update index to send the message to Computer Vision

Now that we have everything configured, let's update our code to send the image to Computer Vision. Replace the `index` function, which is **after** the line which reads `app.secret_key = os.random(24)` (which should be on line 20).

``` python
@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        # Display the image that was uploaded
        image = request.files["file"]
        uri = "data:image/jpg;base64," + base64.b64encode(image.read()).decode("utf-8")
        image.seek(0)

        # Use the Computer Vision API to extract text from the image
        lines = extract_text_from_image(image, vision_client)

        # Flash the extracted text
        for line in lines:
            flash(line)

    else:
        # Display a placeholder image
        uri = "/static/placeholder.png"

    return render_template("index.html", image_uri=uri)
```

### Breaking down the code

Let's talk about the new lines of code setting the `uri` variable (which should be after line 27).

#### Reset the image stream

``` python
        image.seek(0)

        # Use the Computer Vision API to extract text from the image
        lines = extract_text_from_image(image, vision_client)
```

We start by resetting the binary stream to the beginning, which will allow us to upload the picture to Computer Vision for analysis. We then call a helper function (which we'll see in the next section) which will return all lines of text detected in the image and store it in a variable named `lines`.

#### Reading the result

``` python
        # Flash the extracted text
        for line in lines:
            flash(line)
```

We'll read each line, and add them to FlashMessages for use in our template.

## Calling Computer Vision

Let's create the helper function which will send the image to Computer Vision and prepare the results. Add the following function to the end of **app.py**.

``` python
# Function that extracts text from images
def extract_text_from_image(image, client):
    try:
        result = client.recognize_printed_text_in_stream(image=image)

        lines=[]
        if len(result.regions) == 0:
            lines.append("Photo contains no text to translate")

        else:
            for line in result.regions[0].lines:
                text = " ".join([word.text for word in line.words])
                lines.append(text)

        return lines

    except ComputerVisionErrorException as e:
        return ["Computer Vision API error: " + e.message]

    except:
        return ["Error calling the Computer Vision API"]
```

### Breaking down the code

#### Calling Optical Character Recognition

``` python
    try:
        result = client.recognize_printed_text_in_stream(image=image)
```

Using the Computer Vision client, we call `recognize_printed_text_in_stream` to retrieve the words found in the image from Compuer Vision.

#### Preparing the result

``` python
        lines=[]
        if len(result.regions) == 0:
            lines.append("Photo contains no text to translate")

        else:
            for line in result.regions[0].lines:
                text = " ".join([word.text for word in line.words])
                lines.append(text)

        return lines
```

The result we receive from Computer Vision is in JSON. We are going to convert it to an `list` of lines of text for simplicity. We start by creating an `list` to store the lines. We then examine `result.regions` to see if there are any regions on the image which contain text. An image could have one or more regions. If no regions are returned there is no text, so we append an error message.

Assuming there are regions, we are going to retrieve the lines from the first region. `regions` is a list, so you could create another `for` loop to iterate through all of the regions if you desire. In our case we just use the first. We then grab all of the `words` in each `line` and add a space between them by using `join`. Finally, we append the `line` to our `lines` list and return it.

#### Basic error handling

``` python
    except ComputerVisionErrorException as e:
        return ["Computer Vision API error: " + e.message]

    except:
        return ["Error calling the Computer Vision API"]
```

We finish off by returning whatever error messages were raised.

## Configuring environmental variables with dotenv

We are using **dotenv** to manage our environmental variables. For development, we'll use a **.env** file, which will contain key/value pairs.

### Creating the .env file

In the root of your project directory, create a new file named **.env**. This will be a plain text file. Add the following to the file:

``` bash
ENDPOINT=<endpoint_from_prior_exercise>
VISION_KEY=<vision_key_from_prior_exercise>
```

Previously you [created keys](./create-azure-keys.md), and were asked to store the key and endpoint values. Now it's time to use them! You'll replace the placeholders of `<vision_key_from_prior_exercise>` and `<endpoint_from_prior_exercise>` respectively.

## Updating index.html to display the results

We need to update **index.html** to display the results of the text display. Just before the `<script>` element at the bottom of **index.html**, add the following markup:

``` html
<div class="container">
    <div class="row">
        <div id="myModal" class="modal fade" role="dialog">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <button type="button" class="close" data-dismiss="modal">&times;</button>
                        <h4 class="modal-title">Result</h4>
                    </div>
                    <div class="modal-body"></div>
                </div>
            </div>
        </div>
    </div>
</div>

{% with messages = get_flashed_messages() %}
    {% if messages %}
        <script type="text/javascript">
            // If flash messages are queued up, show them in a modal dialog
            var messages = {{ messages | safe }};
            body = $(".modal-body");
            body.empty();

            for (i=0; i<messages.length; i++) {
                body.append("<h2>" +  messages[i] + "</h2>");
            }

            $("#myModal").modal("show");
        </script>
    {% endif %}
{% endwith %} 
```

### Breaking down the markup

#### Creating a modal dialog

``` html
<div class="container"> 
    <div class="row">
        <div id="myModal" class="modal fade" role="dialog">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <button type="button" class="close" data-dismiss="modal">&times;</button>
                        <h4 class="modal-title">Result</h4>
                    </div>
                    <div class="modal-body"></div>
                </div>
            </div>
        </div>
    </div>
</div>
```

The first section of markup is [boilerplate code](https://en.wikipedia.org/wiki/Boilerplate_code) from [Bootstrap](https://getbootstrap.com) for [modal dialog boxes](https://getbootstrap.com/docs/4.3/components/modal/).

#### Adding the translated lines

``` html
{% with messages = get_flashed_messages() %}
    {% if messages %}
        <script type="text/javascript">
            // If flash messages are queued up, show them in a modal dialog
            var messages = {{ messages | safe }};
            body = $(".modal-body");
            body.empty();

            for (i=0; i<messages.length; i++) {
                body.append("<h2>" +  messages[i] + "</h2>");
            }

            $("#myModal").modal("show");
        </script>
    {% endif %}
{% endwith %}
```

We start by retrieving the messages from `get_flashed_messages`. We use the `with` keyword to instruct Python to dispose of the object when we're done using it. Assuming messages exist, we set them to the JavaScript variable `messages`. The syntax `{{ messages | safe }}` passes `messages` through [safe](https://jinja.palletsprojects.com/en/2.10.x/templates/#safe), a [Jinja filter](https://jinja.palletsprojects.com/en/2.10.x/templates/#filters) which will ensure we don't have someone injecting HTML or scripting into our page.

We then use jQuery to retrieve the `.modal-body`, and clear it out by calling `empty`, and then `append` each translated line. We finish by calling `modal("show")` to display the modal dialog.

## Testing our site

Make sure you've saved all of the files you created or modified. Let's see if the page works.

Navigate to **http://localhost:5000** in your browser. Click **Upload Photo** and upload a picture which contains text. Confirm that after a brief pause the extracted text from the photo appears in the modal dialog box.

> **NOTE:** The image must be smaller than 4MB. If you receive an error message indicating your key or endpoint are invalid (or None), try restarting the application manually.

## Summary and next steps

You'll probably notice the text recognition isn't perfect. Depending on the clarity of the photo, the fonts, etc., there may be some small differences between actual text and what is detected. But you'll notice a high degree of success. Now we can take the [text and translate it](text-analytics.md).
