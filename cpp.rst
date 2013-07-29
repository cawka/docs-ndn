Recommendations for C++
=======================

In addition to the :doc:`general code guidelines <general>` the following recommendations apply specifically for C++ implementations.

Filename conventions
--------------------

File names should have all-lowercase names, matching to the implemented class.
If class name is a composite of several words, each word in a file name should be separated with a ``-``.
For example, a header file that declares ``KeyLocator`` class should be named ``key-locator.h`` and the corresponding source file defining the class should be named ``key-locator.cc``.

Combining implementation of different (related) classes in the same header/source file is highly discourabged, unless the secondary class implementation is relatively short.

The goal of this naming pattern is to allow a reader to quickly navigate through the NDN-CCL codebase to locate the source file relevant to a specific type.
Also, using lowercase names does not introduce unexpected errors for case insensitive filesystem.

Code layout
-----------

The code layout follows the GNU coding standard layout for C and extends it to C++. 
**Do not use tabs for indentation**.
Indentation spacing is 2 spaces (the default emacs C++ mode) as outlined below:

.. code-block:: c++

    void
    fooBar (void)
    {
      if (test)
        {
          // do stuff here
        }
      else
        {
          // do other stuff here
        }
    
      for (int i = 0; i < 100; i++)
        {
          // do loop
        }
    
      while (test)
        {
          // do while
        }
    
      do
        {
          // do stuff
        } while ();
    }

Each statement should be put on a separate line to increase readability, and multi-statement blocks following conditional or looping statements are always delimited by braces (single-statement blocks may be placed on the same line of an ``if ()`` statement). 
Each variable declaration is on a separate line. 
Variables should be declared at the point in the code where they are needed, and should be assigned an initial value at the time of declaration. 
Except when used in a switch statement, the open and close braces ``{`` and ``}`` are always on a separate line. Do not use the C++ ``goto`` statement.

The layout of variables declared in a class may either be aligned with the variable names or unaligned, as long as the file is internally consistent and no tab characters are included. 
Examples:

.. code-block:: c++

    int varOne;
    double varTwo;   // OK (unaligned)
    int      varOne;
    double   varTwo;  // also OK (aligned)

Naming convention for member variables
--------------------------------------

Global variables should be prefixed with a ``g_`` and member variables should be prefixed with a ``m_`` (static member variables should be prefixed ``s_``).
The goal of that prefix is to give a reader a sense of where a variable of a given name is declared to allow the reader to locate the variable declaration and infer the variable type from that declaration. 

For example, you could declare in your class header ``my-class.h``:

.. code-block:: c++

    typedef int NewTypeOfInt_t;
    const uint8_t PORT_NUMBER = 17;
    extern int g_aGlobalVar;
    
    class MyClass
    {
    public:
      void
      myMethod (int aVar);

    private:
      int m_aVar;
      static int s_anotherVar;
    };

and implement in your class file ``my-class.cc``:

.. code-block:: c++

    static int g_aStaticVar = 100;
    int g_aGlobalVar = 1000;
    int MyClass::s_anotherVar = 10;

    void
    MyClass::myMethod (int aVar)
    {
      m_aVar = aVar;
    }

As an exception to the above, the members of structures do not need to be prefixed with an ``m_``.

Finally, do not use Hungarian notation, and do not prefix enums, classes, or delegates with any letter.

File layout and code organization
---------------------------------

Each ``my-class.h`` header should start with the following comments: the first line ensures that developers who use the emacs editor will be able to indent your code correctly. 
The following lines ensure that your code is licensed under the BSD, that the copyright holders are properly identified (typically, you or your employer), and that the actual author of the code is identified.
If you are releasing the code under different license, adjust the header appropriately.
The latter is purely informational and we use it to try to track the most appropriate person to review a patch or fix a bug. 
Please do not add the "All Rights Reserved" phrase after the copyright statement.

.. code-block:: c++

    /* -*- Mode:C++; c-file-style:"gnu"; indent-tabs-mode:nil -*- */
    /*
     * Copyright (c) YEAR COPYRIGHTHOLDER
     *
     * BSD license, See the LICENSE file for more information
     *
     * Author: MyName
     */

Below these C-style comments, always include the following which defines a set of header guards (``MY_CLASS_H``) used to avoid multiple header includes and provide a set of doxygen comments for the public part of your class API.
Detailed information on the set of tags available for doxygen documentation is described in the `doxygen website <http:/doxygen.org>`_.

