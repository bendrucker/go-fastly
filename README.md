Go Fastly
=========
[![Build Status](http://img.shields.io/travis/fastly/go-fastly.svg?style=flat-square)][travis]
[![Go Documentation](http://img.shields.io/badge/go-documentation-blue.svg?style=flat-square)][godocs]

[travis]: http://travis-ci.org/fastly/go-fastly
[godocs]: http://godoc.org/github.com/fastly/go-fastly

Go Fastly is a Golang API client for interacting with most facets of the
[Fastly API](https://docs.fastly.com/api).

Installation
------------
This is a client library, so there is nothing to install. You must be running Go
1.8 or higher.

Usage
-----
Download the library into your `$GOPATH`:

    $ go get github.com/fastly/go-fastly/fastly

Import the library into your tool:

```go
import "github.com/fastly/go-fastly/fastly"
```

Examples
--------
Fastly's API is designed to work in the following manner:

1. Create (or clone) a new configuration version for the service
2. Make any changes to the version
3. Validate the version
4. Activate the version

This flow using the Golang client looks like this:

```go
// Create a client object. The client has no state, so it can be persisted
// and re-used. It is also safe to use concurrently due to its lack of state.
// There is also a DefaultClient() method that reads an environment variable.
// Please see the documentation for more information and details.
client, err := fastly.NewClient("YOUR_FASTLY_API_KEY")
if err != nil {
  log.Fatal(err)
}

// You can find the service ID in the Fastly web console.
var serviceID = "SERVICE_ID"

// Get the latest active version
latest, err := client.LatestVersion(&fastly.LatestVersionInput{
  Service: serviceID,
})
if err != nil {
  log.Fatal(err)
}

// Clone the latest version so we can make changes without affecting the
// active configuration.
version, err := client.CloneVersion(&fastly.CloneVersionInput{
  Service: serviceID,
  Version: latest.Number,
})
if err != nil {
  log.Fatal(err)
}

// Now you can make any changes to the new version. In this example, we will add
// a new domain.
domain, err := client.CreateDomain(&fastly.CreateDomainInput{
  Service: serviceID,
  Version: version.Number,
  Name: "example.com",
})
if err != nil {
  log.Fatal(err)
}

// Output: "example.com"
fmt.Println(domain.Name)

// And we will also add a new backend.
backend, err := client.CreateBackend(&fastly.CreateBackendInput{
  Service: serviceID,
  Version: version.Number,
  Name:    "example-backend",
  Address: "127.0.0.1",
  Port:    80,
})
if err != nil {
  log.Fatal(err)
}

// Output: "example-backend"
fmt.Println(backend.Name)

// Now we can validate that our version is valid.
valid, _, err := client.ValidateVersion(&fastly.ValidateVersionInput{
  Service: serviceID,
  Version: version.Number,
})
if err != nil {
  log.Fatal(err)
}
if !valid {
  log.Fatal("not valid version")
}

// Finally, activate this new version.
activeVersion, err := client.ActivateVersion(&fastly.ActivateVersionInput{
  Service: serviceID,
  Version: version.Number,
})
if err != nil {
  log.Fatal(err)
}

// Output: true
fmt.Printf("%t\n", activeVersion.Locked)
```

More information can be found in the
[Fastly Godoc](https://godoc.org/github.com/fastly/go-fastly).

Testing
-------
Go Fastly uses [go-vcr](https://github.com/dnaeon/go-vcr) to "record" and
"replay" API request fixtures to improve the speed and portability of
integration tests. The test suite uses a single test service ID for all test
fixtures.

Contributors _without_ access to the test service can disable go-vcr, set a
different test service ID, and use their own API credentials by setting the
following environment variables:
* `VCR_DISABLE`: disables go-vcr fixture recording and replay
* `FASTLY_TEST_SERVICE_ID`: set default service ID used by the test suite
* `FASTLY_API_KEY`: set a Fastly API key to be used by the test suite

Example (`go test`):
```sh
go test -v ./... -run=Logentries \
	VCR_DISABLE=1 \
	FASTLY_TEST_SERVICE_ID="SERVICEID" \
	FASTLY_API_KEY="TESTAPIKEY"
```

Example (`make test-full`):
```sh
make test-full FASTLY_TEST_SERVICE_ID="SERVICEID" \
	       FASTLY_API_KEY="TESTAPIKEY" \
	       TESTARGS="-run=Logentries"
```

When adding or updating client code and integration tests, contributors may
record a new set of fixtures by running the tests without `VCR_DISABLE`. Before
submitting a pull request with new or updated fixtures, we ask that contributors
update them to use the default service ID by running `make fix-fixtures` with
`FASTLY_TEST_SERVICE_ID` set to the same value used to run your tests.

Example (`make test`, `make fix-fixtures`):
```sh
export FASTLY_TEST_SERVICE_ID="SERVICEID"
export FASTLY_API_KEY="TESTAPIKEY"

make test TESTARGS="-run=NewClient"
make fix-fixtures

# Re-run test suite with newly recorded fixtures
unset FASTLY_TEST_SERVICE_ID FASTLY_API_KEY
make test
```

License
-------
```
Copyright 2015 Seth Vargo

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
