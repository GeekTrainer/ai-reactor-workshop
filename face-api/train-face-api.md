# Train Face API

[Prerequisite: Introduction](./introduction.md)

In order to recognize people in an image you need to first train up your model. This is done by uploading pictures containing the face of the person you wish to be able to identify, and specifying a name. There is no pre-built UI for performing the task, but it can be completed with a couple of lines of code.

## Creating the key

As we've seen, in order to us a Cognitive Service we need to have a key. You can create an [All-in-One](https://portal.azure.com/#create/Microsoft.CognitiveServicesAllInOne) key, which will give you access to almost every Cognitive Service, or create a key for each separate service. The advantage to the latter (creating a key for each service) is there is a free pricing tier available. We're going to create a key for Face API by using the [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest).

1. In a command or terminal window, execute the following command. The endpoint will be displayed as part of the output; make sure you **log the endpoint** somewhere, as we'll need it momentarily.

``` bash
az cognitiveservices account create --resource-group contoso-travel-rg --name face-api --location northcentralus --kind Face --sku F0 --yes
```

> **NOTE:** As before, we are placing our service in northcentralus. We want to ensure all related services are placed in the same region for performance.

1. Retrieve the key you just created by using the Azure CLI. Make sure you **log this key somewhere**, as we'll be using it momentarily.

``` bash
az cognitiveservices account keys list --resource-group contoso-travel-rg --name computer-vision --query key1 --output tsv
```

## Adding the key to .env

We're using dotenv to manage our environmental variables. We now have a new key we need to store, the one for Face API.

1. At the end of **.env**, add the following line

``` bash
FACE_API_KEY=<your_face_api_key>
FACE_API_ENDPOINT=<your_endpoint>
```

Update `<your_face_api_key>` and `<your_endpoint>` with the key and URL you received from the prior step.

## Adding the package

Like any Cognitive Service, it is possible to perform Face API operations via REST calls. However, we're going to use the SDK to make things easier. We'll start by installing the SDK.

1. Open **requirements.txt** and add the following to the bottom. This is the SDK for Face API.

``` bash
azure-cognitiveservices-vision-face
```

2. Install the new package by using `pip`. Ensure you are using the [virtual environment you created previously](../computer-vision-text-analytics/installation.md). In a command or terminal window, change directories to the one containing your source code, and issue the `pip` command to install from **requirements.txt**.

``` bash
pip install -r requirements.txt
```

> **NOTE:** This step assumes you have activated the environment. If not, you can do so by calling `.\env\Scripts\activate` on Windows or `. ./env/bin/activate` on macOS or Linux (ensuring you have the leading `.`).

## Create the face_client

We're going to use `face_client` to interact with Face API. We'll start by adding the code to read the key, and then create the object.

1. Open **app.py**. Right below the line which reads `translate_key = os.environ["TRANSLATE_KEY"]`, add the following:

``` python
from azure.cognitiveservices.vision.face import FaceClient
face_api_key=os.environ["FACE_API_KEY"]
face_api_endpoint=os.environ["FACE_API_ENDPOINT"]
face_client = FaceClient(face_api_endpoint, CognitiveServicesCredentials(face_api_key))
person_group_id = 'reactor'
```

Similar to the steps performed with Computer Vision, we import the class we'll be using (`FaceClient`), read the key, and then create the client by specifying the endpoint and key. The final line of `person_group_id = 'reactor'` is the name of the Person Group we're going to be creating. Each group you create needs to be uniquely named.

## Create the train route

We're going to add two routes to our application, one for training and the other for detection. We'll start by adding the route and code to train our mode.

1. Add the following code to the end of **app.py**.

``` python
@app.route('/train', methods=['GET', 'POST'])
def train():
    if request.method == 'GET':
        return render_template('train.html')

    if 'file' not in request.files:
        return 'No file detected'

    image = request.files['file']
    name = request.form['name']

    # Helper function to ensure a Face API group is created
    people = get_people()

    # Look to see if the name already exists
    # If not create it
    operation = "Updated"
    person = next((p for p in people if p.name.lower() == name.lower()), None)
    if not person:
        operation = "Created"
        person = face_client.person_group_person.create(person_group_id, name)

    # Add the picture to the person
    face_client.person_group_person.add_face_from_stream(person_group_id, person.person_id, image)

    # Train the model
    face_client.person_group.train(person_group_id)

    # Display the page to the user
    return render_template('train.html', message="{} {}".format(operation, name))
```

### Breaking down the code

#### Check for an image

``` python
if request.method == 'GET':
    return render_template('train.html')

if 'file' not in request.files:
    return 'No file detected'
```

The basic flow for a user training an image is to first see the page (via **GET**), and then provide a name and image (via **POST**). We start by checking if the user has requested the page via **GET**, and if so returning the **train.html** page. If they have uploaded a file via **POST**, we then check to see if there is a file in the form, and if not return an error message.

#### Retrieving the people in our model

``` python
people = get_people()
```

We're calling a helper function (which we'll create in a few minutes) to both create our group if it doesn't exist and get us the list of people in the group.

#### Create the person

``` python
# Look to see if the name already exists
# If not create it
operation = "Updated"
person = next((p for p in people if p.name.lower() == name.lower()), None)
if not person:
    operation = "Created"
    person = face_client.person_group_person.create(group_id, name)
```

If the person doesn't exist, as identified by their name, we're going to create them. We start by looping through the list of people (unfortunately there is no `get` function) to find the person. If no match is found, we create the person by calling `create`.

> **NOTE:** The `next` function works similar to `coalesce` in SQL. If the first value returns nothing, then it uses the second value. This is needed when using the `for` loop to find an item, as if no item is returned Python will raise an error.

#### Upload the image and train the model

``` python
# Add the picture to the person
face_client.person_group_person.add_face_from_stream(person_group_id, person.person_id, image)

# Train the model
face_client.person_group.train(person_group_id)

# Display the page to the user
return render_template('train.html', message="{} {}".format(operation, name))
```

We finish by adding a face to the person by calling `add_face_from_stream` and sending the image the user uploaded. We then train the model by calling `train` and specifying the group. We close by returning the template to display the message confirming we created or updated the mode.

> **NOTE:** In our example we're retraining the model each time we add a new picture. While this is fine for our sample, in the real world you should consider uploading images and people in bulk and then training at the end.

## Managing the group

Let's create the `get_people` helper function to manage our group and retrieve the list of people in the group.

1. Add the following to the end of **app.py**.

``` python
def get_people():
    try:
        face_client.person_group.create(group_id, name=group_id)
    except:
        pass
    people = face_client.person_group_person.list(group_id)
    return people
```

### Breaking down the code

#### Creating the group

``` python
try:
    face_client.person_group.create(group_id, name=group_id)
except:
    pass
```

Unfortunately, `face_client` doesn't support an `exists` function or another way to simply check for the existence of a group. We can either make a call to `get`, check for an error, and then `create`, or simply attempt `create`, and ignore any errors. That's the approach we've taken here.

#### Retrieving and returning people

``` python
people = face_client.person_group_person.list(group_id)
return people
```

Each group has a collection of people. There is no ability to retrieve a single person, so instead we call `list`. We return the list to the `train` function we created earlier for use in creating or updating the person.

## Summary and next steps

We've seen how to create the necessary key for Face API, and add the appropriate code to train the model with a person. Next, we'll see how we can [detect a person in an image](./detect-face-api.md).
