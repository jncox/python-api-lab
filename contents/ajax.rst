*************
The AJAX View
*************

JavaScript function
-------------------

The `ajax` view is a key part of our application.  When the user enters their cluster or CVM IP address, credentials and hits the 'Go!' button, JavaScript makes various calls to the Nutanix APIs.  These calls are handled via AJAX_ so the user's browser doesn't get refreshed every time.

.. _AJAX: https://en.wikipedia.org/wiki/Ajax_(programming)

As an example, let's first take a look at the JavaScript function that gets a count of VMs running on our cluster.

.. code-block:: JavaScript

    vmInfo: function( cvmAddress, username, password )
    {

        vmData = $.ajax({
            url: '/ajax/vm-info',
            type: 'POST',
            dataType: 'json',
            data: { _cvmAddress: cvmAddress, _username: username, _password: password },
        });

        vmData.success( function(data) {
            NtnxDashboard.resetCell( 'vmInfo' );
            $( '#vmInfo' ).addClass( 'info_big' ).append( '<div style="color: #6F787E; font-size: 25%; padding: 10px 0 0 0;">VM(s)</div><div>' + data['metadata']['count'] + '</div><div></div>');
        });

        vmData.fail(function ( jqXHR, textStatus, errorThrown )
        {
            console.log('error getting vm info')
        });
    },

Here are the most important steps carried out by this function:

- `vmInfo: function( cvmAddress, username, password )` - the name of the function, accepting the cluster/CVM IP address & and credentials.
- `vmData = $.ajax({` - use jQuery_ to initiate an AJAX request.
- `url: '/ajax/vm-info',` - the URL of the AJAX call that will be made.
- The block beginning with `vmData.success( function(data) {` - the JavaScript that will run when the AJAX call is successful.
- The block beginning with `vmData.fail(function ( jqXHR, textStatus, errorThrown )` - displays a message in the browser console when any error is encountered.  Of course, this "catch-all" approach is something that should be avoided before deploying to or developing for a production environment, but provides information that can be used to diagnose an issue.

.. _jQuery: https://jquery.com/

The AJAX
--------

Now that we are familiar with the simple JavaScript code that will make the AJAX call, let's look at the Python code that carries out the first part of the API request.

.. code-block:: python

    """
    get the vm count
    """
    @bp.route('/vm-info',methods=['GET','POST'])
    def vm_info():
        # get the request's POST data
        get_form()
        client = apiclient.ApiClient('get', cvmAddress,'vms','',username,password,'v2.0')
        results = client.get_info()
        return jsonify(results)

Here are the most important steps carried out by this function:

- `@bp.route('/vm-info',methods=['GET','POST'])` - Specify the URL that will respond to the AJAX call and allow both GET and POST methods.  During testing it can be useful to allow both methods, although production apps would only allow the methods that are explicitly required.
- `get_form()` - Get the user data available in the POST request.  This includes the CVM/Cluster IP address, username and password.
- `client = apiclient.ApiClient('get', cvmAddress,'vms','',username,password,'v2.0')` - Create an instance of our `ApiClient` class and set the properties we'll need to execute the API request.

**Note:** You'll notice a few parameters being passed during instantiation of the ApiClient class.  As an optional step, open `lab/util/apiclient/__init__py` and look at the other parameters that can be passed.  For example, you can specify the API endpoint we're interested and the API version.  These are useful options for using the same ApiClient class with different versions of the APIs.

- `results = client.get_info()` - Execute the actual API request.
- `return jsonify(results)` - Convert the API request results to JSON format and return the JSON back to the calling JavaScript, where it will be processed and displayed in our app.

The API request
---------------

Because the Nutanix REST APIs are designed to be simple to use, it's very easy to understand what the request itself is doing.

In the **Intro** section of this lab, we looked at the various Nutanix API versions that are available to you.  In the example above, we are using Nutanix API v2.0 to get a count of VMs running on our cluster.  The JavaScript making the AJAX call and the Python executing the API request, are constructing the following GET request.

**Note:** The request coming from the JavaScript to our Python view is an HTTP POST request.  The request to the API itself, in this example, is an HTTP GET request.

.. code-block:: html

    https://<cluster_virtual_ip>:9440/api/nutanix/v2.0/vms

If you were to change `<cluster_virtual_ip>` to your cluster IP address and browse to that URL, you would probably see an error saying `"An Authentication object was not found in the SecurityContext"`.  That's because we haven't specified the credentials that should be used for the request.

**Note:** If you have an open browser tab where you are already logged in and authenticated with an active Nutanix Prism session, it is possible the request may succeed.

Now that we have our API request URL, we can add HTTP Basic Authentication in the form of a username and password, then simulate the entire request using cURL.  For this quick test we will assume the following:

- Cluster virtual IP address is **192.168.1.10**
- Cluster username is **admin**
- Cluster password is **nutanix4u**

.. code-block:: bash

  curl -X GET \
    https://192.168.1.10:9440/api/nutanix/v2.0/vms \
    -H 'Accept: application/json' \
    -H 'Content-Type: application/json' \
    --insecure \
    --basic --user admin:nutanix4u

Please be mindful of the `--insecure` parameter, as detailed in the lab intro.