.. code-block:: c++

    /* -*- Mode:C++; c-file-style:"gnu"; indent-tabs-mode:nil -*- */
    /*
     * Copyright (c) YEAR COPYRIGHTHOLDER
     *
     * BSD license, See the LICENSE file for more information
     *
     * Author: MyName
     */

    #ifndef MY_CLASS_H
    #define MY_CLASS_H
    
    /**
     * @brief Short one-line description of the purpose of your class
     *
     * A longer description of the purpose of your class after a blank
     * empty line.
     */
    class MyClass
    {
    public:
      MyClass ();
      /**
       * @brief Short one-line description of the purpose of the method
       * @param firstParam a short description of the purpose of this parameter
       * @returns a short description of what is returned from this function.
       *
       * A detailed description of the purpose of the method.
       */
      int 
      doSomething (int firstParam);

    private:
      void 
      myPrivateMethod ();

    private:
      int m_myPrivateMemberVariable;
    };
    
    #endif // MY_CLASS_H

The ``my-class.cc`` file is structured similarly:

.. code-block:: c++

    /* -*- Mode:C++; c-file-style:"gnu"; indent-tabs-mode:nil -*- */
    /*
     * Copyright (c) YEAR COPYRIGHTHOLDER
     *
     * BSD license, See the LICENSE file for more information
     *
     * Author: MyName <my@email>
     */
    #include "my-class.h"

    MyClass::MyClass ()
    {
    }

    ...


Comments
--------

The project uses `Doxygen to document the interfaces <http://doxygen.org>`_, and uses comments for improving the clarity of the code internally.
All public methods should have Doxygen comments.
Doxygen comments should use the C comment style.
For non-public methods, the ``@internal`` tag should be used. Please use the ``@see`` tag for cross-referencing.
All parameters and return values for public interface should be documented.

As for comments within the code, comments should be used to describe intention or algorithmic overview where is it not immediately obvious from reading the code alone.
There are no minimum comment requirements and small routines probably need no commenting at all, but it is hoped that many larger routines will have commenting to aid future maintainers.
Please write complete English sentences and capitalize the first word unless a lower-case identifier begins the sentence.
Two spaces after each sentence helps to make emacs sentence commands work.
Short one-line comments and long comments can use the C++ comment style; that is, '//', but longer comments may use C-style comments:

.. code-block:: c++

    /*
     * A longer comment,
     * with multiple lines.
     */

Variable declaration should have a short, one or two line comment describing the purpose of the variable, unless it is a local variable whose use is obvious from the context. 
The short comment should be on the same line as the variable declaration, unless it is too long, in which case it should be on the preceding lines.

Every function should be preceded by a detailed (Doxygen) comment block describing what the function does, what the formal parameters are, and what the return value is (if any).
Either Doxygen style ``\`` or ``@`` is permitted.  Every class declaration should be preceded by a (Doxygen) comment block describing what the class is to be used for.

Miscellaneous items
-------------------

- The following emacs mode line should be the first line in a file:

    .. code-block:: c++

        /* -*- Mode:C++; c-file-style:"gnu"; indent-tabs-mode:nil; -*- */

- Const reference syntax:

    .. code-block:: c++

        void mySub (const T&);    // Method 1  (prefer this syntax)
        void mySub (T const&);    // Method 2  (avoid this syntax)

- Use a space between the function name and the parentheses, e.g.:

    .. code-block:: c++

        void mySub(const T&);     // avoid this
        void mySub (const T&);    // use this instead
    
    This spacing rule applies both to function declarations and invocations.

- Do not use ``nil`` or ``NULL`` constants; use ``0`` (improves portability)

    For C++11 code, use ``nullptr`` instead.

- Consider whether you want the default copy constructor and assignment operator in your class, and if not, make them private such as follows:

    .. code-block:: c++

        private:
            ClassName (const Classname&) { };
            ClassName& operator= (const ClassName&) { return *this; }

    For C++11 code, you can use the following instead:

    .. code-block:: c++

        ClassName & operator = (const ClassName &) = delete;
        ClassName (const ClassName &) = delete;

- Avoid returning a reference to an internal or local member of an object:

    .. code-block:: c++

        a_type& foo ();  // should be avoided, return a pointer or an object.
        const a_type& foo () const; // same as above

    This guidance does not apply to the use of references to implement operators.

- Expose class members through access functions, rather than direct access to a public object. The access functions are typically named ``get`` and ``set``. For example, a member ``m_delayTime`` might have accessor functions ``getDelayTime ()`` and ``setDelayTime ()``.

- For standard headers, use the C++ style of inclusion, such as
 
    .. code-block:: c++

        #include <cheader>

    instead of

    .. code-block:: c++

        #include <header.h>

- Do not bring the C++ standard library namespace into header files using the ``using`` directive; i.e., avoid ``using namespace std;`` in header files.  This restriction does not apply to source files (``.cc``).
