# Deserialization RCE

## Basic Information
- **Serialization** is understood as the method of converting an object into a format that can be preserved, with the intent of either storing the object or transmitting it as part of a communication process. This technique is commonly employed to ensure that the object can be recreated at a later time, maintaining its structure and state.

- **Deserialization**, conversely, is the process that counteracts serialization. It involves taking data that has been structured in a specific format and reconstructing it back into an object.

- Deserialization can be dangerous because it potentially allows attackers to manipulate the serialized data to execute harmful code or cause unexpected behavior in the application during the object reconstruction process.

## What is Insecure Deserialization?
Insecure deserialization is when user-controllable data is deserialized by a website. This potentially enables an attacker to manipulate serialized objects in order to pass harmful data into the application code. It is even possible to replace a serialized object with an object of an entirely different class. Alarmingly, objects of any class that is available to the website will be deserialized and instantiated, regardless of which class was expected. For this reason, insecure deserialization is sometimes known as an "object injection" vulnerability.

An object of an unexpected class might cause an exception. 
By this time, however, the damage may already be done. Many deserialization-based attacks are completed before deserialization is finished. 
This means that the deserialization process itself can initiate an attack, even if the website's own functionality does not directly interact with the malicious object. 
For this reason, websites whose logic is based on strongly typed languages can also be vulnerable to these techniques.

## What is the Impact of Insecure Deserialization?
The impact of insecure deserialization can be very severe because it provides an entry point to a massively increased attack surface. It allows an attacker to reuse existing application code in harmful ways, resulting in numerous other vulnerabilities, often remote code execution. Even in cases where remote code execution is not possible, insecure deserialization can lead to privilege escalation, arbitrary file access, and denial-of-service attacks.

## Conceptual Mechanics of Exploitation
In security research and code analysis, understanding how insecure deserialization leads to vulnerabilities involves looking at how execution flow is intercepted during object reconstruction:

1. **Leveraging Built-in Language Methods (Magic Methods / Hooks):** Many programming languages automatically invoke specific methods during the lifecycle of an object, such as during its creation or destruction. 
If an application deserializes untrusted data into a class that utilizes these automatic methods, and those methods contain logic that interacts with system resources (like files or database connections), the altered properties of the injected object can divert the logic to perform unauthorized actions.

2. **Utilizing Existing Code Sequences (Gadget Chains):** Often, a single class cannot achieve arbitrary code execution on its own. 
Instead, analysis focuses on identifying a chain of method calls across various libraries available within the application's environment. 
The input is structured so that the first object invokes a method on a second object, which in turn invokes a method on a third, ultimately leading to a sink function that executes system commands or manipulates critical application logic.

