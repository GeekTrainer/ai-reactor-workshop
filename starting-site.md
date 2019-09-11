## Explore the current files

1. Take a moment to browse the files that were copied into the project directory. Verify that they include:

- **app.py**, which holds the Python code that drives the site
- **image.py**, which holds a helper class we'll use for image management
- **templates/index.html**, the template for the home page
- **templates/base.html**, the template the remainder inherits from
- **templates/translate.html**, the template for translating signs
- **templates/train.html**, the template for training faces
- **templates/detect.html**, the template for detecting faces
- **static/main.css**, which contains CSS to dress up the home page
- **static/banner.jpg**, which contains the website banner
- **static/placeholder.jpg**, which contains a placeholder image for photos that have yet to be uploaded

> **NOTE:** We won't be focusing on working with HTML during this course. Again, we want to be able to focus on the code necessary for Cognitive Services.

## Launch the starter site

Let's see the starter site using Flask. We will configure Flask to run in development mode by setting the `FLASK_ENV` environmental variable. Running Flask in development mode is helpful when you’re developing a website because Flask automatically reloads any files that change while the site is running. If you let Flask default to production mode and change the contents of an HTML file or other asset, you have to restart Flask to see the change in your browser.

``` bash
# Windows
set FLASK_ENV=development
flask run

# macOS or Linux
export FLASK_ENV=development
flask run
```

> **NOTE:** Keep this window open, as we're going to be making changes to our site throughout the day. If you accidentally close it, you can restart your site by issuing the same commands from above.

## Open the site

Open a browser and navigate to `http://localhost:5000`. Confirm the website appears. If you click the buttons the three pages we'll be working with will each open. You will notice the functionality is limited to just displaying the image you upload. We're going to start adding code to add the ability to translate street signs, and eventually detect people in images.
