## Debugging Integration Tests

### Some useful things

### Resources

When debugging in IntelliJ, and requiring resources (`.getResources(nameOfResource)`), 
there is a high probability (always, in my experience), that the debugging thread
will read the application-class' resources in the root of the `out/` directory.

Normally, when debugging an application, you will have to supply any necessary resources 
as arguments (`server some-application.yaml`). In those cases having something like this is fine:
 ```
 ApplicationClassIntegrationTest.class.getResources('/some-resource.yaml')
 ```
This would also be fine if you are running a test-runner like Gradle to do your testing,
as it will have a much wider access to the package's resources on the filesystem.

But that's not the case when debugging (integration) tests (`JUnit`).
If you put a breakpoint on the same line like above, you will see that the 
the instance of `ApplicationClassIntegrationTest.class` no longer is in your 
package's source code, but rather in the package's `out/` directory.
E.g.: `out/com/organisation/developer/package/ApplicationClassIntegrationTest`.

As such, the resources that your integration tests needs, have to exist in that folder. So just move them across!

**Note**: There is a "/" before resources, which tells DropWizard that the file isn't in the same directory as the 
java module (`ApplicationClassIntegrationTest`).
For the above *fix* to work, you will have to remove that "/" such that
`getResources('/some-resource.yaml')` becomes `getResources('some-resource.yaml')`.
