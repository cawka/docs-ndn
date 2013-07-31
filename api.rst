Propsed general API for NDN-CCL
===============================

The following describes a proposed API for NDN-CCL.
The description uses an abstract python-like syntax and just gives a guideline for components, naming of the components, and naming of the components' method in the API.

Base elements (fields)
----------------------

Blob
~~~~

.. class:: Blob

    Generic class representing continuous memory region.  Equivalent to :py:class:`bytes` in python and almost equivalent to :cpp:class:`std::vector<char>` in C++.

    In C++ in addition to standard :cpp:class:`std::vector<char>` interface, there could additional helpers to facilitate access to the underlying data:

    .. method:: buf ()

        Const and non-const versions to get pointer to the first byte

    .. method:: toBlob ()

        Create a copy of Blob in :cpp:class:`std::string`

Name component
~~~~~~~~~~~~~~

.. class:: name.Component

    Bases: :class:`Blob`

    Specialized form of memory region that is used for name components.
    In addition to being memory, this component implements the appropriate ordering (e.g., canonical ordering in CCNx sense or any other ordering that we define later).
    In case name component 

    :Constructors:

    .. method:: __init__ (self, value = None)

        Potentially overloaded constructor taking zero or one argument: 

        - If value is None (no arguments), an empty name component is created.
        - If value is :py:class:`string`, :py:class:`bytes`, or another :py:class:`name.Component`, a new name component will be created, containing a copy of the supplied data

        C++ can also have forms of 2 argument constructor:

        - ``(const void *buf, size_t length)``: create from a memory buffer
        - ``(const_iterator begin, const_iterator end)``: templated version to create component from another container of bytes

    .. classmethod:: fromUri (cls, uri)

        Construct :py:class:`name.Component` from URI

    .. classmethod:: fromNumber (cls, number)

        Create network-ordered numeric component

        Number is encoded and added in network order. Tail zero-bytes are not included.
        For example, if the number is 1, then 1-byte binary blob will be added  0x01.
        If the number is 256, then 2 binary blob will be added: 0x01 0x01
  
        If the number is zero, an empty component will be created

    .. classmethod:: formNumberWithMarker (cls, number, marker)

    .. classmethod:: fromSeqNum (cls, seqnum)

    .. classmethod:: fromControlNum (cls, controlNum)

    .. classmethod:: fromBlkId (cls, blkId)

    .. classmethod:: fromVersion (cls, version)

    :Accessors:

    The stored data should be directly accessible via :py:class:`Blob` interface.

    .. method:: toUri (self)

        Another variant in C++ ``toUri (std::ostream &os)``

        There should also be a language-specific of this method to convert name component to URI.
        For example, in python it is :py:meth:`str` method and in C++ it is ``operator<<(ostream &,const Component&)``.

    .. method:: toNumber (self)

    .. method:: toNumberWithMarker (self)

    .. method:: toSeqNum (self)

    .. method:: toControlNum (self)

    .. method:: toBlkId (self)

    .. method:: toVersion (self)

    .. method:: compare (self, other)

        Implementation of appropriate ordering

        If possible, name.Component need to overload ``==``, ``<=``, ``<``, ``>=``, and ``>`` operations

Name
~~~~

