


# The following 12 best practices can help expand and elevate the security of an organization's APIs:
---

## 1. Authenticate and authorize
---

   - To control access to API resources, you must carefully and comprehensively identify all related users and devices. This typically requires client-side applications to include a token in the API call so that the service can validate the client.

   - Use standards such as OAuth 2.0, OpenID Connect and JSON web tokens to authenticate API traffic and to define access control rules or grant types that determine which users, groups and roles can access specific API resources.

   - Always follow POLP. If a user just needs to read a blog or post a comment, those are the only permissions that should be assigned.

## 2. Implement access control
---


   - Organizations that want to enable third parties to access internal data and systems through APIs must introduce and test controls to manage that access: who, what and when, as well as checks on data access, creation, update and deletion -- the zero-trust security model.

   - Keep APIs behind a firewall, web application firewall or API gateway -- accessed through a secure protocol, such as HTTPS -- to provide baseline protection, such as scanning for signature-based threats and injection-based attacks.

   - Well-designed APIs can also apply rate limits and geo-velocity checks, as well as act as an enforcement point for policies such as geo-fencing and I/O content validation and sanitization. Geo-velocity checks provide context-based authentication by determining access based on the speed of travel required between the previous and current login attempts.

   - All these checks are applied by middleware code that's part of the API application. Middleware handles requests before passing them on to be fulfilled.

## 3. Encrypt requests and responses
---


   - All network traffic should be encrypted -- particularly API requests and responses, as they'll likely contain sensitive credentials and data. All APIs should use and require HTTPS. Enabling HTTP Strict Transport Security where possible is better than redirecting HTTP traffic to HTTPS, as API clients may not behave as expected.

## 4. Validate the data
---

   - Never assume API data has been cleansed or validated correctly. Implement your own data cleaning and validation routines server side to prevent standard injection flaws and cross-site request forgery attacks. Debugging tools such as Postman and Chrome DevTools can help examine the API's data flow and track errors and anomalies.

## 5. Assess your API risks
---

   - Another important API security best practice is to perform a risk assessment for all APIs in your existing registry. Establish measures to ensure they meet security policies and are not vulnerable to known risks. The Open Web Application Security Project's "API Security Top 10" vulnerabilities list is a good resource for keeping tabs on existing attacks and malicious software.

   - A risk assessment should identify all systems and data affected if an API should be compromised, and then outline a treatment plan and the controls required to reduce any risks to an acceptable level.

   - Document review dates and repeat assessments whenever new threats arise or the API is modified. This documentation should be reviewed prior to any subsequent code changes to ensure security and data-handling requirements aren't compromised.

## 6. Share only necessary information
---


   - API responses often include an entire data record rather than just the relevant fields, relying on the client application to filter what a user sees. This is lazy programming, and it not only slows response times but also provides attackers with additional information about the API and the resources it accesses.

   - Responses should contain the minimum information necessary to fulfill a request. For example, if an employee's age is requested, the date of birth shouldn't be returned as well.

## 7. Choose your web services API
---


   - There are two dominant options to access web services via APIs: Simple Object Access Protocol (SOAP), a communications protocol; and the Representational State Transfer API (REST API or RESTful API), a set of architectural principles for data transmission. They use different formats and semantics and require different strategies to ensure robust security.

   - SOAP security is applied at the message level using digital signatures and encrypted parts within the XML message itself. REST relies heavily on access control rules associated with the API's universal resource identifier, such as HTTP tags and the URL path.

   - Use SOAP if your primary concerns are standardization and security. While both options support Secure Sockets Layer/Transport Layer Security (SSL/TLS), SOAP also supports Web Services Security, identity verification through intermediaries rather than just point-to-point verification provided by SSL/TLS, and built-in error handling. However, SOAP exposes components of application logic as services rather than data, which can make SOAP complex to implement and might require an application to be refactored.

   - REST, meanwhile, is compatible with various data output types -- including JSON, comma-separated values and HTTP -- while SOAP can only handle XML and HTTP. In addition, REST merely accesses data, so it's a simpler way to access web services. For these reasons, organizations often prefer REST for web development projects. However, security must be built in for data exchanges, deployment and interaction with clients.

## 8. Record APIs in an API registry
---


   - Nobody can secure what they don't know. It's therefore essential to record all APIs in a registry to define characteristics such as its name, purpose, payload, usage, access, live date, retired date and owner. This will avoid shadow or silo APIs that have been forgotten, never documented or developed outside of a main project, possibly through mergers, acquisitions, or test or deprecated versions.

   - Record details of the information to be logged -- such as who, what and when -- which will help meet compliance and audit requirements, as well as aid forensic analysis in the event of a security incident.

   - Good documentation is particularly important for third-party developers who wish to incorporate those APIs into their own projects. The API registry should include links to the document or manual that contains all technical API requirements, including functions, classes, return types, arguments and integration processes.

## 9. Conduct regular security tests
---


   - In addition to thoroughly testing APIs during development, security teams need to regularly check the security controls protecting live APIs to ensure they're functioning as expected and behaving as documented.

   - Incident response teams should create a plan to handle the alerts produced by threat detection and other security controls that indicate an API attack.

## 10. Stash your API keys
---


    - API keys identify and verify access for the application or site that calls an API. They can also block or throttle calls made to an API and identify usage patterns.

    - API keys are less secure than authentication tokens and require careful management. Avoid embedding API keys directly in their code or in files within the application's source tree, where they can be accidentally exposed. Instead, store them in environment variables or in files outside of the application's source tree. Better still, use a secrets management service, which protects and manages an application's API keys.

    - Even with these measures in place, always delete unneeded keys to minimize exposure to attack, and periodically regenerate keys -- particularly if you suspect a breach has occurred.

## 11. Add AI to API monitoring and threat detection
---

    - AI-enabled behavior analysis can greatly improve overall API security. It benchmarks normal API traffic and provides visibility into how users access and consume APIs, which can help developers fine-tune threshold settings for context security checks. Threat detection tools can use this information to look for anomalous behavior to flag or to stop misuse or a potential attack.

    - Attackers must probe an API repeatedly to find vulnerabilities or logic they can exploit, so real-time monitoring is essential for attack detection and response. This approach requires no predefined policies, rules or attack signatures, so it reduces the need for constant updates to stop new and evolving attacks.

## 12. Understand the full scope of secure API consumption
---


    - API security also covers the third-party APIs it consumes. Before you build an application or service that handles third-party data via APIs, you must fully understand how they work and the correct way to integrate them.

    - Read API documentation thoroughly, paying attention to the process and security aspects of the API's function and routines, such as required authentication, call processes, data formats and any potential error messages to expect. One good approach to this is to build a threat model to help you understand the attack surface, identify potential security issues and incorporate appropriate security mitigations from the beginning.

    - APIs create countless opportunities for organizations to improve and deliver services, engage customers, and increase productivity and profits -- but only if you securely implement them.
