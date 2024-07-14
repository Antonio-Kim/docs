## API Testing using Postman

Assuming you have created an account to login to Postman and save collections, you can do the following to make your life easier:

- variables: instead of copy-pasting Id, account, password, API tokens, etc., you can save it to Postman's Collection Variables and share with other requests. Go to Collection->Variables and add the key and value of the variable and its value, respectively, to the table. If you want use the variable, wherever you want to use it, add `{{tokenId}}` as an example
- Scripts: there are two sections of the script: pre-request, and post-response. Majority of the cases you will be writing your test on Post-response. Most of the Postman's script starts with `pm` object, but everything else is very similar to Jest/Mocha test framework. Here's an example of running status code 200 test:

```js
pm.test('Status code is 204', () => {
  pm.response.to.have.status(204);
});
```