.. class:: Name

    Bases: :class:`vector` or :class:`list`

    NDN name abstraction

    :Constructors:

    .. method:: __init__ (self, value = None)

        Potentially overloaded constructor taking zero or one argument: 

        - If value is None (no arguments), an empty Name is created.
        - If value is :py:class:`string`, equivalent to :py:meth:`Name.fromUri`
        - If value is :py:class:`list` or another :py:class:`Name`, a new name will be created, containing a copy of the supplied data

        C++ can also have forms of 2 argument constructor:

        - ``(const_iterator begin, const_iterator end)``: templated version to create Name from another container (templated version)

    .. classmethod:: fromUri (cls, uri)

        Construct :py:class:`name.Component` from URI

    .. classmethod:: fromWire (cls, buffer)

    :Modifiers:

    .. method:: append (self, value)

        Potentially overloaded method, taking one argumnet:

        - if value is :py:class:`name.Component` or any other object "convertable" to name component, append this component to self and return self (to allow chaining)
        - if value is :py:class:`Name` or any other object "convertable" to Name, append all name components to self and return self

        C++ may have anoher method ``appendBySwap (name::Component &comp)`` to provide way for memory-optimized component appending.

    .. method:: appendNumber (self, number)

    .. method:: appendNumberWithMarker (self, number, marker)

    .. method:: appendSeqNum (self, seqNum)

    .. method:: appendControlNum (self, controlNum)

    .. method:: appendBlkId (self, blkId)

    .. method:: appendVersion (self, version)

    :Accessors:

    In addition to standard :class:`vector` or :class:`list` accessors

    .. method:: toUri (self)

        Another variant in C++ ``toUri (std::ostream &os)``

        There should also be a language-specific of this method to convert name to URI.
        For example, in python it is :py:meth:`str` method and in C++ it is ``operator<<(ostream &,const Name&)``.

    .. method:: toWire (self)

    .. method:: get (self, index)

        Get binary blob of name component
        
        :type index: int
        :param index: index of the name component.  If less than 0, then getting component from the back:
                      get(-1) getting the last component, get(-2) is getting second component from back, etc.
        :returns: :py:class:`Blob` of the requested name component
  
        If index is out of range, an exception will be thrown
  
        In C++ const and non-const versions


    .. method:: getSubName (self, pos = 0, len = npos)

        Get a new name, constructed as a subset of components

        :type pos: int
        :param pos: Position of the first component to be copied to the subname
        :type len: int
        :param len: Number of components to be copied. Value Name::npos indicates that all components till the end of the name.
  
    .. method:: getPrefix (self, len, skip)

    .. method:: getPostfix (self, len, skip)

    .. method:: isPrefixOf (self, name)

    .. method:: compare (self, other)

        Implementation of appropriate ordering

        If possible, name.Component need to overload ``==``, ``<=``, ``<``, ``>=``, and ``>`` operations

Exclude
~~~~~~~

.. class:: Exclude

    Exclude filter abstraction.  

    The interface is ignorant about the underlying implementation (wire format) of the exclude filter, but the assumption is that we are using a simple form of exclude, with name components and ``<ANY/>`` tags to exclude ranges (no bloom filters).

    :Constructors:
    
    .. method:: __init__ (self)

        Create an empty exclude filter

    .. classmethod:: fromWire (cls, buffer)

    :Modifiers:

    .. method:: excludeOne (self, component)

        Exclude specific name component

        :type component: name.Component
        :param component: component to exclude
        :returns: self to allow chaining

    .. method:: excludeRange (self, from, to)

        Exclude components from inclusive range [from, to]

        :param from: first element of the range
        :param to: last element of the range
        :returns: self to allow chaining

    .. method:: excludeBefore (self, to)
  
        Exclude all components from range [/, to]

        :param to: last element of the range
        :returns: self to allow chaining
  
    .. method:: excludeAfter (self, from)
  
        Exclude all components from range [from, +Inf]

        :param to: last element of the range
        :returns: self to allow chaining

    :Accessors:

    .. method:: toWire (self)

    .. method:: isExcluded (self, component)

        :type component: name.Component
        :param component: Name component to check against the exclude filter

    Depending on the implementation, the underlying data structure (e.g., :py:class:`dict` or :cpp:class:`std::map`) can be exposed through a language specific interface.

