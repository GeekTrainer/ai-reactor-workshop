# Detect a person using Face API

[Prerequisite: Train a model with Face API](./train-face-api.md)

With a model trained, we're now able to detect faces. When working with any AI (or ML) solution, the answers are never perfect. In the case of Face API, a confidence score will be returned with the execution, which we can then use to indicate to the user how sure we are the correct person was found. We're going to update **app.py** to add detection capabilities, and display the list of users found by Face API.

## General flow of identification with Face API

Identifying a face using Face API requires a few steps. When you send an image to Face API, an ID is automatically generated and applied to every unique face detected, and is stored for 24 hours. We can then use this ID to perform additional functions, such as asking for similar faces or, in our case, identification. In the helper function we create, you'll notice these high level steps: (we'll review the code as always)

1. Upload an image to Face API, and receive a collection of faces
2. Convert the list of faces to just a list of IDs (we just need the IDs)
3. Send the IDs to Face API for a list of possible candidates
4. Each face will have 0 **or more** possible candidates
5. Get the top candidate, and ask for the `person` (which we created earlier)
6. Retrieve the name from the `person`

## Create detect_people helper function

At the bottom of **app.py**, add the following code. As with `train_person`, there's quite a bit required as we can't simply ask for the person directly. But we'll step through the code to help explain what's going on.

``` python
def detect_people(client, person_group_id, image):
    # Find all faces in the image
    faces = face_client.face.detect_with_stream(image)

    # Get just the IDs (for identification)
    face_ids = list(map((lambda face: face.face_id), faces))

    # Identify the faces
    identified_faces = face_client.face.identify(face_ids, person_group_id)

    results = []
    # Loop through each identified face
    for face in identified_faces:
        # Get all candidates - who might this face be?
        candidates = face.candidates

        # If no candidates, continue
        if len(candidates) == 0:
            continue

        # Sort by most likely candidate
        candidates = sorted(candidates, key=(lambda candidate: candidate.confidence), reverse=True)

        # Get just the top candidate
        top_candidate = candidates[0]

        # Ask our model who this person is
        person = face_client.person_group_person.get(person_group_id, top_candidate.person_id)

        # Add the person to the list of results
        if top_candidate.confidence > .8:
            results.append('I see ' + person.name)
        else:
            results.append('I think I see ' + person.name)

    return results
```

### Breaking down the code

#### Identifying the faces

``` python
# Find all faces in the image
faces = face_client.face.detect_with_stream(image)

# Get just the IDs (for identification)
face_ids = list(map((lambda face: face.face_id), faces))

# Identify the faces
identified_faces = face_client.face.identify(face_ids, person_group_id)
```

`detect_with_stream` will return a collection of faces. Each face will have all the metadata associated with the face, such as the coordinates of its location in the image, the location of the landmarks (such as nose and mouth), and an ID which Face API will keep to provide additional information about the face.

In our case, the only thing we need is just the ID, as it's the only component required for asking Face API for person identification. We use the built-in [map](https://docs.python.org/3/library/functions.html#map) function to get a list of just the `face_id`s.

Finally, we pass the `face_ids` into `identify` to ask Face API to perform the identification. `identify` will give us a list of possible matches from the model we've trained up. Each possible match, or `candidate` will contain a `confidence` score (0-1 index), and `person_id`, which is the ID of the person possibly matched in our model.

#### Getting the top candidates

``` python
for face in identified_faces:
    # Get all candidates - who might this face be?
    candidates = face.candidates

    # If no candidates, continue
    if len(candidates) == 0:
        continue

    # Sort by most likely candidate
    candidates = sorted(candidates, key=(lambda candidate: candidate.confidence), reverse=True)

    # If no candidates, continue
    if len(candidates) == 0:
        continue

    # Get just the top candidate
    top_candidate = candidates[0]
```

We start our loop through the list of possible matches by getting list of `candidates` from the face. If there are no candidates, we look for the next possible matched face. If there are candidates, we sort them by `confidence` in reverse order (biggest to smallest) by using [sorted](https://docs.python.org/3/library/functions.html#sorted). `sorted` will give us a list, so we grab the first (the most likely match or `candidate`) by using the 0 indexer.

#### Getting the name of the top candidate

``` python
# Ask our model who this person is
person = face_client.person_group_person.get(person_group_id, top_candidate.person_id)

# Add the person to the list of results
if top_candidate.confidence > .8:
    results.append('I see ' + person.name)
else:
    results.append('I think I see ' + person.name)
```

Sadly the candidate doesn't include the person's name, but it does have their `person_id`. We can pass this ID, along with the ID of the group, into `get` on `person_group_person` to get the `person`. And from `person` we can finally retrieve the name.

We finish by appending to our results list an appropriate message. If the score is greater than .8 (80%), then it's very likely we have the right person. Otherwise, we're not entirely sure.

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

## Calling our helper function from detect

Let's update `detect` to now call our `detect_people` function. Inside of `detect` in **app.py**, just below the comment which reads `# TODO: Add code to detect people in picture`, add the following code. We call our helper function, passing in the `face_client`, the ID of the person group, and the blob of the image.

``` python
    # TODO: Add code to detect people in picture
    messages = detect_people(face_client, person_group_id, image.blob)
```

> **NOTE:** The tab at the beginning of the line of code is required. Python uses tab levels to identify enclosures, and we want to put the call to `detect_people` inside `detect`. It should be in line with the existing comment.

## Testing our site

Let's see our model in action! After saving all files, the process running our site should automatically reload. If you closed the command or terminal window you used to launch the site previously, you can open a new one, navigate to the directory containing your code, and then execute the following commands:

``` bash
# Windows
set FLASK_ENV=development
flask run

# macOS and Linux
export FLASK_ENV=development
flask run
```

In a browser, navigate to **http://localhost:5000/detect**. Click the **Upload Photo** button, and select a picture containing a person you've used to train your model. The picture can contain multiple faces. Check out the results!

> **NOTE:** You might not get perfect results. Remember the key to training any model is plenty of varied data. You may need to upload pictures of the same person in various settings, lighting configurations, etc.

## Summary and next steps

We've now added the detection side of facial recognition. As we've seen there's a few steps we need to complete to make the magic happen. But once its built we can now discover who's in a photo! Next we'll [see the UI and test our application](./deploy.md).
