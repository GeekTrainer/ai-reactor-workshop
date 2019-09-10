# Set up a development environment

[Prerequisite: Getting started](../2-computer-vision-text-analytics.md)

The first order of business is to setup a development environment for Python, and to install the tools for managing Azure.

## Install Python and create a virtual environment

If you already have Python installed, you can skip ahead to the bottom of this document where you'll install the Azure CLI, which we'll be using to manage our Azure resources.

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

## Summary and next steps

We now have Python and the Azure CLI installed. We also have the environment setup for development. It's now time to turn our attention to learning about [Python](./python.md).
