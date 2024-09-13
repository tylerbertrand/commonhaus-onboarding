# Develocity-configuration

Configuring your maven build is a relatively simple process. Instructions are available on the develocity site [here](https://docs.gradle.com/develocity/maven-extension/current/). For those instructions to work properly, you will need to be logged so be sure your account is set up and active already. Gradle users can find the relevant instructions [here](https://docs.gradle.com/develocity/get-started/) but the rest of this document assumes a maven project. For projects that use neither, you'll need to consult the develocity documentation but the rest of this should hopefully apply as well.

Setting up GitHub actions is relatively straightforward once you've set up your local workspace. Log in to develocity as your CI/github user and generate an access token for that account. Once you have your access token, you'll want to create a secret for your repository or organization as the case may be. In example below, I have created a secret called `DEVELOCITY_ACCESS_KEY` and for simplicity's sake, that secret is in the form of `develocity.commonhaus.dev=*************`. This is the format of the [environment variable](https://docs.gradle.com/develocity/maven-extension/current/#via_environment_variable) you will need to set in your workflows in order to publish build scans to develocity.

As you can see below, I have defined an env var named `DEVELOCITY_ACCESS_KEY` which simply uses the value of the secret directly. If you choose to have the secret value be solely the access token and define the env var here with the `<host>=<token>` format, that will also work. Either way, you should now be able to push build scans to your develocity account provided you've already configured your maven build appropriately.  

```yaml
name: Develocity job

on:
  push:
  workflow_dispatch:

env:
  DEVELOCITY_ACCESS_KEY: ${{ secrets.DEVELOCITY_ACCESS_KEY }}

jobs:
  develocity:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: 17
          distribution: 'zulu'
          server-id: central

      - name: Quick One
        run: ./mvnw -N install
```

Reusable builds
===

If your workflows use reusable build files, you'll need to do a little extra work. In the reusable workflow file, you'll need to declare that it accepts secrets from the calling workflow as shown here:

```yaml
on:
  workflow_call:
    inputs:
      <normal input parameters>
    secrets:
      DEVELOCITY_ACCESS_KEY:
        required: false
```

With this declaration done, you can then declare an env var just as shown above. In order to call this reusable workflow, then, you'll need to tell actions to pass those values along. For this, you'll need to add a new section to your call site like this:

```yaml
  Build:
    uses: <github account>/<repository>/.github/workflows/build.yml@master
    secrets:
      DEVELOCITY_ACCESS_KEY: ${{ secrets.DEVELOCITY_ACCESS_KEY }}
```

With that done, you should be all set up to begin pushing to develocity from GitHub Actions.
