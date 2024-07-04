# Testing Concepts

Unit test is a good negative indicator - it points out poor quality with high accuracy. If you find it's difficult to implement unit test, it is a sign that your code requires improvement. It's also a bad positive indicator - just because you can unit test your code doesn't mean it's a good code. The goal of unit test, then is to create your code to be sustainable throughout your project. It's looking for long-term benefit, despite some short-term downfall.

Testing act as an insurance policy. It mitigates regression of your software quality by notifying you if there is any. A good software increases scalability and sustainability in the long run.

Coverage metrics can't possibly tell you that your test are complete, nor it can tell you that you have enough. It's just a metric, and that's important fact to remember: do not implement a policy to meet specific criteria. A successful test suit has the following properties:

- It is integrated into your development cycle
- It targets only the most important part of the code
- It provides maxmimum value with minimal maintenance cost

Act Phase: checks the behavior you want to verify. Watch out if your act section is larger than two or more line. This is an indication that there's problem with the code.

Test Double: includes all non-production ready, set of dependencies for the test
Mocks: only one kind of dependency

Do not use constructors for test fixtures. Create a separate private method for initializing common objects and variables and then

## Definitions

- Code Coverage: shows how much source code the test executes, usually done in percentage. Like unit tests, code coverages are negative indicator; higher does not mean it's good code. However, low coverage can mean it's not covering enough.
