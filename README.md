# AWS Lambda HelloWorld with DynamoDB Config

This project for `helloWorldFunction` contains the lambda request handler for replying with a hello response to a name parameter, a context wrapper class and some unit tests.

The response is formatted into HTML using a Mustache template defined in `src/main/resources/hellopage.mustache`. To return turn the HTML string from the lambda to users as a proper HTML response, use an API Gateway endpoint mapping the request parameter and returning the response with a body mapping template to set a response type of 'text/html'.

The `ContextWrapper` class is used to wrap the lambda execution context class with a few convenience methods for determining the alias and version of the lambda, and from inferring the execution environment from the function name assuming a naming convention like `helloWorldFunction_dev` or `helloWorldFunction_prd`.

Configuration is stored in a DynamoDB to which the lambda must be given IAM access permissions, and assumes a configuration table naming convention like `helloWorldFunctionConfig`.

When executing, the lambda depends on a DynamoDB table entry for the environment keyed on `environmentId` and containing a string property `responseLanguage`.

See accompanying blog post for details {TBC}

### To run the lambda

Set up the DynamoDB table and config entries as described above.

Using Gradle 2.9 or higher, `gradle build` will package the jar and dependencies as a zip file for upload.

Upload the lambda using a function name matching the pattern above. 192MB should be enough memory given the size of the lambda, and set the lambda timeout to 5s. Lower memory may result in OutOfMemoryError and will likely reduce in greater likelihood of first invocation timeout, as memory allocation is directly proportional to compute resources.

**On first cold invocation the lambda will sometimes time out on retrieving a value from DynamoDB but the next warm invocation should return within the expected timeout.** This is something to keep an eye on - I haven't played around with the timeout enough to determine which value would prevent the first cold invocation from timing out.

Subsequent warm invocations will cache the config value in a class-local variable, so if you want to adjust the config value in DynamoDB live you'll need to deploy a new version or let the current one go cold.
