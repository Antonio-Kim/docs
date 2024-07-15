## API Testing using Postman

Assuming you have created an account to login to Postman and save collections, you can do the following to make your life easier:

- variables: instead of copy-pasting Id, account, password, API tokens, etc., you can save it to Postman's Collection Variables and share with other requests. Go to Collection->Variables and add the key and value of the variable and its value, respectively, to the table. If you want use the variable, wherever you want to use it, add `{{tokenId}}` as an example. Ensure that you either set variables manually or by script. Most of the time you will want to set variables by scripts.
- Scripts: there are two sections of the script: pre-request, and post-response. Majority of the cases you will be writing your test on Post-response. Most of the Postman's script starts with `pm` object, but everything else is very similar to Jest/Mocha test framework. Here's an example of running status code 200 test:

```js
pm.test('Status code is 204', () => {
  pm.response.to.have.status(204);
});
```

Here are more useful tests with little bit of refactoring

```js
console.clear();
const response = pm.response.json();

pm.test('Status code is 200', () => {
  pm.response.to.have.status(200);
});

pm.test('Response is an object', () => {
  pm.expect(response).to.be.an('object');
  pm.expect(response).to.haveOwnProperty('id'); // check if object has an key of id
});

pm.test('Product name', () => {
  pm.expect(response.name).to.be.a('string');
});

pm.test('Product price is a number', () => {
  pm.expect(response.price).to.be.a('number');
  pm.expect(response.price).to.be.above(0);
});

pm.test('Product in in stock', () => {
  pm.expect(response.inStock).to.be.true;
});
```

You can set different folders for each type of tests, and inside the folder you can set scripts that runs on all the test inside the test. For example, `const response=pm.repsonse.js()` can be set at the folder level since it will be run on every test.

You can set variables for the collection using `pm.collectionVariables.set('key', 'value');`. You can also get variables for your test by `pm.collectionVariables.get('key')`. To remove the variable, we use `pm.collectionVariables.unset('key')`

### Environments

you can set different environments for testing, and have tests run in different environment as well. For example, some tests will not work on production (nor should you test it in production), but will work on dev or test environment. You can check your environment inside the script by using `pm.environment.name`. This can be useful when you want to run specific tests inside a specific environment. For example, the following code will only run inside "Testing" environment:

```js
if (pm.environment.name === 'Testing') {
  pm.test('Status code is 200', () => {
    pm.response.to.have.status(200);
  });
}
```

However, you cannot select environment inside script. If there's no environment, the `pm.environment.name` will be undefined.

Just like collection, you can also set and get variables using `pm.environment.set("key", "value")` and `pm.environment.get("key")`, and also remove an environment variable by `pm.environment.unset("key")`

Exporting and importing environment file (.json) will not export Current value; only the initial values are exported. API and environments are separate and must be exported and import separately

There are also global variables that are available in all collections and all environments. Unsurprisingly, you can set global variable with `pm.globals.set("key", "value")`, get global variable by `pm.globals.get("key")`, and remove an environment vaible by `pm.gloabls.unset("key")`
