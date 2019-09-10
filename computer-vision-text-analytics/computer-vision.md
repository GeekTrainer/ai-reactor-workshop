# Adding Computer Vision to our application

[Prerequisites: Create Azure resources](./create-azure-resources.md)

Like all Cognitive Services APIs, the Computer Vision API is invoked by placing calls over the Internet to a REST endpoint. Rather than placing those calls directly, you can use the [Python SDK for the Computer Vision API](https://pypi.org/project/azure-cognitiveservices-vision-computervision/) to simplify your code. The SDK abstracts calls to the Computer Vision API and handles JSON payloads for you so that you don’t have to write code to generate and consume JSON yourself.

It is the OCR feature of the Computer Vision API that will enable the Contoso Travel site to extract text from images. In this unit, you will modify Contoso Travel to use the Computer Vision API to extract text from photos uploaded to the site.

## Create .env file

As mentioned before, we are using [dotenv](https://github.com/theskumar/python-dotenv) to manage our environmental variables. Let's create the file we'll be using.

In your code editor, create a new file named **.env**, ensuring you have the leading **.** in front of the filename. **.env** is a set of key/value pairs, so we'll add the appropriate values:

``` bash
ENDPOINT=<endpoint_value>
VISION_KEY=<vision_key>
```

Replace `<endpoint_value>` and `<vision_key>` with the endpoint and key values you stored from the prior exercise. No quotes or other special characters are needed to contain the values.

## Add configuration code to app.py

You have now subscribed to the Computer Vision API and stored the endpoint and API key. The next step is to modify the Contoso Travel site to load these values.

Open **app.py**. Just below the comment which reads `# Load keys` add the following code, which will read environmental variables of `ENDPOINT`, which contains the URL of the endpoint, and `VISION_KEY`, which is the authentication key for Computer Vision:

``` python
# Load keys
endpoint = os.environ["ENDPOINT"]
vision_key = os.environ["VISION_KEY"]
```

## Create the client

We'll be using the `ComputerVisionClient` to use [Computer Vision](https://docs.microsoft.com/en-us/azure/cognitive-services/Computer-vision/Home). We'll need to load the appropriate classes, `CognitiveServiceCredentials`, which will be used to store the password, `ComputerVisionClient`, which is the actual client, and `ComputerVisionErrorException`, which represents errors raised by the service. The code finishes with the creation of the credentials, followed by the client.

``` python
# Create vision_client
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import ComputerVisionErrorException

vision_credentials = CognitiveServicesCredentials(vision_key)
vision_client = ComputerVisionClient(endpoint, vision_credentials)
```

## translate

The core Flask functionality is already provided in `translate`, which is the function you'll update to add the ability to translate signage. The content of the function is below, with additional comments than the actual file to help describe what the function is doing for you.

``` python
@app.route("/translate", methods=["GET", "POST"])
def translate():
    # Load image or placeholder
    # This uses the helper function to simplify management of images
    image = get_image(request)

    # Set the default for language translation
    target_language = "en"
    # Read the selection from the form and store it in target_language if it exists
    if request.form and "target_language" in request.form:
        target_language = request.form["target_language"]

    # If it's a GET, just return the form
    if request.method == "GET":
        # We're passing the defaults for both image and language
        return render_template("translate.html", image_uri=image.uri, target_language=target_language)

    # Create a placeholder for messages
    messages = []

    # TODO: Add code to retrieve text from picture

    # TODO: Add code to translate text

    # The user has submitted an image and language
    # We'll be growing to add OCR and translation
    # We're returning the template with the image, the language, 
    # and the appropriate text in messages
    return render_template("translate.html", image_uri=image.uri, target_language=target_language, messages=messages)
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
