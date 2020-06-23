# Rest API YAML Test Runner

The specs in `elasticsearch-api` and `elasticsearch-xpack` automatically run the tests from [Elasticsearch's REST API Spec tests](https://github.com/elastic/elasticsearch/tree/master/rest-api-spec/src/main/resources/rest-api-spec/test#test-suite). The test runner is defined in each of these project's `spec` folder, starting with the `rest_api_yaml_spec.rb` file.

## REST API YAML Spec

The file that traverses the yaml files and loads a **TestFile** object per each of them:
`elasticsearch-(api|xpack)/spec/elasticsearch/api/rest_api_yaml_spec.rb`

You can use the SINGLE_TEST env variable to run just one test, or add code like this on the first line of the tests.each block:  
```ruby
next unless file =~ /indices.put_mapping\/all_path_options_with_types.yml/
```

## TestFile
Class representing a single test file. Contains setup, teardown and tests.   
`../api-spec-testing/test_file.rb`

## Test
Every single test in the test file is represented in the Test object.   
`../api-spec-testing/test_file/test.rb`

## TaskGroup

Tests are ordered in task groups, an array of TaskGroup objects.  
`../api-spec-testing/test_file/task_group.rb`

Task Groups are a representation of a block of actions consisting of 'do' actions and their verifications. e.g.: 
```yaml
 - do:
      index:
          index:  test-index
          id:     1
          body:   { foo: bar }

 - match:   { _index:   test-index }
 - match:   { _id:      "1"}
 - match:   { _version: 1}
```

**Before** each test, the spec runner runs `clear_data` on the test_file. This clears indices, index templates, snapshots and repositories. For xpack it also clears roles, users, privileges, datafeeds, ml_jobs and more.

**After** each test, it runs the test file teardown and `clear_data` again.

For each TaskGroup, it sees what's in the task group definition and runs an expectation test.

## Rest YAML tests Helper

`elasticsearch-(api|xpack)/spec/rest_yaml_tests_helper.rb`

- `ADMIN_CLIENT` is defined here.
- `SINGLE_TEST` is defined here.
- Skipped tests are listed here

## Spec Helper

- `DEFAULT_CLIENT` is defined here

## Enable Logging

To enable logging, set the environment `QUIET` to false before running the tests. In CI, this is located in the [Dockerfile](https://github.com/elastic/elasticsearch-ruby/blob/master/.ci/Dockerfile). The environment variable is evaluated in the Rest YAML tests Helper file.

# Features

## RSpec Matchers

The tests use custom [RSpec Matchers](https://www.rubydoc.info/gems/rspec-expectations/RSpec/Matchers) defined in `api-spec-testing/rspec_matchers.rb`.

From the [Rest API test docs](https://github.com/elastic/elasticsearch/tree/master/rest-api-spec/src/main/resources/rest-api-spec/test#do):

## `catch`

> If the arguments to `do` include `catch`, then we are expecting an error, which should be caught and tested.

In `rest_api_yaml_spec`, there's a check for `catch_exception?` per task_group. This checks if the `do` definitions have any `catch` definitions. If there is a `catch`, it'll send the request and use the `match_error` RSpec custom matcher to validate the expected error.

## `warnings`

>If the arguments to `do` include `warnings` then we are expecting a Warning header to come back from the request. If the arguments don’t include a warnings argument then we don’t expect the response to include a Warning header. The warnings must match exactly. Using it looks like this:

```
- do:
    warnings:
        - '[index] is deprecated'
        - quotes are not required because yaml
        - but this argument is always a list, never a single string
        - no matter how many warnings you expect
    get:
        index:    test
        type:    test
        id:        1
```
