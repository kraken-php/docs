# About the Documentation

- [Documentation](#documentation)
    - [Status](#status)
    - [Intentions](#intentions)
    - [Syntax and Conventions](#syntax-and-conventions)

<a name="documentation"></a>
## Documentation

<a name="status"></a>
### Status

This documentation is community-driven and will evolve together with Kraken as the code will be further developed and changed. Pages may be deleted or moved without notice between Kraken releases. If you run across any issues, please create an issue on [GitHub](https://github.com/kraken-php/docs) describing the problem or submit a pull request.

<a name="intentions"></a>
### Intentions

Main intention for this documentation is to provide enough information for developer that uses Kraken Framework to be able to grasp fundamental concepts and learn about most important features. After reading this documentation developer should be able to create basic application using Kraken Framework and be able to browse the code and learn about advanced features for PHPDoc. This documentation does not contain all information and never will.

<a name="syntax-and-conventions"></a>
### Syntax and Conventions

This documentation uses several conventions to help explain concepts and document features. 

*Italic* and **bold** type faces are used to provide emphasis. `Monospace` text is used for class names, function names, variable names and paths. It is also used for console commands and terminal return values.

Note boxes will contain supplementary information set apart from the text.

> {notice} This is note box.

Tip boxes will offer helpful advices on code usage and best practices.

> {tip} This is tip box.

Warning boxes will help you avoid any problems or unexpected behaviours.

> {warning} This is warning box.

Prototypes for class methods are described using the following syntax:

    static|final|void ClassOrInterfaceName::methodName(ArgumentType $arg = 'default-value') : ReturnType

To document the expected prototype of a callback function used as method arguments or return types following syntax is applied:

    callable(ArgumentType $arg) : ReturnType

Possible events thrown by any class are document via following syntax:

    event eventName : ReturnType 

Documentation for methods inherited from a parent class or implementing an interface are not re-documented unless there is a significant difference in behavior.
