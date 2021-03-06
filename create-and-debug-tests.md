# Create and Debug Tests
Testing can be a complex system itself. Depending on your test case, you may need to set up data before running each test or the test suite. A setup step can be creating a test user. Another step may be the cleanup after test runs. This document shows you how to create useful tests in Supercharge.


## Create Tests
Supercharge comes with a `base-test` utility that is a class you can build on. The suggested way to create tests in Supercharge is to write a class that extends the `base-test`. A base test example looks like this:

```js
const BaseTest = require('@supercharge/framework/base-test')

class TestCase extends BaseTest {
  /**
   * A passing test example.
   */
  async testMethod (t) {
    t.pass()
  }
}

module.exports = new TestCase()
```

Each test method accepts the `t` parameter from AVA. Use `t` for assertions or to store context data for the test suite.

Export a new instance of the `TestCase` class so that Supercharge knows how to handle it. Supercharge will pick up all test methods and pass them down to AVA for test runs. The following section shows you how to modify the handling of each test method using keywords.


### Lifecycle Hooks
Lifecycle hooks are a powerful way to run methods before or after test cases. The supported lifecycle methods in Supercharge are:

- `before`: runs before all tests in the suite
- `beforeEach`: runs before each test case
- `afterEach`: runs after each test case
- `after`: runs after all tests in the suite
- `alwaysAfter`: runs after all tests in the suite, even if a test fails

The order of the lifecycle hooks above matches the execution plan in AVA.

Implement lifecycle hoooks as methods in your test class:

```js
const BaseTest = require('@supercharge/framework/base-test')

class TestWithLifecycleHooks extends BaseTest {
  async before (t) {}
  async beforeEach (t) {}

  async afterEach (t) {}
  async after (t) {}

  async alwaysAfter (t) {}
}
```

The `alwaysAfter` hook will always run as soon as your tests complete. It will also run if your suite includes failing tests and makes it a good place for cleanup tasks.


### Test Context
Lifecycle hooks are a common place to create dummy data for your tests. For example, if you need a user instance in each test method, you can create a fake user before each test and save it in the test context. The test context is available at `t.context`.

```js
const BaseTest = require('@supercharge/framework/base-test')

class ContextData extends BaseTest {
  async beforeEach (t) {
    t.context.user = await this.fakeUser()
  }

  async testMethod (t) {
    const user = t.context.user
    t.pass()
  }

  async alwaysAfter (t) {
    await this.deleteUsers()
  }
}
```

Each test method has access to the test context via the `t` parameter. The context object is a convenient place if you need to pass data through the test suite.


### Assertions
Each test method receives the `t` parameter which makes [assertions](https://github.com/avajs/ava#assertions) directly available.

```js
const BaseTest = require('@supercharge/framework/base-test')

class Assertions extends BaseTest {
  async assertions (t) {
    t.is(1, 1)

    // many more assertions

    t.pass()
  }
}
```

Here’s a list of available assertions:

- `t.pass()`: passing assertion
- `t.fail()`: failing assertion
- `t.truthy(value)`: assert that `value` is truthy
- `t.falsy(value)`: assert that `value` is falsy
- `t.true(value)`: assert that `value` is `true`
- `t.false(value)`: assert that `value` is `false`
- `t.is(value, expected)`: assert that `value` is the same as `expected`
- `t.not(value, expected)`: assert that `value` is not the same as `expected`
- `t.deepEqual(value, expected)`: assert that `value` is deeply equal to `expected`
- `t.notDeepEqual(value, expected)`: assert that `value` is not deeply equal to `expected`
- `t.throws(fn, [expected])`: assert that an error is thrown
- `t.throwsAsync(thrower, [expected])`: assert that an error is thrown
- `t.notThrows(fn)`: assert that no error is thrown.
- `t.notThrowsAsync(nonThrower)`: assert that no error is thrown. Like the `.throwsAsync()` assertion, you must wait for the assertion to complete
- `t.regex(contents, regex)`: assert that `contents` matches `regex`
- `t.notRegex(contents, regex)`: assert that `contents` does not match `regex`
- `t.snapshot(expected)`: compare `expected` with a previously recorded snapshot
- `t.snapshot(expected, [options])`: snapshots are stored and assigned to a single test which requires unique test names. You can pass in a custom identifier via the `options` object: { id: 'my-snapshot-id' }. Find our more details on [snapshot testing in the AVA docs](https://github.com/avajs/ava#snapshot-testing)

Each assertion accepts a `message` as the last parameter. The message is a nice way if you need to print an additional message for an assertion.

Here are examples for custom messages: `t.pass('Successful assertion')` or `t.is(1, 1, 'Yep, 1 is 1')`.


### Run Specific Tests
In a large test class you might want to run only a subset of all test cases.

Prefix test methods with **`only`** to run only these methods.

```js
const BaseTest = require('@supercharge/framework/base-test')

class Only extends BaseTest {
  async onlyTest (t) {
    // this runs
  }

  async anotherTest (t) {
    // this won’t
  }
}
```

**Notice**: the “only” prefix is applicable in a single test file. Using `npm test` will run all your test files which might cause a lot more tests to run than you want. If you want to run a single test, you should use the NPM command [`npm run test-single <testCase>`](/docs/master/testing#run-a-single-test).


### Skip Tests
Skipping tests allows you you to simply ignore these (failing) test cases.

Prefix test methods with **`skip`** to skip these methods.

```js
const BaseTest = require('@supercharge/framework/base-test')

class Skip extends BaseTest {
  async skipTest (t) {
    // skipped
  }

async doNotSkip (t) {
    // not skipped
  }
}
```

Skipped tests are reported in the test result.

**Notice:** you cannot skip tests from inside the test method.


### Tests as “Todo”
Planning out test cases (without writing them yet) can be easily with the “todo” modifier.

Prefix test methods with **`todo`** to mark them as placeholders.

```js
const BaseTest = require('@supercharge/framework/base-test')

class Todo extends BaseTest {
  async todoTest (t) {
    // marked as TODO
  }
}
```

Test cases marked “todo” are reported in the test result.


### Run Tests Serially
Tests in Supercharge run concurrently by default. Test cases that can't run concurrently can be marked to run serially.

Prefix test methods with **`serial`** to run them sequentially before the concurrent test cases run.

```js
const BaseTest = require('@supercharge/framework/base-test')

class Serial extends BaseTest {
  async serialFirst (t) {
    // runs first
  }

  async serialSecond (t) {
    // runs second, after first finished
  }
}
```

Unfortunately, you can’t combine the `skip`, `todo` and `serial` modifier for a test. You can only use one of them. This method name is currently not available: `serialTodoOnlyTest`.


## Debug Tests
AVA runs tests in parallel using child processes. That’s why you need a workaround to attach a debugger to code in your tests.

Please follow the [advice on debugging with AVA](https://github.com/avajs/ava#debugging) given in the readme. You’ll find tips for populare IDEs and editors, like WebStorm and Visual Studio Code.
