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

This buildpack is meant to be used with the [gliderlabs/herokuish](https://github.com/gliderlabs/herokuish) [image](https://hub.docker.com/r/gliderlabs/herokuish/dockerfile) primarily in the [gitlab AutoDevOps Workflow](https://docs.gitlab.com/ee/topics/autodevops/customize.html#custom-buildpacks).
Obviously this is a buildpack for the herokuish workflow. You need to set `AUTO_DEVOPS_BUILD_IMAGE_CNB_ENABLED` to `false`.

You can emulate that workflow using just a local docker setup:

```sh
docker build --build-arg BUILDPACK_URL=https://github.com/simar0at/heroku-buildpack-basex -f ./test/gliderlabs/Dockerfile <your BaseX+nodejs based app>
``` 

It may also work with the original heroku service

```sh
heroku buildpacks:set https://github.com/simar0at/heroku-buildpack-basex#latest -a my-app
```

## BaseX settings

BaseX uses settings persisted in a `.basex` file. The following settings can be specified in `package.json` so they will be set accordingly during build:

```json
[...]
  "basex": {
    "parallel": "8",
    "parserestxq": "3",
    "cachetimeout": "3600",
    "timeout": "30",
    "gzip": "false",
    "fairlock": "false",
  }
[...]
```

For further documentation see ["Options" in the BaseX documentation](https://docs.basex.org/wiki/Options#Global_Options).

## Populating the database during build

If you have a file `deployment/initial.sh` in your sources this will be executed during build.

- You can use `basexclient` and the BaseX commands in script form or as bxs XML files to import data.
- You can use for example git or curl to fetch the data from some repository or URL

`deployment/initial.sh` is called with tha BaseX distribution base directory as parameter and the
environment variable `BUILD_DIR` set to the directory that contains the sources during the build process.
(Note that this is not the final location of your app so if you set up anything prefer relative paths.)
In this phase of the build process the password for the `admin` user is still `admin`.

## Testing

This buildpack is meant to run tests using nodejs. This should make it especially easy to create API tests or
end to end tests. If you want to run BaseX unit tests you can call `basexclient` and use the [TEST command](https://docs.basex.org/wiki/Commands#TEST).

```sh
source .heroku/basex/data/credentials
basexclient -u admin -p$BASEX_admin_pw -c 'TEST /app'
```

The password of the `admin` user has been randomized. You can get it from the `.heroku/basex/data/credentials` file.

## Persistent BaseX data base

If you want a persistent BaseX database across reboots of the container you have to attach a volume to
`/app/.heroku/basex/data`
If the attached volume is empty the initial state of BaseX from the build phase is copied to the mounted volume.

## The admin password for BaseX

This buildpack does not use the standard `admin` password. Instead a randomly generated password is set.

- If you have a persistent data directory the admin password is stored in this directory. So it does not change.
- If you don't interact with the BaseX database in the container using `basexclient` or the dba BaseX app
  this does not matter to you. RestXQ apps for example run as admin user without needing to know the password.
- If you need the password it is stored in a text file within the container: `/app/.heroku/basex/data/credentials`
  That means: Access to the container means access to the DB so protect that.
- If you want to set a particular password pass the `BASEX_admin_pw` environment variable to the container and
  the admin password will be set accordingly.

If you don't set the password using the environment variable a new random password is set every time the container starts.

## Using root ('') in your app

There is no way to have two RestXQ functions serving `''`. So unfortunately if your app source contains code for `''` it will not
run in when you just check out the code into a `webapp` directory of an unzipped ZIP distribution of BaseX.
Best practice is that your code usually is located in some "sub directory" `/yourapi`.
In Kubernetes for example you can tell the ingress to automatically forward to `/yourapi`.

This buildpack provides a mechanism to put a _disabled_ RestXQ function serving `''` into your repository that then for example forwards to `/yourapi`.
The idea is to write a usually very small module that as checked into the Git repository disables the %rest:* annotations by specifying
a custom URI for the rest prefix.

```xquery
xquery version "3.1";
module namespace _ = "uri:_";
declare namespace rest = "uri:rest";

(:~
 : Redirects to API path.
 : @return rest response
 :)
declare
  %rest:path('')
function _:index-file() as item()+ {
  <rest:response>
    <http:response status="302">
      <http:header name="Location" value="yourapi/"/>
    </http:response>
  </rest:response>
};
```

## Admin interface and examples

In the ZIP distribution of BaseX there is an admin web UI and a few examples. _These are deleted during build._
So if you don't supply some sort of forwarding mechanism or your RestXQ code runs in `''` a dysfunctional version of the web page you get in
the ZIP distribution of BaseX is returned in when accessing `''`.

## Locking to a buildpack version

Even though it's suggested to use the latest release, you may want to lock dependencies - including buildpacks - to a specific version.

First, find the version you want from
[the list of buildpack versions](https://github.com/heroku/heroku-buildpack-nodejs/tags).
Then, specify that version with `buildpacks:set`:

```
heroku buildpacks:set https://github.com/heroku/heroku-buildpack-nodejs#v176 -a my-app
```

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
make heroku-18-build
make heroku-20-build
make heroku-22-build
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
