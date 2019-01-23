*************
App Structure
*************

The key directories of our app are as follows.

- A folder called `lab`.  This folder contains our project's code and all associated files.
- `nutanix/`, the folder containing our virtual environment files.

**Please create the `lab/` folder now, if you haven't already.**  The rest of this section is meant as FYI only - we'll create these files as we go through the lab.

- `lab/__init__.py`, the application's "main" entry point.
- `lab/static/`, the folder containing our JavaScript, CSS and JSON files that describe our main view's layout.
- `lab/static/js/lib/` and `lab/static/css/lib/`, collections of third-party JavaScripts and CSS to support our app.
- `lab/templates/`, our HTML layouts.
- `lab/util/apiclient/__init__.py`, our custom Python class that describes what an API request looks like and the functions associated with it.
- `lab/ajax.py`, the Python class that carries out AJAX requests called via JavaScript.
- `lab/forms.py`, the Python script that describes the user input form responsible for collecting credentials and cluster IP address.
- `config.py`, the Python script that contains configuration information.  Note that this file is **not** in the `lab` folder!
- `lab/index.py`, the Python blueprint responsible for handling the `/` route (URL).

In addition to the actual project files, most of which don't exist yet, the Git `.gitignore` file for this application is as follows.  This specific `.gitignore` file is often used with Python projects.  For our app, the `instance/` line has been added.  If you are using Git to manage your project's source, this `.gitignore` is a good one to use.

- If you are going to store your application in Git or GitHub (recommended), create a file named `.gitignore` and add the following contents:

.. code-block:: bash

    venv/

    *.pyc
    __pycache__/

    instance/

    .pytest_cache/
    .coverage
    htmlcov/

    dist/
    build/
    *.egg-info/

The ApiClient class
-------------------

The first file we'll create is one of the most supporting files in the app.

This file is the **ApiClient** class and describes what an API request looks like as well as the functions associated with it.

- Create the `lab/util` folder.
- Create the `lab/util/apiclient/` folder.
- Create `lab/util/apiclient/__init__py`.  `__init__.py` is a reserved filename that Python looks for when instantiating a class.  The contents of `__init.py__` should be as follows:

.. code-block:: python

    #!/usr/bin/env python3.6

    import sys
    import requests
    from requests.auth import HTTPBasicAuth
    import json

    class ApiClient():

        def __init__(self, method, cluster_ip, request, body, username, password, version='v3',root_path='api/nutanix'):
            self.method = method
            self.cluster_ip = cluster_ip
            self.username = username
            self.password = password
            self.base_url = f"https://{self.cluster_ip}:9440/{root_path}/{version}"
            self.entity_type = request
            self.request_url = f"{self.base_url}/{request}"
            self.body = body

        def get_info(self, show_info=False):

            if show_info == True:
                print(f"Requesting '{self.entity_type}' ...")
            headers = {'Content-Type': 'application/json; charset=utf-8'}
            try:
                if(self.method == 'post'):
                    r = requests.post(self.request_url, data=self.body, verify=False, headers=headers, auth=HTTPBasicAuth(self.username, self.password), timeout=60)
                else:
                    r = requests.get(self.request_url, verify=False, headers=headers, auth=HTTPBasicAuth(self.username, self.password), timeout=60)
            except requests.ConnectTimeout:
                print(f'Connection timed out while connecting to {self.cluster_ip}. Please check your connection, then try again.')
                sys.exit()
            except requests.ConnectionError:
                print(f'An error occurred while connecting to {self.cluster_ip}. Please check your connection, then try again.')
                sys.exit()
            except requests.HTTPError:
                print(f'An HTTP error occurred while connecting to {self.cluster_ip}. Please check your connection, then try again.')
                sys.exit()

            if r.status_code >= 500:
                print(f'An HTTP server error has occurred ({r.status_code}, {r.text})')
            else:
                if r.status_code == 401:
                    print(f'An authentication error occurred while connecting to {self.cluster_ip}. Please check your credentials, then try again.')
                    sys.exit()
                #if r.status_code > 401:
                    #print(json.loads(r.text)['message_list'][0]['message'])
                    #sys.exit()
                # else:
                    # print('Connected and authenticated successfully.')

            return(r.json())

A few things to note about this class:

- The `__init__` function runs when the class is instantiated and describes **how** it should be instantiated.
- In our ApiClient class, we are setting some properties of the class, such as the IP address of our cluster, the cluster and password (etc).
- The `get_info` function is called on-demand after the class is instantiated and carries out the actual API request.
- The `try` section of the `get_info` function attempts to complete the API request and get an HTTP response from the Nutanix API.
- The remaining `except` sections specify various exceptions that can be caught and dealt with accordingly.  For example, looking for `r.status_code >= 500` will catch any HTTP 500 errors.  This type of catch-all is bad practice in production environments but suits our basic demo requirements well enough.
- If no exceptions are caught, the JSON response from the API request is returned via `return(r.json())`.

With the basic application structure and main supporting class created, we can move forward with creating the other parts of our app.