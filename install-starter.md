# Install the starter site

## Create a directory

We'll start by opening a command or terminal window, and then creating a directory for use during this workshop:

``` bash
# Windows
md contoso
cd contoso

# macOS or Linux
mkdir contoso
cd contoso
```

## Download the starter site

1. If Git isn’t installed on your computer, go to the [Git website](https://git-scm.com/) and install it now. Versions are available for Windows, macOS, and Linux.
2. In a command window or terminal, use `cd` to switch to the project directory you created in the previous subsection. Then use the following command to clone the GitHub repo containing the starter files for the website:

``` bash
git clone https://github.com/GeekTrainer/ai-workshop-starter.git .
```

> **Note:** Don’t forget to include the period at the end of the command. Otherwise, the files will be copied into a subdirectory of the project directory rather than into the project directory itself.

## Create a virtual environment and install packages

Let's create a virtual environment for the packages we'll be using. Virtual environments allow us to separate packages from other environments. Return to the command line and issuing the following command:

``` bash
# Windows
python -m venv env
.\env\Scripts\activate

# macOS or Linux
python3 -m venv env
. ./env/bin/activate
```

> **Note:** If you're using macOS or Linux the leading `.` for the `. ./env/bin/activate` **is** required as it indicates to Python where your source code resides.

## Install the necessary Python packages

Install the packages listed in `requirements.txt` by using `pip`

``` bash
pip install -r requirements.txt
```

## Next steps

If you're following along, the next step is to [start the site](./starting-site.md).
