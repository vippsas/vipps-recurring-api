Vipps Recurring will soon (tm) come with the option to request and receive
access to certain personal details belonging to the user.
For example: Name or Address.
Certain personal data, like SSN, is considered extra sensitive and the merchant
needs to contact Vipps to prove that they have a legitimate use case for such
data.

 Resources:
 Recurring Swagger.
 Flow Diagram.

In summary the flow is

1. Add the optional paramater "scope: personaldetails" to the initial request.

2. You will then retrieve a URL "userprofileUrl" in the synchronous response.

3. When the user has approved in the app you can do a GET  on that URL as if it
was any other resource.

4. Retrieve the Users personal information.

5. Store safely and responsibly.
