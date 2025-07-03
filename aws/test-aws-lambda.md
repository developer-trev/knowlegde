## How to test AWS Lambdas written in NodeJS (Javascript)

### 1. Separate Business Logic from AWS Lambda Handler
Why? Keeps your logic testable without needing to mock AWS infrastructure.

How?
Put your core functionality into separate modules/functions.

The Lambda handler just calls these functions with the event data.

Example:

```
// businessLogic.js
function processData(data) {
  // core logic here
  return data * 2;
}
```

module.exports = { processData };

```
// handler.js
const { processData } = require('./businessLogic');

exports.handler = async (event) => {
  const result = processData(event.input);
  return {
    statusCode: 200,
    body: JSON.stringify(result),
  };
};
```

### 2. Use Testing Frameworks
Popular choices:
- Jest (most popular for Node.js)
- Mocha + Chai
- AVA

Example with Jest:

```
const { processData } = require('./businessLogic');

test('processData doubles the input', () => {
  expect(processData(2)).toBe(4);
});
```

### 3. Mock AWS Services
If your Lambda interacts with AWS SDK (DynamoDB, S3, etc.), mock those SDK calls.

Tools for mocking:

jest.mock() for mocking modules

aws-sdk-mock (though it’s less maintained, still useful)

aws-sdk-client-mock for v3 AWS SDK clients

Example (using jest.mock):

```
const AWS = require('aws-sdk');
jest.mock('aws-sdk', () => {
  return {
    DynamoDB: jest.fn(() => ({
      putItem: jest.fn().mockReturnValue({
        promise: () => Promise.resolve({}),
      }),
    })),
  };
});
```

### 4. Test Lambda Handler Directly
- Call the handler function with mock event and context objects.
- Use async/await if your handler returns a promise.

Example:

```
const { handler } = require('./handler');

test('handler returns expected response', async () => {
  const event = { input: 2 };
  const response = await handler(event);
  expect(response.statusCode).toBe(200);
  expect(JSON.parse(response.body)).toBe(4);
});
```

### 5. Use Local Environment Variables
Use dotenv or pass environment variables in your test setup to mimic Lambda env vars.

### 6. Avoid Real AWS Calls
- Never call real AWS services during unit tests. Use mocks/stubs.
- Integration tests or end-to-end tests can verify real AWS calls separately.

### 7. Test Edge Cases & Errors
- Simulate errors thrown by AWS SDK or your logic.
- Check if your Lambda handler handles them gracefully (returns proper error codes/messages).

### Bonus Tips
- Use tools like Serverless Framework or AWS SAM CLI for easier local testing and simulation.
- Keep your tests fast and isolated.
- Test individual units rather than full deployments (leave deployment testing for integration tests).