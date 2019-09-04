# Computer Vision API and Text Analytics

As we've discussed, AI is a tool whose usefulness spans the practical and inspirational. We can sometimes see the reach of its capabilities even within a single app. For example, consider an application which could translate signage in a foreign language. Such an application would be useful for tourists on vacation, or could even help doctors save lives as they travel out to foreign countries to stop an epidemic, such as ebola, before it can spread.

In this module we'll examine building a web application like this - a website we'll call Contoso Travel - step by step. Users of this website, whether travelling for business or pleasure, will be able to upload photos of street signs taken from their phones and see the translation of the sign. To build the application, we'll use [Computer Vision API](https://azure.microsoft.com/services/cognitive-services/computer-vision/) and [Translator Text API](https://azure.microsoft.com/services/cognitive-services/translator-text-api/) to read the text and translate it respectively.

To build the website, you'll perform the following tasks:

1. [Install Python and configure a virtual environment for development](./computer-vision-text-analytics/installation.md)
2. Get a quick [overview of Python](./computer-vision-text-analytics/python.md)
3. Get a quick [overview of Flask](./computer-vision-text-analytics/flask.md)
4. [Build a Flask application](./computer-vision-text-analytics/site-creation.md) which accepts photo uploads
5. [Create keys](./computer-vision-text-analytics/create-azure-resources.md) for Computer Vision and Text Analytics
6. [Call Computer Vision API](./computer-vision-text-analytics/computer-vision.md) to extract text from photos
7. [Call the Translator Text API](./computer-vision-text-analytics/text-analytics.md) to translate text from extracted photos
8. [Deploy the application](./computer-vision-text-analytics/deploy.md) to Azure

To produce the application you'll use [Python](https://python.org), one of the world's most popular programming languages, and [Flask](http://flask.pocoo.org/) for web development. Don't worry if you're not familiar with either Python or Flask; we will cover enough in this section for you to be able to produce the website.
