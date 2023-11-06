### SOLID
Solid is a way to write good code. It is sort of guidance points.
- **Single Responsible Principle** 
    - A class focused, single functionality
    - A class addresses a specific concern
    - There should **never be more than one reason for a class to chnage**:
    ```
    Example:
    Lets say there is a server and super cool class that is communicating with this server. It has:
    - Json formatter
    - Http connection manager
    - And some authentication method

    But what if the connection is no longer an http?
    And tomorrow it is only XML?
    And the authentication suddenly is different?

    All this will lead to CHANGES IN THIS CLASS.
    ```
- **Open - Close Principle**
    - Software entities (classes, modules, methods, etc...) Should be **OPEN for extension, but CLOSE for modification**.
        - The Base code is already **WRITTEN AND TESTED**!
        - Also the **Base code is opened for extension**, and hence the new code car override it.