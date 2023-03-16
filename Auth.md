> “Passwords are like underwear: don’t let people see it, change it very often, and you shouldn’t share it with strangers.” – Chris Pirillo

There are multiple ways of doing auth, but in this chapter, we'll focus on the Single sign-on approach.

## Single sign-on (SSO)

Single Sign-On (SSO) is a method of authentication that allows users to access multiple systems and applications with a single set of login credentials. SSO process typically starts when a user attempts to access a protected resource. Instead of prompting the user to enter their login credentials, the application redirects the user to an SSO service, which authenticates the user by checking their login credentials against an identity provider, such as a directory service or a database. Once the user is authenticated, the SSO service will generate a token, which is used to represent the user's identity.

The token is usually a JSON Web Token (JWT) which contains claims that are encoded with the user's identity, as well as other information such as the expiration date of the token. The token is then sent back to the application, which uses it to grant the user access to the protected resource.

The tokens are usually signed by the SSO service using a private key, which ensures that the token has not been tampered with, and can be verified using a public key. The tokens can also be encrypted, to ensure that the claims inside the token are not visible to anyone other than the intended recipient.

The SSO service also acts as a central location for managing and auditing access to all the systems and applications that use SSO. It allows IT administrators to manage and secure access, and with the use of token, it can also check for the token's expiration date, and the scopes and audiences it's intended for.

### Benefits

There are several benefits to using Single Sign-On (SSO) for authentication:

-  Improved User Experience: SSO eliminates the need for users to remember and manage multiple sets of login credentials, making it easier and more convenient for them to access the systems and applications they need.
-  Increased Security: SSO reduces the risk of password reuse and sharing, which can lead to security breaches. Additionally, SSO can also improve security by requiring users to go through a stronger authentication process, such as multi-factor authentication, before granting access.
-  Increased Productivity: SSO can save users time by eliminating the need to constantly log in and out of different systems. This can also improve productivity by allowing users to quickly access the resources they need to do their jobs.
-  Reduced IT Costs: SSO can reduce the costs associated with managing multiple sets of login credentials and resetting forgotten passwords.
-  Better Compliance: SSO can help organizations meet compliance requirements by providing a centralized location for managing and auditing access to sensitive information.

### Concerns

Implementing Single Sign-On (SSO) can present a number of challenges, such as managing user profile data, handling the creation and deletion of users, and ensuring the security of credentials. Among these challenges, the issue of keeping credentials safe is particularly important.

In this chapter, we will focus on addressing the concern of keeping credentials safe when implementing SSO in a JavaScript application. We will explore best practices and strategies for securing the SSO process, including handling and storing tokens securely, preventing cross-site scripting (XSS) and cross-site request forgery (CSRF) attacks, and using secure communication protocols. By understanding and addressing the issue of keeping credentials safe, we can ensure that the SSO implementation is secure and reliable for both users and IT administrators.

When it comes to saving credentials in the browser, HttpOnly Cookies are often the preferred method as they are not vulnerable to cross-site scripting (XSS) attacks. However, when using Single Sign-On (SSO), the credentials are usually provided in the form of tokens that are intended to be sent via the Authorization header.

While it may be tempting to simply store these tokens in the browser's localStorage, this can introduce security risks if any third-party code is present or if a user is able to add custom JavaScript to the application. Storing the tokens in regular Cookies may also not be the best solution as it defeats the purpose of using Cookies in the first place. In light of this, it's important to find a better and more secure way of handling and storing tokens in a JavaScript application when implementing SSO.

### How to do it right

The specification for Single Sign-On (SSO) provides general guidance on how to implement the process securely. However, in this chapter, we aim to take a more practical approach. We will provide concrete examples and specific steps for ensuring the security of credentials when implementing SSO in a JavaScript application.

#### With server-side rendering

When we're working on an app that might do API calls fro the server (e.g. fetching data for the server-side render), we need to have the token available on the server, which means that the cookie is the only option. In this case, we can use the approach the specification calls ["backend for frontend"](https://www.ietf.org/archive/id/draft-ietf-oauth-browser-based-apps-10.html#section-6.2). In general, that means that our backend is the SSO client and it should create a separate session for our client code:

1. Log in with a SSO provider
2. The backend logic handles the redirect URL, gets the tokens based on the activation code
3. The backend creates a session for the user and saves the tokens in the session
4. The backend redirects the user to the client app
5. Each time an authorized API call needs to be made, it needs to go trough our backend proxy, which will get the tokens, refresh them if necessary and then forward the API call with all the necessary auth headers. Those API calls should also include the CSRF tokens to prevent the CSRF attacks.

#### With client-side rendering

When all API calls are made from the browser, we can utilize a [proxy service worker](https://www.ietf.org/archive/id/draft-ietf-oauth-browser-based-apps-10.html#section-6.3.2) to intercept our requests. This approach is similar to using a backend proxy in the server-side rendering example, as it allows us to handle the SSO flow in the browser.