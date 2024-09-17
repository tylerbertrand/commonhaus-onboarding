# Connecting a project to Develocity

Develocity provides [Build Scans](https://gradle.com/gradle-enterprise-solutions/build-scan-root-cause-analysis-data/) and a remote [build cache](https://gradle.com/gradle-enterprise-solutions/build-cache/) for both local and CI builds. For Maven users, it additionally provides local build caching (Gradle provides this natively). This guide walks through the necessary steps to connect your project to Develocity to begin publishing build scans.

## Develocity user account creation

For developers to publish Build Scans and utilize the remote build cache, a Develocity user account with the `Developer` role is required. Similarly, a user account with the `CI Agent` role is required for CI builds. Contact `<contact>` to request the creation of these accounts.

## Configuring the Develocity Gradle Plugin / Develocity Maven Extension

Connecting to Develocity to publish Build Scans and utilize the remote build cache requires first applying and configuring either the Develocity Gradle Plugin or Develocity Maven Extension. The following sections describe the process for both [Gradle users](#configuring-the-develocity-gradle-plugin) and [Maven users](#configuring-the-develocity-maven-extension).

### Configuring the Develocity Gradle Plugin

For Gradle projects using Groovy DSL, apply the following to your `settings.gradle`:

```groovy
plugins {
    id 'com.gradle.develocity' version '<version>'
    id 'com.gradle.common-custom-user-data-gradle-plugin' version '<version>'
}

def isCI = System.getenv('CI') != null // adjust to your CI provider

develocity {
    server = 'https://develocity.commonhaus.dev'
    allowUntrustedServer = false

    buildScan {
        uploadInBackground = !isCI
        publishing.onlyIf { it.authenticated }
        obfuscation {
            ipAddresses { addresses -> addresses.collect { address -> "0.0.0.0" } }
        }
    }
}

buildCache {
    local {
        enabled = true
    }

    remote(develocity.buildCache) {
        enabled = false
        push = isCI
    }
}

rootProject.name = '<project-name>' // adjust to your project
```

For Gradle projects using Kotlin DSL, apply the following to your `settings.gradle.kts`:

```kotlin
plugins {
    id("com.gradle.develocity") version "<version>"
    id("com.gradle.common-custom-user-data-gradle-plugin") version "<version>"
}

val isCI = System.getenv("CI") != null // adjust to your CI provider

develocity {
    server = "https://develocity.commonhaus.dev"
    allowUntrustedServer = false

    buildScan {
        uploadInBackground = !isCI
        publishing.onlyIf { it.authenticated }
        obfuscation {
            ipAddresses { addresses -> addresses.map { _ -> "0.0.0.0" } }
        }
    }
}

buildCache {
    local {
        isEnabled = true
    }

    remote(develocity.buildCache) {
        isEnabled = false
        isPush = isCI
    }
}

rootProject.name = "<project-name>" // adjust to your project
```

This configuration applies the Develocity Gradle Plugin and the [Common Custom User Data Gradle Plugin](https://github.com/gradle/common-custom-user-data-gradle-plugin?tab=readme-ov-file#common-custom-user-data-gradle-plugin), which enhances published Build Scans by adding a set of tags, links and custom values that have proven to be useful for many projects building with Develocity. It configures the Develocity Gradle Plugin to connect to the Commonhaus Develocity instance and specifies that Build Scans should only be published if authenticated with Develocity. Note that it also disables usage of the remote build cache for the moment.

#### Authenticating with Develocity

After the necessary Develocity user accounts have been created and the Develocity Gradle Plugin and configuration have been applied to your build, in order to publish Build Scans, the next step is to authenticate your build with Develocity.

##### Authenticating with Local Builds

For local builds, you can automatically provision an access key by executing either the `provisionDevelocityAccessKey` Gradle task, which is documented [here](https://docs.gradle.com/develocity/gradle-plugin/current/#automated_access_key_provisioning).

##### Authenticating with CI Builds

For CI builds, you can first manually generate an access token for the Develocity CI user created for your project. How to do this is described [here](https://docs.gradle.com/develocity/gradle-plugin/current/#manual_access_key_configuration). Next, that token can be to authenticate with Develocity via the `DEVELOCITY_ACCESS_KEY` environment variable. This is described in more detail [here](https://docs.gradle.com/develocity/maven-extension/current/#via_environment_variable). Note that the environment variable format is `DEVELOCITY_ACCESS_KEY=«server host name»=«access key»`.

You can find more details about how to set this up with specific CI providers [here](#ci-specific-setup).

### Configuring the Develocity Maven Extension

For Maven projects, apply the following to your `.mvn/extensions.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<extensions>
    <extension>
        <groupId>com.gradle</groupId>
        <artifactId>develocity-maven-extension</artifactId>
        <version><!--version--></version>
    </extension>
    <extension>
        <groupId>com.gradle</groupId>
        <artifactId>common-custom-user-data-maven-extension</artifactId>
        <version><!--version--></version>
    </extension>
</extensions>
```

Create the file `.mvn/develocity.xml` and apply the following:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes" ?>
<develocity
    xmlns="https://www.gradle.com/develocity-maven" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="https://www.gradle.com/develocity-maven https://www.gradle.com/schema/develocity-maven.xsd">
  <server>
    <url>https://develocity.commonhaus.dev</url>
    <allowUntrusted>false</allowUntrusted>
  </server>
  <buildScan>
    <backgroundBuildScanUpload>#{isFalse(env['CI'])}</backgroundBuildScanUpload> <!-- adjust to your CI provider -->
      <obfuscation>
        <ipAddresses>#{{'0.0.0.0'}}</ipAddresses>
      </obfuscation>
  </buildScan>
  <buildCache>
    <local>
      <enabled>true</enabled>
    </local>
    <remote>
      <enabled>false</enabled>
      <storeEnabled>#{isTrue(env['CI'])}</storeEnabled> <!-- adjust to your CI provider -->
    </remote>
  </buildCache>
</develocity>
```

This configuration applies the Develocity Maven Extension and the [Common Custom User Data Maven Extension](https://github.com/gradle/common-custom-user-data-maven-extension?tab=readme-ov-file#common-custom-user-data-maven-extension), which enhances published Build Scans by adding a set of tags, links and custom values that have proven to be useful for many projects building with Develocity. It configures the Develocity Maven Extension to connect to the Commonhaus Develocity instance and specifies that Build Scans should only be published if authenticated with Develocity. Note that while this configuration enables the local build cache, it disables the usage of the remote build cache for the moment.

#### Authenticating with Develocity

After the necessary Develocity user accounts have been created and the Develocity Maven Extension and configuration have been applied to your build, in order to publish Build Scans, the next step is to authenticate your build with Develocity.

##### Authenticating with Local Builds

For local builds, you can automatically provision an access key by executing the `develocity:provision-access-key` Maven Goal, which is documented [here](https://docs.gradle.com/develocity/maven-extension/current/#automated_access_key_provisioning).

##### Authenticating with CI Builds

For CI builds, you can first manually generate an access token for the Develocity CI user created for your project. How to do this is described [here](https://docs.gradle.com/develocity/maven-extension/current/#creating_access_keys). Next, that token can be to authenticate with Develocity via the `DEVELOCITY_ACCESS_KEY` environment variable. This is described in more detail [here](https://docs.gradle.com/develocity/maven-extension/current/#via_environment_variable). Note that the environment variable format is `DEVELOCITY_ACCESS_KEY=«server host name»=«access key»`.

You can find more details about how to set this up with specific CI providers [here](#ci-specific-setup).

## CI Provider-Specific Setup

### GitHub Actions

Once you have an access token for your CI user (the process to generate this is described in the [Gradle setup guide](#authenticating-with-ci-builds) and the [Maven setup guide](#authenticating-with-ci-builds-1)), you'll want to create a secret for your repository or organization as the case may be. In example below, the secret is `DEVELOCITY_ACCESS_KEY`, and for simplicity's sake, that secret is in the form of `develocity.commonhaus.dev=*************`. This is the format of the [environment variable](https://docs.gradle.com/develocity/maven-extension/current/#via_environment_variable) you will need to set in your workflows in order to publish build scans to develocity.

As you can see in the below example executing a Maven build, the environment variable `DEVELOCITY_ACCESS_KEY` has been defined, which simply uses the value of the secret directly. If you choose to have the secret value be solely the access token and define the env var here with the `<host>=<token>` format, that will also work. Either way, you should now be able to push build scans to Develocity provided you've already configured your build appropriately.

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

#### Reusable Builds

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

With that done, you should be all set up to begin pushing Build Scans to Develocity from GitHub Actions.
