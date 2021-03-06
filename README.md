<!-- archived-provider -->
Please note: This Terraform provider is archived, per our [provider archiving process](https://terraform.io/docs/internals/archiving.html). What does this mean?
1. The code repository and all commit history will still be available.
1. Existing released binaries will remain available on the releases site.
1. Issues and pull requests are not being monitored.
1. New releases will not be published.

If anyone from the community or an interested third party is willing to maintain it, they can fork the repository and [publish it](https://www.terraform.io/docs/registry/providers/publishing.html) to the Terraform Registry. If you are interested in maintaining this provider, please reach out to the [Terraform Provider Development Program](https://www.terraform.io/guides/terraform-provider-development-program.html) at *terraform-provider-dev@hashicorp.com*.

---

<!-- /archived-provider -->

[![Build Status](https://travis-ci.org/ewilde/terraform-provider-runscope.svg?branch=master)](https://travis-ci.org/ewilde/terraform-provider-runscope)

# Terraform Runscope Provider

- Website: https://www.terraform.io
- [![Gitter chat](https://badges.gitter.im/hashicorp-terraform/Lobby.png)](https://gitter.im/hashicorp-terraform/Lobby)
- Mailing list: [Google Groups](http://groups.google.com/group/terraform-tool)

<img src="https://cdn.rawgit.com/hashicorp/terraform-website/master/content/source/assets/images/logo-hashicorp.svg" width="600px">

The Runscope provider is used to create and manage Runscope tests using
the official [Runscope API](https://www.runscope.com/docs/api)

## Requirements

-	[Terraform](https://www.terraform.io/downloads.html) 0.10.x
-	[Go](https://golang.org/doc/install) 1.11 (to build the provider plugin)


## Installing

See the [Plugin Basics][4] page of the Terraform docs to see how to plug this
into your config. Check the [releases page][5] of this repository to get releases for
Linux, OS X, and Windows.

## Usage

The following section details the use of the provider and its resources.

These docs are derived from the middleman templates that were created for the
PR itself, and can be found in their original form [here][5].

### Example Usage

The below example is an end-to-end demonstration of the setup of a basic
runscope test:


### Creating a test with a step
```hcl
resource "runscope_step" "main_page" {
  bucket_id      = "${runscope_bucket.bucket.id}"
  test_id        = "${runscope_test.test.id}"
  step_type      = "request"
  url            = "http://example.com"
  method         = "GET"
  variables      = [
  	{
  	   name     = "httpStatus"
  	   source   = "response_status"
  	},
  	{
  	   name     = "httpContentEncoding"
  	   source   = "response_header"
  	   property = "Content-Encoding"
  	},
  ]
  assertions     = [
  	{
  	   source     = "response_status"
       comparison = "equal_number"
       value      = "200"
  	},
  	{
  	   source     = "response_json"
       comparison = "equal"
       value      = "c5baeb4a-2379-478a-9cda-1b671de77cf9",
       property   = "data.id"
  	},
  ],
  headers        = [
  	{
  		header = "Accept-Encoding",
  		value  = "application/json"
  	},
  	{
  		header = "Accept-Encoding",
  		value  = "application/xml"
  	},
  	{
  		header = "Authorization",
  		value  = "Bearer bb74fe7b-b9f2-48bd-9445-bdc60e1edc6a",
	}
  ]
}

resource "runscope_test" "test" {
  bucket_id   = "${runscope_bucket.bucket.id}"
  name        = "runscope test"
  description = "This is a test test..."
}

resource "runscope_bucket" "bucket" {
  name      = "terraform-provider-test"
  team_uuid = "dfb75aac-eeb3-4451-8675-3a37ab421e4f"
}
```

## Argument Reference

The following arguments are supported:

* `bucket_id` - (Required) The id of the bucket to associate this step with.
* `test_id` - (Required) The id of the test to associate this step with.
* `step_type` - (Required) The type of step.
 * [request](#request-steps)
 * pause
 * condition
 * ghost
 * subtest

### Request steps
When creating a `request` type of step the additional arguments also apply:

* `method` - (Required) The HTTP method for this request step.
* `variables` - (Optional) A list of variables to extract out of the HTTP response from this request. Variables documented below.
* `assertions` - (Optional) A list of assertions to apply to the HTTP response from this request. Assertions documented below.
* `headers` - (Optional) A list of headers to apply to the request. Headers documented below.
* `body` - (Optional) A string to use as the body of the request.

Variables (`variables`) supports the following:

* `name` - (Required) Name of the variable to define.
* `property` - (Required) The name of the source property. i.e. header name or json path
* `source` - (Required) The variable source, for list of allowed values see: https://www.runscope.com/docs/api/steps#assertions

Assertions (`assertions`) supports the following:

* `source` - (Required) The assertion source, for list of allowed values see: https://www.runscope.com/docs/api/steps#assertions
* `property` - (Optional) The name of the source property. i.e. header name or json path
* `comparison` - (Required) The assertion comparison to make i.e. `equals`, for list of allowed values see: https://www.runscope.com/docs/api/steps#assertions
* `value` - (Optional) The value the `comparison` will use

**Example Assertions**

Status Code == 200

```json
"assertions": [
    {
        "source": "response_status",
        "comparison": "equal_number",
        "value": 200
    }
]
```

JSON element 'address' contains the text "avenue"


```json
"assertions": [
    {
        "source": "response_json",
        "property": "address",
        "comparison": "contains",
        "value": "avenue"
    }
]
```

Response Time is faster than 1 second.


```json
"assertions": [
    {
        "source": "response_time",
        "comparison": "is_less_than",
        "value": 1000
    }
]
```

The `headers` list supports the following:

* `header` - (Required) The name of the header
* `value` - (Required) The name header value

## Attributes Reference

The following attributes are exported:

* `id` - The ID of the step.


[1]: https://github.com/hashicorp/terraform/pull/14221
[2]: https://www.runscope.com/docs/api
[3]: https://www.terraform.io/docs/plugins/basics.html
[4]: https://github.com/ewilde/terraform-provider-runscope/releases
[5]: website/source/docs/providers/runscope

# Developing

## Building The Provider

Clone repository to: `$GOPATH/src/github.com/terraform-providers/terraform-provider-runscope`

```sh
$ mkdir -p $GOPATH/src/github.com/terraform-providers; cd $GOPATH/src/github.com/terraform-providers
$ git clone git@github.com:terraform-providers/terraform-provider-runscope
```

Enter the provider directory and build the provider

```sh
$ cd $GOPATH/src/github.com/terraform-providers/terraform-provider-runscope
$ make build
```

## Using the provider

See [examples](examples/)

See [runscope providers documentation](https://www.terraform.io/docs/providers/runscope/index.html)

## Developing the Provider

If you wish to work on the provider, you'll first need [Go](http://www.golang.org) installed on your machine (version 1.11+ is *required*). You'll also need to correctly setup a [GOPATH](http://golang.org/doc/code.html#GOPATH), as well as adding `$GOPATH/bin` to your `$PATH`.

To compile the provider, run `make build`. This will build the provider and put the provider binary in the `$GOPATH/bin` directory.

```sh
$ make build
...
$ $GOPATH/bin/terraform-provider-runscope
...
```

## Running the integration tests

`make TF_ACC=1 RUNSCOPE_TEAM_ID=xxx RUNSCOPE_ACCESS_TOKEN=xxx RUNSCOPE_INTEGRATION_DESC="Slack: #test1 channel, send message on all test runs"`


| Environment variables           | Description                             |
|:----------------|:----------------------------------------|
| TF_ACC| `1` runs acceptance tests, do not set to skip acceptance tests |
| RUNSCOPE_TEAM_ID | Runscope [team uuid](https://www.runscope.com/docs/api/teams)|
| RUNSCOPE_ACCESS_TOKEN | Runscope [access token](https://www.runscope.com/applications/create) |
| RUNSCOPE_INTEGRATION_DESC | Description that matches a pre-existing runscope integration associated with your account  |

## Vendoring / dependency management
Dependencies are managed using [Go Modules](https://github.com/golang/go/wiki/Modules)

### Add or update a dependency

```
$ GO111MODULE=on go get <module>@<version>
$ GO111MODULE=on go mod tidy
```

For the time being we will continue to vendor
```
$ GO111MODULE=on go mod vendor
```

## Releasing
Releases are automatically setup to go out from the master branch after a build is made on master with a tag.

To perform a release simply create a tag:
` git tag -a v0.0.2 -m "Release message"`

Then push your tag:
`git push origin v0.0.2`


That's it, the build will now run and create a new release on [github](https://github.com/form3tech/ewilde/terraform-provider-runscope) :
