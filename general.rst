General code guidelines
=======================

Type, class, function, method, and variable name naming convention
------------------------------------------------------------------

- Type and class names should follow the **CamelCase** convention: words are joined without spaces and are capitalized.
  Functions, methods, and variables should follow **camelBack** (differs from CamelCase by initial lowercase character!).

  The goal of the CamelCase and mixedCase conventions is to ensure that the words which make up a name can be separated by the eye: the initial Caps fills that role. 

- Defined types may optionally end with a ``_t``. 

- For example, "key locator" is transformed into **KeyLocator** type, operation of getting key locator to **getKeyLocator** method, and variable storing key locator to **keyLocator**.

- Do not use all capital letters such as PEM or, DER, but choose instead Pem (pem) or Der (der). 
  Do not use all capital letters, even for acronyms such as EDCA: use Edca (edca) instead. 
  This applies also to two-letter acronyms, such as IP (which becomes Ip or ip). 

- Defined constants (such as static const class members, or enum constants) will be all uppercase letters or numeric digits, with an underscore character separating words. 

  Otherwise, the underscore character should not be used in a variable name. 

Choosing names
--------------

Variable, function, method, and type names should be based on the English language, American spelling. 
Furthermore, always try to choose descriptive names for them. 
Types are often english names such as: Interest, Data, Key, or KeyLocator. 
Functions and methods are often named based on verbs and adjectives: getX, clearArray, etc.

A long descriptive name which requires a lot of typing is always better than a short name which is hard to decipher. 
Do not use abbreviations in names unless the abbreviation is really unambiguous and obvious to everyone (e.g., use ``size`` over ``sz``). 
Do not use short inappropriate names such as foo, bar, or baz. 
The name of an item should always match its purpose. 
As such, names such as ``tmp`` to identify a temporary variable or such as ``i`` to identify a loop index are ok.

If you use predicates (that is, functions, variables or methods which return a single boolean value), prefix the name with ``is`` or ``has``.


