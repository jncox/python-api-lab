****************
Supporting Files
****************

In this part of the lab we're going to add the supporting JavaScript and CSS files.  These files are critical to the layout and functionality of the app.

Checking directory structure
----------------------------

Create the following directories and files:

- **lab/static**
- **lab/static/css**
- **lab/static/css/ntnx.css**
- **lab/static/css/lib**
- **lab/static/js**
- **lab/static/js/ntnx.js**
- **lab/static/js/lib**
- **lab/static/layouts**

Adding Third Party Files
------------------------

From the URLs below, grab the relevant file, make sure the name is correct and extract it into the appropriate directory.

- CSS_ - extract to **lab/static/css/lib**
- Javascript_ - extract to **lab/static/js/lib**

**Note**: When extracting the ZIP files, ensure they are extracted **directly** to the directories above and not into subdirectories.

.. _CSS: https://github.com/nutanix-engineering/lab-content/raw/master/python-lab/v1/resources/css-lib.zip
.. _Javascript: https://github.com/nutanix-engineering/lab-content/raw/master/python-lab/v1/resources/js-lib.zip

Adding Custom Files
-------------------

From the URLs below, grab the relevant file, make sure the name is correct and copy it into the appropriate directory.

- ntnx.css_ - copy to **lab/static/css**
- ntnx.js_ - copy to **lab/static/js**
- dashboard.json_ - copy to **lab/static/layouts**

.. _ntnx.css: https://github.com/nutanix-engineering/lab-content/raw/master/python-lab/v1/ntnx.css
.. _ntnx.js: https://github.com/nutanix-engineering/lab-content/raw/master/python-lab/v1/ntnx.js
.. _dashboard.json: https://github.com/nutanix-engineering/lab-content/raw/master/python-lab/v1/dashboard.json

Referencing Supporting Files
----------------------------

- Open `lab/__init__.py` and, under the line that reads `assets = Environment(app)`, add the following Python code.

**Important note:** Python has strict indentation_ requirements.  For the code below, make sure the indentation begins at the same point as the `assets = Environment(app)` line.

.. _indentation: https://docs.python.org/3.6/reference/lexical_analysis.html

.. code-block:: python

    home_css = Bundle(
            'css/lib/reset.css',
            'css/lib/built-in.css',
            'css/lib/jquery-ui-custom.css',
            'css/lib/jq.gridster.css',
            'css/lib/jq.jqplot.css',
            'css/ntnx.css'
    )
    home_js = Bundle(
            'js/lib/jquery-2.1.3.min.js',
            'js/lib/classie.min.js',
            'js/lib/ntnx-bootstrap.min.js',
            'js/lib/modernizr.custom.min.js',
            'js/lib/jquery.jqplot.min.js',
            'js/lib/jqplot.logAxisRenderer.js',
            'js/lib/jqplot.categoryAxisRenderer.js',
            'js/lib/jqplot.canvasAxisLabelRenderer.js',
            'js/lib/jqplot.canvasTextRenderer.js',
            'js/lib/jquery.gridster.min.js',
            'js/ntnx.js'
    )

    assets.register('home_css',home_css)
    assets.register('home_js',home_js)

This code block registers two 'bundles' that allow our app to correctly load and access the JavaScript and CSS files.  Firstly, the bundles are created as `home_css` and `home_js`, then registered as application assets using `assets.register`.

With this done, we can continue with working on our application.