SignedInfo
~~~~~~~~~~

    Abstraction for the data block, associated with Data packet, which is signed alongside the name and the content, but which is not part of the content itself: META info about the content and/or signature.

    :Constants:

    - .. data:: Content.BLOB
    
        Data packet contains content of unknown (application-specific format)  (aka ``CONTENT_DATA``)

    The following are the same as defined in CCNx document, but I'm not sure that we actually need to explicitly specify any of them.
    All of these define application-specific data, which is used (should be used) only in application (library=application).

    We can actually go beyond the type and use ContentType equivalent of HTTP, but again, this is app-specific.

    - .. data:: Content.ENCR
    - .. data:: Content.GONE
    - .. data:: Content.KEY
    - .. data:: Content.LINK
    - .. data:: Content.NACK

    :Constructors:
    
    .. method:: __init__ (self, type = Content.BLOB, freshness = None, finalBlock = None, timestamp = None)
    
        Create an empty exclude filter

        **Note** SignedInfo does not contain keyLocator and publicKeyDigest items, it is part of the Signature block
    
    .. classmethod:: fromWire (cls, buffer)

    :Modifiers:
    
    .. method:: setType (self, type)
    .. method:: setFreshness (self, type)
    .. method:: setFinalBlock (self, type)
    .. method:: setTimestamp (self, type)

    .. method:: setAppData (self, type, value)

        Set application-specific data to be bundled with Data packet, but which is not part of the content, e.g., HTTP ``ContentType``.

    :Accessors:
    
    .. method:: toWire (cls)

    .. method:: getType (self)
    .. method:: getFreshness (self)
    .. method:: getFinalBlock (self)
    .. method:: getTimestamp (self)
    .. method:: getAppData (self, type)

Error reporting
---------------

Most of the erros should be reported, if possible, in form of exceptions.  The following list of exception is currently defined:

.. exception:: error.Error (Exception)

    Generic NDN error (base class of other errors)

.. exception:: error.Uri

.. exception:: error.Name

.. exception:: error.name.Component

.. exception:: error.Exclude

.. exception:: error.KeyChain

*Others TBD*

NDN primitives
--------------

