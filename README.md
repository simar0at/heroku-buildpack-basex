# Heroku Buildpack for BaseX

This is a 3rd party Heroku buildpack for BaseX based apps. It is based on the node.js buildpack and the common jvm parts used in Java buildpacks.
## Documentation

For more information about using this Node.js buildpack on Heroku, see these Dev Center articles:

- [Basex](https://www.basex.org)
- [Heroku Buildpack for the JDK](https://elements.heroku.com/buildpacks/heroku/heroku-buildpack-jvm-common)
- [Heroku Node.js Support](https://devcenter.heroku.com/articles/nodejs-support)
- [Getting Started with Node.js on Heroku](https://devcenter.heroku.com/articles/nodejs)
- [Troubleshooting Node.js Deploys](https://devcenter.heroku.com/articles/troubleshooting-node-deploys)

For more general information about buildpacks on Heroku:

- [Buildpacks](https://devcenter.heroku.com/articles/buildpacks)
- [Buildpack API](https://devcenter.heroku.com/articles/buildpack-api)

## Using the Heroku BaseX buildpack

It's suggested that you use the latest version of the release buildpack. You can set it using the `heroku-cli`.

You need to use the git url. You can use the `latest` tag to make sure you always have the latest release. **The `main` branch will always have the latest buildpack updates, but it does not correspond with a numbered release.**

```sh
heroku buildpacks:set https://github.com/heroku/heroku-buildpack-nodejs#latest -a my-app
```

## The admin password for BaseX

This buildpack does not use the standard `admin` password. Instead a randomly generated password is set.

- If you have a persistent data directory the admin password is stored in this directory. So it does not change.
- If you don't interact with the BaseX database in the container using `basexclient` or the dba BaseX app
  this does not matter to you. RestXQ apps for example run as admin user without needing to know the password.
- If you need the password it is stored in a text file within the container: `/app/.heroku/basex/credentials`
  That means: Access to the container means access to the DB so protect that.
- If you want to set a particular password pass the `BASEX_admin_pw` environment variable to the container and
  the admin password will be set accordingly.

If you don't set the password using the environment variable a new random password is set every time the container starts.
## Locking to a buildpack version

Even though it's suggested to use the latest release, you may want to lock dependencies - including buildpacks - to a specific version.

First, find the version you want from
[the list of buildpack versions](https://github.com/heroku/heroku-buildpack-nodejs/releases).
Then, specify that version with `buildpacks:set`:

```
heroku buildpacks:set https://github.com/heroku/heroku-buildpack-nodejs#v176 -a my-app
```

### Chain Node with multiple buildpacks

This buildpack automatically exports node, npm, and any node_modules binaries
into the `$PATH` for easy use in subsequent buildpacks.

## Feedback

Having trouble? Dig it? Feature request?

- [help.heroku.com](https://help.heroku.com/)
- [@adamzdanielle](http://twitter.com/adamzdanielle)
- [GitHub issues](https://github.com/heroku/heroku-buildpack-nodejs/issues)

## Development

### Prerequisites

For local development, you may need the following tools:

- [Docker](https://hub.docker.com/search?type=edition&offering=community)
- [Go 1.14](https://golang.org/doc/install#install)
- [upx](https://upx.github.io/)

### Deploying an app with a fork or branch

To make changes to this buildpack, fork it on GitHub.
Push up changes to your fork, then create a new Heroku app to test it,
or configure an existing app to use your buildpack:

```
# Create a new Heroku app that uses your buildpack
heroku create --buildpack <your-github-url>

# Configure an existing Heroku app to use your buildpack
heroku buildpacks:set <your-github-url>

# You can also use a git branch!
heroku buildpacks:set <your-github-url>#your-branch
```

### Downloading Plugins

In order to download the latest plugins that have been released, run the following:

```
plugin/download.sh v$VERSION
```

Make sure the version is in the format `v#`, ie. `v7`.

## Tests

The buildpack tests use [Docker](https://www.docker.com/) to simulate
Heroku's stacks.

To run the test suite:

```
make test
```

Or to just test a specific stack:

```
make heroku-16
make heroku-18
make heroku-20
```

The tests are run via the vendored
[shunit2](https://github.com/kward/shunit2)
test framework.

### Debugging

To display the logged build outputs to assist with debugging, use the "echo" and "cat" commands. For example:

```sh
test() {
  local log_file var

  var="testtest"
  log_file=$(mktemp)
  echo "this is the log file" > "$log_file"
  echo "test log file" >> "$log_file"

  # use `echo` and `cat` for printing variables and reading files respectively
  echo $var
  cat $log_file

  # some cases when debugging is necessary
  assertEquals "$var" "testtest"
  assertFileContains "test log file" "$log_file"
}
```

Running the test above would produce:

```log
testtest
this is the log file
test log file
```

The test output writes to `$STD_OUT`, so you can use `cat $STD_OUT` to read output.