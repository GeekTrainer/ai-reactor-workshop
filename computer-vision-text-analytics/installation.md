# Set up a development environment

[Prerequisite: Getting started](../2-computer-vision-text-analytics.md)

The first order of business is to setup a development environment for Python., This will enable you to build and test your Contoso Travel website locally before deploying it to Azure and making it available publicly.

In this unit, you will install Python on your computer if it isn't already installed. Then you will create a [virtual Python environment](https://docs.python.org/3/library/venv.html), and [install Flask](https://flask.palletsprojects.com/en/1.1.x/installation/), the [Python SDK for Computer Vision API](https://pypi.org/project/azure-cognitiveservices-vision-computervision/), and [Python-dotenv](https://github.com/theskumar/python-dotenv).

## Install Python and create a virtual environment

In order to run Flask websites on your computer, both Flask and Python must be installed. The virtual environment will isolate the packages we install from other applications on your computer, and it's where we'll install the additional SDKs.

1. If Python 3.6 or higher isn't already installed on your computer, you can install it by visiting [python.org](https://python.org) and following the installation steps. If you're not sure if it's installed, you can test it by running the following command. If Python is installed the version number will appear in the console window.

``` bash
# Windows
python --version

# macOS or Linux
python3 --version
```

> **Note** If you do need to install Python, ensure you select the option to *Add Python 3.x to PATH*

![Dialog box for installing Python showing PATH option selected](../images/vision_python.png)

2. Upgrade to the latest version of `pip`, the Python package manager.

``` bash
# Windows
python -m pip install --upgrade pip

# macOS or Linux
python3 -m pip install --upgrade pip
```

3. Create a directory in the location of your choice. This will be the *project directory*, and will hold all the files for the Contoso Travel website. It's also where our virtual environment will be created. Change directories into the directory you created.

``` bash
# Windows
md contoso && cd contoso

# macOS or Linux
mkdir contoso && cd contoso
```

4. Open the directory in the code editor of your choice. If you're using [Visual Studio Code](https://code.visualstudio.com/) you can do this by typing the command `code .`, which will automatically open Code for the current directory.

5. Create a `requirements.txt` file. `requirements.txt` will be used to list the packages we'll be using for our application. Add the following to the file:

```
# web framework
flask
# Azure SDK components
azure-cognitiveservices-vision-computervision
# System variables
python-dotenv
# REST calls
requests
```

6. Create the virtual environment by returning to the command line and issuing the following command:

``` bash
# Windows
python -m venv env
.\env\Scripts\activate

# macOS or Linux
python3 -m venv env
. ./env/bin/activate
```

> **Note:** If you're using macOS or Linux the leading `.` for the `. ./env/bin/activate` **is** required as it indicates to Python where your source code resides.

7. Install the packages listed in `requirements.txt` by using `pip`

``` bash
pip install -r requirements.txt
```

## Install the Azure Command-Line Interface (CLI)

The [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) is a command-line environment for creating and managing Azure resources. Versions are available for Windows, macOS, and Linux. In subsequent unites you'll use the Azure CLI to create various Azure resources. Here you will install the Azure CLI and login to Azure.

1. If you haven't already installed Azure CLI, visit the [Azure CLI web page](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) and follow the installation instructions. You can determine if it's already installed by running `az -v` in a console window, which will display the verson number.

2. Login to Azure using the Azure CLI. The process will be completed using a browser window (the command will automatically open it); the browser can be closed once login is complete.

``` bash
az login
```

3. **optional** You may have multiple subscriptions for Azure associated with your account. We will be using the default subscription. If you aren't sure which one the Azure CLI is using, or you need to change it, you can use the following commands.

``` bash
# List all subscriptions
az account list

# Change the default (if needed)
az account set -s <SUBSCRIPTION_ID>
```

If you aren't familiar with the Azure CLI, you can learn more about it and the numerous commands it supports in [Get started with Azure CLI](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli?view=azure-cli-latest). Most operations in Azure can be performed by using either the CLI or the [Azure Portal](https://portal.azure.com). Power users tend to prefer the CLI, in part because CLI commands can be used in scripts to automate repetitive tasks.

## Summary and next steps

We now have Python and the Azure CLI installed. We also have the environment setup for development. It's now time to turn our attention to learning about [Python](./python.md).
