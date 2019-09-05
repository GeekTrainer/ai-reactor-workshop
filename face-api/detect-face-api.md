# Detect a person using Face API

[Prerequisite: Train a model with Face API](./train-face-api.md)

With a model trained, we're now able to detect faces. When working with any AI (or ML) solution, the answers are never perfect. In the case of Face API, a confidence score will be returned with the execution, which we can then use to indicate to the user how sure we are the correct person was found.

We're going to update **app.py** to add a **detect** route, and display the list of users found by Face API.

## Create detect function

1. Just below the end of `train` in **app.py**, add the following code.

``` python
@app.route('/detect', methods=['GET', 'POST'])
def detect():
    if request.method == 'GET':
        return render_template('detect.html')

    if 'file' not in request.files:
        return 'No file detected'
    image = request.files['file']

    # Find all the faces in the picture
    faces = face_client.face.detect_with_stream(image)

    # Get just the IDs so we can see who they are
    face_ids = list(map((lambda f: f.face_id), faces))

    # Ask Azure who the faces are
    identified_faces = face_client.face.identify(face_ids, person_group_id)

    names = get_names(identified_faces)

    if len(names) > 0:
        # Display the people
        return render_template('detect.html', names=names)
    else:
        # Display an error message
        return render_template('detect.html', message="Sorry, nobody looks familiar")
```

### Breaking down the code

#### Reading the image from the post

``` python
if request.method == 'GET':
    return render_template('detect.html')

if 'file' not in request.files:
    return 'No file detected'
image = request.files['file']
```

Similar to our boilerplate code in our [train function](./train-face-api.md), we start by looking to see if the user requested the page by **GET**, meaning they navigated to it. If so, we simply display the **detect** template. If they uploaded an image via **POST** we ensure there's a file uploaded, and then store it in a variable named `image`.

#### Getting the detected faces

``` python
# Find all the faces in the picture
faces = face_client.face.detect_with_stream(image)

# Get just the IDs so we can see who they are
face_ids = list(map((lambda f: f.face_id), faces))

# Ask Azure who the faces are
identified_faces = face_client.face.identify(face_ids, person_group_id)

names = get_names(identified_faces)
```

Finding the people in an image involves a few steps.

First, we send the image to Azure by calling `detect_with_stream`, which will return a collection of `face` objects. Each `face` object includes a series of properties, such as landmarks (facial features), bounding box, and an ID for the detected face. Unfortunately, we don't get the names as part of this execution. To get the name, we need to walk through a few more steps.

We need to get the ID for each detected face, and only the IDs. We use Python's `map` function to eliminate all other information from the face, retrieving just `face_id`. [map](https://docs.python.org/3/library/functions.html?highlight=sorted#map) accepts a function or lambda which it will call for each item in a list (the second parameter).

Once we have the IDs of the faces we can then call `identify`, which will give us a list of possible matches for each id. Unfortunately, this still doesn't give us the name! We're going to add another helper function to perform the final step of mapping the identified face to the name.

#### Displaying the results

``` python
if len(names) > 0:
    # Display the people
    return render_template('detect.html', names=names)
else:
    # Display an error message
    return render_template('detect.html', message="Sorry, nobody looks familiar")
```

Once we have the list of names from our `get_names` helper function, we then check to see if any were returned. Assuming there are, we pass them into the template of **detect.html**. If not, we simply send an error message.

## Finding the names

With our code built to detect the various faces in our image, it's time to turn our attention to putting names to them. We'll do this by creating a helper function to walk through the faces, and then ask our model to identify them.

1. Add the following to the end of **app.py**.

``` python
def get_names(identified_faces):
    names = []
    for result in identified_faces:
        # Find the top candidate for each face
        candidates = sorted(identified_faces.candidates, key=lambda c: c.confidence, reverse=True)
        # Was anyone recognized?
        if len(candidates) > 0:
            # Get just the top candidate
            top_candidate = candidates[0]
            # See who the person is
            person = face_client.person_group_person.get(person_group_id, top_candidate.person_id)

            # How certain are we this is the person?
            if top_candidate.confidence > .8:
                names.append('I see ' + person.name)
            else:
                names.append('I think I see ' + person.name)
```

### Breaking down the code

#### Finding our candidates

``` python
names = []
for result in identified_faces:
    # Find the top candidate for each face
    candidates = sorted(result.candidates, key=lambda c: c.confidence, reverse=True)
```

Each face will have a list of possible `candidates`, or who the person might be. Each candidate will have a `confidence`, which will be a 0.0 to 1.0 score of how certain the model is it identified a person. We want to find the person who most likely belongs to our face, meaning we need to perform a sort.

We perform the sort by using the Python function `sorted`. [sorted](https://docs.python.org/3/library/functions.html?highlight=sorted#sorted) accepts the list to sort as the first parameter. The second parameter is the function we wish to use to sort them, where we'll return the `confidence` score. The list will be sorted from lowest to highest, which we want to `reverse` so we can see the most likely candidate on the top.

#### Finding the name

``` python
if len(candidates) > 0:
    # Get just the top candidate
    top_candidate = candidates[0]
    # See who the person is
    person = face_client.person_group_person.get(person_group_id, top_candidate.person_id)
```

Each face might have multiple candidates (or none). We check to see if any possible matches were found by using `len`. Assuming there's at least one candidate, we grab the first by using `candidates[0]` and storing it in `top_candidate`. We **finally** put the name to the face by making a call to `get`, which allows us to pass in the ID of our person group, and then the ID of the person we found.

#### Match confidence

``` python
# How certain are we this is the person?
if top_candidate.confidence > .8:
    names.append('I see ' + person.name)
else:
    names.append('I think I see ' + person.name)
```

We close off by checking the confidence score and changing our output message accordingly. You can choose whatever threshold you feel is appropriate. For this sample, we've gone with 0.8, or 80%.

## Summary and next steps

We've now added the detection side of facial recognition. As we've seen there's a few steps we need to complete to make the magic happen. But once its built we can now discover who's in a photo! Next we'll [see the UI and test our application](./ui-test.md).
