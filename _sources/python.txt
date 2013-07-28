Recommendations for Python
==========================

In addition to the :doc:`general code guidelines <general>` the following recommendations apply specifically for Python implementations.

Filename conventions
--------------------

File names should have all-lowercase names, matching to the implemented class.
If class name is a composite of several words, each word in a file name should be separated with an underscore ``-`` character.
For example, module implementing ``KeyLocator`` class should be named ``key_locator.py``.

Combining implementation of different (related) classes in the same module is highly discourabged, unless the secondary class implementation is relatively short.

The goal of this naming pattern is to allow a reader to quickly navigate through the NDN-CCL codebase to locate the source file relevant to a specific type.
Also, using lowercase names does not introduce unexpected errors for case insensitive filesystem.

Code layout
-----------

The code layout follows the GNU coding standard layout extended to Python.
**Do not use tabs for indentation**.
Indentation spacing is 4 spaces as outlined below:

.. code-block:: python

    def fooBar (test):
        if test:
            # do stuff here
            pass 
        else:
            # do other stuff here
            pass 

        for i in xrange (0, 100):
            # do loop
            pass
    
        while test:
            # do while


File layout and code organization
---------------------------------

Each ``my_class.py`` module file should start with the following comments: the first line ensures that developers who use the emacs editor will be able to indent your code correctly. 
The following lines ensure that your code is licensed under the BSD, that the copyright holders are properly identified (typically, you or your employer), and that the actual author of the code is identified.
If you are releasing the code under different license, adjust the header appropriately.
The latter is purely informational and we use it to try to track the most appropriate person to review a patch or fix a bug. 
Please do not add the "All Rights Reserved" phrase after the copyright statement.

Each module should contain docstring that briefly describes purpose of the module.
Docstring may contain any information in reStructuredText format, which will be used for automated document generation.
For more information about docstring, refer to `PEP 257 -- Docstring Conventions <http://www.python.org/dev/peps/pep-0257/>`_.

.. code-block:: python

    # -*- Mode:python; c-file-style:"gnu"; indent-tabs-mode:nil -*- */
    # 
    # Copyright (c) YEAR COPYRIGHTHOLDER
    # 
    # BSD license, See the LICENSE file for more information
    # 
    # Author: MyName <my@email>
    #

    """
    Describe purpose of the module
    """

Comments
--------

The project uses `Sphinx Documentation Generator to document the interfaces <http://http://sphinx-doc.org/>`_, and uses docstring with reStructuredText content for improving the clarity of the code internally.
All classes and public API methods should have docstring comments, for example:

.. code-block:: python

    class MessageError (Exception):
        """Base class for errors in the email package."""

    def myMethod (self, value, optional = None):
        """
        Do something

        :param value: Value parameter
        :type value: str
        :param optional: Optional parameter
        :type optional: bool
        """
        

All parameters and return values for public interface should be documented.

As for comments within the code, comments should be used to describe intention or algorithmic overview where is it not immediately obvious from reading the code alone.
There are no minimum comment requirements and small routines probably need no commenting at all, but it is hoped that many larger routines will have commenting to aid future maintainers.
Please write complete English sentences and capitalize the first word unless a lower-case identifier begins the sentence.
Two spaces after each sentence helps to make emacs sentence commands work.

Variable declaration should have a short, one or two line comment describing the purpose of the variable, unless it is a local variable whose use is obvious from the context. 
The short comment should be on the same line as the variable declaration, unless it is too long, in which case it should be on the preceding lines.


Miscellaneous items
-------------------

- The following emacs mode line should be the first line in a file (or after the shebang line):

    .. code-block:: python

        # -*- Mode:python; c-file-style:"gnu"; indent-tabs-mode:nil -*- */

- Use a space between the function name and the parentheses, e.g.:

    .. code-block:: c++

        def mySub(var):     # avoid this
            pass
        def mySub (var):    # use this instead
            pass
    
    This spacing rule applies both to function declarations and invocations.

.. - Expose class members through access functions, rather than direct access to a public object. The access functions are typically named ``get`` and ``set``. For example, a member ``m_delayTime`` might have accessor functions ``getDelayTime ()`` and ``setDelayTime ()``.
