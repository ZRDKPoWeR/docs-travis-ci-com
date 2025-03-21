---
title: Build an Android Project
layout: en

---


This guide covers build environment and configuration topics specific to Android projects. Please make sure to read our [Onboarding](/user/onboarding/) and [General Build configuration](/user/customizing-the-build/) guides first.

Android builds are not available on the macOS environment.


## CI Environment for Android Projects

### Overview

> Android builds are officially supported on our Bionic, Focal, and Jammy build environments; hence, you'll need to explicitly specify `dist: bionic`, `dist: focal`, or `dist: jammy` in your .travis.yml file.

Travis CI environment provides a large set of build tools for JVM languages with [multiple JDKs, Ant, Gradle, Maven](/user/languages/java/#overview), [sbt](/user/languages/scala/#projects-using-sbt) and [Leiningen](/user/languages/clojure/).

By setting

```yaml
language: android
dist: focal  # or dist: jammy, dist: bionic
```
{: data-file=".travis.yml"}

in your `.travis.yml` file, your project will be built in the Android environment which provides [Android SDK Tools](http://developer.android.com/tools/sdk/tools-notes.html).

Here is an example `.travis.yml` for an Android project:

```yaml
language: android
dist: focal
android:
  components:
    # Uncomment the lines below if you want to
    # use the latest revision of Android SDK Tools
    # - tools
    # - platform-tools

    # The BuildTools version used by your project
    - build-tools-30.0.0

    # The SDK version used to compile your project
    - android-30

    # Additional components
    - extra-google-google_play_services
    - extra-google-m2repository
    - extra-android-m2repository

    # Specify at least one system image,
    # if you need to run emulator(s) during your tests
    - sys-img-x86-android-30
    - sys-img-armeabi-v7a-android-17
```
{: data-file=".travis.yml"}

### Install Android SDK Components

In your `.travis.yml`, you can define the list of SDK components to be installed, as illustrated in the following example:

```yaml
language: android
dist: focal
android:
  components:
    - build-tools-30.0.0
    - android-30
    - extras;google;google_play_services
    - extras;google;m2repository
    - extras;android;m2repository
```
{: data-file=".travis.yml"}

The exact component names must be specified (filter aliases like `add-on` or `extra` are also accepted). To get a list of available exact component names and descriptions run the command `sdkmanager --list` (preferably in your local development machine).

Here are specific extra components you can install:

```yaml
android:
  components:
    # Google Play services
    - extras;google;google_play_services
    
    # Google Repository
    - extras;google;m2repository
    
    # Android Support Repository
    - extras;android;m2repository
    
    # Android Support Library
    - extras;android;support
    
    # Google Analytics libraries
    - extras;google;analytics_sdk_v2
    
    # Google Cloud Messaging libraries
    - extras;google;gcm
    
    # Google USB Driver (Windows only)
    - extras;google;usb_driver
    
    # Intel x86 Emulator Accelerator (HAXM installer)
    - extras;intel;Hardware_Accelerated_Execution_Manager
```

#### Deal with Licenses

By default, Travis CI will accept all the requested licenses, but it is also possible to define a white list of licenses to be accepted, as shown in the following example:

```yaml
language: android
dist: focal
android:
  components:
    - build-tools-30.0.0
    - android-30
    - add-on
    - extra
  licenses:
    - 'android-sdk-preview-license-52d11cd2'
    - 'android-sdk-license-.+'
    - 'google-gdk-license-.+'
```
{: data-file=".travis.yml"}

For more flexibility, the licenses can also be referenced with regular expressions (using Tcl syntax as `expect` command is used to automatically respond to the interactive prompts).

### Pre-installed Components

While the following components are preinstalled, the exact list may change without prior notice. To ensure the stability of your build environment, we recommend that you explicitly specify the required components for your project.

- tools
- platform-tools
- build-tools;30.0.0
- platforms;android-30
- extras;google;google_play_services
- extras;google;m2repository
- extras;android;m2repository

### Create and Start an Emulator

If you need to use an emulator for your tests, you can use the script [`/usr/local/bin/android-wait-for-emulator`](https://github.com/travis-ci/travis-cookbooks/blob/precise-stable/ci_environment/android-sdk/files/default/android-wait-for-emulator) and adapt your `.travis.yml` to make this emulator available for your tests. For example:

```yaml
# Emulator Management: Create, Start and Wait
before_script:
  - echo no | android create avd --force -n test -t android-30 --abi armeabi-v7a -c 100M
  - emulator -avd test -no-audio -no-window &
  - android-wait-for-emulator
  - adb shell input keyevent 82 &
```
{: data-file=".travis.yml"}

## Dependency Management

Travis CI Android builder assumes that your project is built with a JVM build tool like Maven or Gradle that will automatically pull down project dependencies before running tests without any effort on your side.

If your project is built with Ant or any other build tool that does not automatically handle dependencies, you need to specify the exact command to run using `install:` key in your `.travis.yml`, for example:

```yaml
language: android
dist: focal
install: ant deps
```
{: data-file=".travis.yml"}

## Default Test Command for Maven

If your project has `pom.xml` file in the repository root but no `build.gradle`, Maven 3 will be used to build it. By default it will use

```bash
mvn install -B
```

to run your test suite. This can be overridden as described in the [general build configuration](/user/customizing-the-build/) guide.

## Default Test Command for Gradle

If your project has `build.gradle` file in the repository root, Gradle will be used to build it. By default it will use

```bash
gradle build connectedCheck
```

to run your test suite. If your project also includes the `gradlew` wrapper script in the repository root, Travis Android builder will try to use it instead. The default command will become:

```bash
./gradlew build connectedCheck
```

This can be overridden as described in the [general build configuration](/user/customizing-the-build/) guide.

### Caching

A peculiarity of dependency caching in Gradle means that to avoid uploading the cache after every build you need to add the following lines to your `.travis.yml`:

```yaml
before_cache:
  - rm -f  $HOME/.gradle/caches/modules-2/modules-2.lock
  - rm -fr $HOME/.gradle/caches/*/plugin-resolution/
cache:
  directories:
    - $HOME/.gradle/caches/
    - $HOME/.gradle/wrapper/
    - $HOME/.android/build-cache
```
{: data-file=".travis.yml"}

## Default Test Command

If Travis CI cannot detect Maven or Gradle files, Travis CI Android builder will try to use Ant to build your project. By default, it will use

```bash
ant debug install test
```

to run your test suite. This can be overridden as described in the [general build configuration](/user/customizing-the-build/) guide.

## Test against Multiple JDKs

As for any JVM language, it is also possible to [test against multiple JDKs](/user/languages/java/#testing-against-multiple-jdks).

## Build Matrix

For Android projects, `env` and `jdk` can be given as arrays to construct a build matrix.

## Build Android projects on different build environments

Android projects are supported on `dist: bionic`, `dist: focal`, and `dist: jammy` build environments. If you have specific requirements for other build environments, you can install required packages and tools as shown in this example:

```yaml
os: linux
language: java
jdk: openjdk17

env:
 global:
  - ANDROID_HOME=$HOME/travis-tools/android
  - ANDROID_SDK_ROOT=$HOME/travis-tools/android

before_install:
 # PREPARE FOR ANDROID SDK SETUP
 - mkdir -p $HOME/travis-tools/android && mkdir $HOME/.android && touch $HOME/.android/repositories.cfg
 - cd $ANDROID_HOME && wget -q "https://dl.google.com/android/repository/commandlinetools-linux-10406996_latest.zip" -O commandlinetools.zip
 - unzip -q commandlinetools.zip && cd cmdline-tools
 - mv * tools | mkdir tools && cd $TRAVIS_BUILD_DIR

 # SETUP PATH(s)
 - export PATH=$ANDROID_HOME/cmdline-tools/tools/bin/:$PATH
 - export PATH=$ANDROID_HOME/emulator/:$PATH
 - export PATH=$ANDROID_HOME/platform-tools/:$PATH

install:
 # INSTALL REQUIRED ANDROID SDK TOOLS
 - sdkmanager --sdk_root=$ANDROID_HOME --list | awk '/Installed/{flag=1; next} /Available/{flag=0} flag'
 - yes | sdkmanager --sdk_root=$ANDROID_HOME --install "platform-tools" "platforms;android-30" "build-tools;30.0.0" "emulator" "system-images;android-30;google_apis;x86_64"
 - sdkmanager --list --sdk_root=$ANDROID_HOME | awk '/Installed/{flag=1; next} /Available/{flag=0} flag'
 # CREATE AVD
 - echo "no" | avdmanager --verbose create avd --force --name "my_android_30" --package "system-images;android-30;google_apis;x86_64" --tag "google_apis" --abi "x86_64"
 - sudo chmod -R 777 /dev/kvm
 # Start emulator in background and wait for it to start
 - adb kill-server && adb start-server &
 - sleep 15 # sleep values may require adjusting depending on the specific build environment
 - emulator @my_android_30 -no-audio -no-window &
 - sleep 60

script:
 - sdkmanager --list --sdk_root=$ANDROID_HOME | awk '/Installed/{flag=1; next} /Available/{flag=0} flag'
 - adb devices
 - .....
```
{: data-file=".travis.yml"}

## Examples

- [roboguice/roboguice](https://github.com/roboguice/roboguice/blob/master/.travis.yml) (Google Guide on Android)
- [ruboto/ruboto](https://github.com/ruboto/ruboto/blob/master/.travis.yml) (A platform for developing apps using JRuby on Android)
- [RxJava in Android Example Project](https://github.com/andrewhr/rxjava-android-example/blob/master/.travis.yml)
- [Gradle Example Project](https://github.com/pestrada/android-tdd-playground/blob/master/.travis.yml)
- [Maven Example Project](https://github.com/embarkmobile/android-maven-example/blob/master/.travis.yml)
- [Ionic Cordova Example Project](https://github.com/samlsso/Calc/blob/master/.travis.yml)

## Build Config Reference

You can find more information on the build config format for [Android](https://config.travis-ci.com/ref/language/android) in our [Travis CI Build Config Reference](https://config.travis-ci.com/).