.. class:: Face

    Abstraction to create a communication channel an NDN daemon (local or remote).

    :Constructors:

    .. method:: __init__ (self, ioEventLoop, autoconnect = True)

        :param eventLoop: An object that represents processing thread and a link to the operating system's I/O services (should be similar to io_service in in `Boost.Asio <http://www.boost.org/doc/libs/1_54_0/doc/html/boost_asio/overview/core/basics.html>`_ )
        :type eventLoop: IoEventLoop

    :Operations:

    .. method:: connect (self, host = localhost)

    .. method:: disconnect (self)

    .. method:: enableAutoVerification (self, enable = True)

        Enable automatic verification of data packets according using either default or configured :py:class:`KeyChain`.

        **Enabled by default**. If disabled, an application needs to explicity call :py:meth:`KeyChain.verify` method for every received data packet.

    .. method:: getKeyChain (self)

        Get :py:class:`KeyChain` object, e.g., to sign Data packet.
        If :py:class:`KeyChain` has been previously set with :py:meth:`setKeyChain` method, the configured :py:class:`KeyChain` will be returned, otherwise the default key chain will be returned.

        :returns: :py:class:`KeyChain`

    .. method:: setKeyChain (self, keyChain)

        Set configured :py:class:`KeyChain` to use for signing/verification operations

        :param keyChain: configure key chain
        :type keyChain: KeyChain

    .. method:: expressInterest (self, interest, onData, onTimeout = None)

        Express interest for the data.
   
        There could be a form of this method (overloaded in C++ or with type check in python), accepting :py:class:`Name` as a first parameter (instead of interest).

        Multiple interests can be expressed for the same name.

        :param interest: Interest to express or simply name to request
        :type interest: :py:class:`Interest` or :py:class:`Name`
        :param onData: Callback to be called when data is received
        :type onData: :py:meth:`onData`
        :param onTimeout: Optional callback to be called when expressed Interest times out.  If the application requires Interest to be reexpressed, it need to explicitly call :py:meth:`Face.expressInterest` call again.
        :type onData: :py:meth:`onTimeout`
        :returns: **??** some identifier, so it is possible to cancel the interest, if necessary

        .. method:: onData (interest, data)

            :param interest: original Interest that was expressed
            :type interest: Interest
            :param data: received data packet
            :type data: Data
            :returns: nothing

        .. method:: onTimeout (interest)

            :param interest: original Interest that was expressed
            :returns: nothing

        In some implementation, e.g., C (and optionally C++, but not recommended), expressInterest should accept an optional :cpp:class:`void*` parameter that can be used to hold user data.

        .. Use of smart pointers (either non-intrusive :cpp:class:`shared_ptr` or intrusive :cpp:class:`intrusive_ptr` is highly encouraged.

        In C++ implementation callback parameter should be either of C++11 ``std::function``, ``boost::function``, or any other similar functor concept (e.g., ``Callback`` in ndnSIM).

    .. method:: cancelInterest (self, interest)

    .. method:: setInterestFilter (self, prefix, onInterest, flags = None)

        Set interest filter for a prefix (aka ``registerPrefix``)

        Multiple filters can be set up for the same prefix

        :param prefix: Name prefix to register filter for (e.g., create a local FIB entry, which may or may not propagate further)
        :type prefix: Name
        :param onInterest: Callback to be called when interest for the registered filter arrives
        :type onInterest: :py:meth:`onInterest`
        :param flags: **??**
        :returns: **??** some identifier, so it is possible to cancel the interest, if necessary

        .. method:: onInterest (prefix, interest)

            :param interest: original prefix set in filter
            :type interest: Name
            :param data: received Interest packet
            :type data: Interest
            :returns: nothing

    .. method:: clearInterestFilter (self, prefix)

    .. method:: put (self, data)
    
        ``Publish`` Data packet, either as a response to the incoming Interest or proactively publish to local cache, e.g., implementing prefetching under the expectation of more Interest to arrive.

        **May be a better name should be used instead**

.. class:: Interest

    Interest abstraction

    :Constructors:
    
    .. method:: __init__ (self, interest = None)

        Create either an empty interst or copy of the existing one

    .. method:: __init__ (self, name, minSuffixComponents = None, \
                          maxSuffixComponents = None, publisherPublicKeyDigest = None, \
                          exclude = None, childSelector = None, answerOriginKind = None, \
                          scope = None, interestLifetime = None, nonce = None)

        Create interest from name and any other optional parameter.

    .. classmethod:: fromWire (cls, buffer)

    :Modifiers:

    .. method:: setName (self, name)
    .. method:: setMinSuffixComponents (self, value)
    .. method:: setMaxSuffixComponents (self, value)
    .. method:: setPublisherPublicKeyDigest (self, value)
    .. method:: setExclude (self, exclude)
    .. method:: setChildSelector (self, value)
    .. method:: setAnswerOriginKind (self, value)
    .. method:: setScope (self, value)
    .. method:: setInterestLifetime (self, value)
    .. method:: setNonce (self, value)

    :Accessors:
    
    .. method:: toWire (self)

    .. method:: getName (self)
    .. method:: getMinSuffixComponents (self)
    .. method:: getMaxSuffixComponents (self)
    .. method:: getPublisherPublicKeyDigest (self)
    .. method:: getExclude (self)
    .. method:: getChildSelector (self)
    .. method:: getAnswerOriginKind (self)
    .. method:: getScope (self)
    .. method:: getInterestLifetime (self)
    .. method:: getNonce (self)

    Language specific accessors (e.g., ``__str`` in python and ``operator<<`` in C++) for debugging purposes.

.. class:: Data

    Data abstraction

    :Constructors:

    .. method:: __init__ (self, data = None)

        Create either an empty Data packet or copy of the existing one.
        The cached/signed wire format should be copied as well.

    .. method:: __init__ (self, name, content = None, signed_info = None)

        Create Data packet from name and any other optional parameter.

    .. classmethod:: fromWire (cls, buffer)

    :Modifiers:

    .. method:: setName (self, name)

    .. method:: setSignedInfo (self, signedInfo)

        :param signedInfo: Information about content and/or signature
        :type signedInfo: SignedInfo

    .. method:: setContent (self, content)

    .. method:: setSignature (self, signature)

        .. method:: _setSignedWire (self, wire)

            Signature module should have access to a private function to set Data packet wire format.

    :Accessors:

    .. method:: toWire (self)

        Get wire represenation of the Data packet.

        **Note** This method should fail (e.g., raise an exception) if Data packet has not been signed or not marked explicitly as unsigned.
        In other words, if :py:meth:`setSignature` was never called (implicitly or explicitly), signing should fail.

    .. method:: getName (self)

    .. method:: getSignedInfo (self)

    .. method:: getContent (self)

    Setting/accessing content of the data packet can be exposed as a language-specific buffer operation on Data packet structure.
    For example, Python can provide ``buffer (data)`` and C++ set of container operations, including ``operator[]``, ``begin``, ``end``, ``size``, and others.

    .. method:: getSignature (self)

        .. method:: _getUnsignedWire (self)

        Signature module should have access to this private method to get unsigned wire format of Data packet (everything else, except Signature block)

Helpers
-------

.. class:: IoEventLoop

    We can try to borrow Proactor async pattern from `Boost.Asio <http://www.boost.org/doc/libs/1_54_0/doc/html/boost_asio/overview/core/async.html>`_.

    (I partially copied some docs from Boost)

    .. image:: _static/io-event-loop.svg

    Not all platforms would require use of IoEventLoop, which can be either implemented (for the API compatibility) as a NOOP, or completely omitted.
    For example, JavaScript and NS-3 provide their own asyncrhonous event dispatchers.

    We can either borrow everything or some subset from `io_service <http://www.boost.org/doc/libs/1_54_0/doc/html/boost_asio/reference/io_service.html>`_ implementation:

    .. method:: __init__ (self)

    .. method:: run (self)

    The run() function blocks until all work has finished and there are no more handlers to be dispatched, or until the IoEventLoop has been stopped.

    .. method:: runOne (self)

    The runOne() function blocks until one handler has been dispatched, or until the io_service has been stopped.

    .. method:: stop (self)

    This function does not block, but instead simply signals the io_service to stop. 
    All invocations of its run() or run_one() member functions should return as soon as possible. Subsequent calls to run(), runOne(), poll() or pollOne() will return immediately until reset() is called.

    .. method:: dispatch (self, callback)

    This function is used to ask the IoEventLoop to execute the given handler.

Security primitives
-------------------

.. class:: KeyChain

    .. method:: __init__ (self, Face)

    Some KeyChain operations require NDN communication (e.g., :py:meth:`verify`) and are asynchronous (Face will provide IoEvenLoop for that).

    .. method:: getPolicyManager (self)

    .. method:: createIdentity (self, identityName, ...)

    .. method:: generateKeyPair (self, keyName, ...)

        :param keyName: requested key name for the generated pair. If keyName is identityName, then the actual key name will be automatically generated.
        :returns: name

    .. method:: generateRsaKeyPair (self, keyName, numBits)

    .. method:: createSigningRequest (self, keyName)

    .. method:: installCertificate (self, certificateDataPacket)

    .. method:: setDefaultIdentity (self, name)
    .. method:: setDefaultKey (self, name)
    .. method:: setDefaultCertificate (self, name)

    *Other methods TBD*

    .. method:: _sign (self, buffer, certName)

        Sign the supplied ``buffer`` with a key identified by ``certName``.

        This version may or may not be private.

        :param buffer: Buffer to sign
        :type buffer: bytes
        :param certName: Name to identify key/certificate to use for signing the supplied buffer
        :type certName: Name
        :returns: The appropriate instance of :py:class:`Signature`, holding signature bits and (if necessary) associated information, such as ``KeyLocator``, ``Witness``, and other.

        C++ version may operate on ``std::istream`` in addition (or instead) memory buffer.

    .. method:: sign (self, data, certName = None)
    
        A helper method to create fully formatted and signed Data packets.
        If ``certName`` is not specified, it will be guessed using :py:class:`policy.Sign`

        :param data: Data packet
        :type data: Data
        :param certName: Optional name to identify key/certificate to use for signing the supplied buffer
        :type certName: Name

        This method should be roughly equivalent to the following operations:
        
        .. code-block:: python

            def sign (self, data, certName = None):
                unsignedBuffer = data._toUnsignedWire ()
                certName = self.getPolicyManager ().getSigningCertName (data.getName ())
                signature = self.sign (unsignedBuffer, certName)
                data.setSignature (signature)

        The following example demonstrates how data packets can be created, signed, and published:

        .. code-block:: python

            face = Face ()
            data = Data ()
            data.setName (Name ("/hello/world"))
            data.setSignedInfo (SignedInfo (freshness = 1.0))
            data.setContent ("Hello, World!")

            face.getKeyChain ().sign (data)
            face.put (data)

    .. method:: _verify (self, buffer, signature)

        This version may or may not be private.

        :param buffer: The buffer to verify
        :type buffer: bytes
        :param signature: Signature object, e.g., :py:class:`signature.Sha256WithRsa`, containing an appropriate signature of the supplied buffer
        :type signature: Signature

    .. method:: verify (self, data, onVerify, onVerifyError = None)

        The helper method to verify data packets (asynchronous operation)

        :param data: Data packet
        :type data: Data

        :param onVerify: Callback to be called when data is verified
        :type onVerify: :py:meth:`onVerify`
        :param onVerifyError: Optional callback to be called when verification fails
        :type onVerifyError: :py:meth:`onVerifyError`

        .. method:: onVerify (data)

            :param data: original data packet
            :type data: Data
            :returns: nothing

        .. method:: onVerifyError (data, error)

            :param data: original data packet
            :type data: Data
            :param error: Error message (or error code)
            :returns: nothing

        Usage example:

        .. code-block:: python

            def onVerify (data):
                print "Huraaaay! %s" % data.getName ()

            def onData (interest, data):                
                face.getKeyChain ().verify (data, onVerify)
                # all non-verifiable packet will be silently dropped

.. class:: Signature

    Base class for NDN signatures (different types of signatures are possible).  Currently define the following:

    .. - **Signature.Empty**
    .. - **Signature.SHA256**

    - **Signature.SHA256_WITH_RSA**
    - **Signature.MERKLE_HASH_WITH_RSA**

    .. method:: getType (self)

        Get signature type

    .. .. class:: signature.Empty

    ..     Empty signature, no real signing is performed (e.g., for debug or some other purposes)

    ..     *TBD*

    .. .. class:: signature.Sha256

    ..     Signature that provides just data integrity via hashing
        
    ..     *TBD*

    .. class:: signature.Sha256WithRsa

        The standard signature that is currently used in CCNx

        .. method:: setKeyLocator (self, certName)

        .. method:: setPublicKeyDigest (self, digest)

            **This may not be necessary, unless there is a good reason to keep it**

        .. method:: setSignatureBits (self, bits)

        .. method:: getKeyLocator (self)

        .. method:: getPublicKeyDigest (self)

            **This may not be necessary, unless there is a good reason to keep it**

        .. method:: getSignatureBits (self)

    .. class:: signature.MerkleHashWithRsa

        The aggregated signature

        *TBD*

.. class:: PolicyManager

    .. method:: setVerificationPolicy (self, policy)

    .. method:: setSigningPolicy (self, policy)

.. class:: policy.Verify

    .. class:: policy.verify.Identity

        .. method:: doesComply (dataPacket)

.. class:: policy.Sign

    .. class:: policy.sign.Identity

        .. method:: doesComply (dataPacket, certName)

        .. method:: getComplyingCertName (dataPacket)
