
# shade-test

### Goal

1. Build nice, small cross-platform JavaFX-based Desktop apps with native installers
2. Continue to use the standard Maven dependency system.

### Problems

The jlink/jpackage tools which provide nice, small installers rely on the new Java
module system. This system is a bit difficult to use, as it expects all of the various
Java libraries used by an application to add additional information (typically either via
extra manifest entries or a new, compiled module-info.java/.class).

In the wild, most often these are dealt with by building project specific batch and shell
scripts. These are a pain to maintain, and are also rather brittle.

### Solution

In this sample project, these problems are manged through the use of a single
application shaded jar. All of the project's Maven dependencies are merged into a single JAR, and 
then jdeps automatically generates the module-info.java.

While this is a *terrible* strategy for libraries, it works just fine for end-user
desktop applications.

So, what we have here is a pretty small, simple Maven project template that can be used to 
build nice, small native installers for a JavaFX application using only Maven, Java 15, and 
the jpackage required macOS XCode or Windows WiX.

# Installation

1. Install [Java 15](https://adoptopenjdk.net/). Verify this by opening a fresh Terminal or
Command Prompt and typing `java --version`.
2. Install [Apache Maven](http://maven.apache.org/install.html) and make sure it's on your path.
Verify this by opening a fresh Terminal or Command Prompt and typing `mvn --version`.
3. Clone/download this project.
4. Download and the [platform-specific JavaFX](https://gluonhq.com/products/javafx/) files.
5. Place the contents of the JavaFX jmods and JavaFX SDK in the proper locations in your project. 
Here's a sample showing what that should look like:
![Install Sample](docs/file-layout.png)
6. Add the jpackage configuration to your MAVEN_OPTS for your shell environment (described in 
more detail below).
As of Java 15, you can verify this is working by observing the warning about using an incubator
project. Here's what the output looks like on Windows - notice the first line WARNING.
``` 
C:\Users\wiver\src\shade-test>mvn --version
WARNING: Using incubator modules: jdk.incubator.jpackage
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: C:\Users\wiver\devenv\apache-maven-3.6.3\apache-maven-3.6.3\bin\..
Java version: 15.0.1, vendor: AdoptOpenJDK, runtime: C:\Program Files\AdoptOpenJDK\jdk-15.0.1.9-hotspot
Default locale: en_US, platform encoding: Cp1252
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```

7. If you are running on macOS, make sure you have XCode installed and accepted any needed
agreements. Try opening Terminal and running the command `hdiutil` and make sure it's on the path.
8. If you are running on Windows, install the [Wix 3 binaries](https://github.com/wixtoolset/wix3/releases/).
As of this writing, merely installing Wix via the installer was sufficient for jpackage to find it.
9. Once all that is working, you should be able to just run `mvn clean install` from the root of the project
and you'll have either a TestApp.dmg or a TestApp.exe (installer).
For reference, here is a complete run log for [a successful run on Windows](docs/sample-windows-run.md).

Because these builds use stripped down JVM images, the final installers on both macOS and Windows are in
the 30-40MB range.

## jpackage Configuration

This project relies on [jtoolprovider-plugin](https://github.com/wiverson/jtoolprovider-plugin) 
to perform key build steps. To generate the actual installers, the 
jpackage tool must be available to the ToolProvider API.  
For most people, adding a `MAVEN_OPTS` environment variable 
will work nicely.

For example, on macOS, this can be done by adding to
the following line to the `~/.zshrc` file.

`export MAVEN_OPTS="--add-modules jdk.incubator.jpackage"`

Current versions of Windows 10 have a nice UI for adding an environment
variable. You can find it in the modern control panel via search -
just start a search for "env" and that should bring it up.

Unfortunately, as of this writing adding this entry to the IntelliJ
options for Maven (either in the IntelliJ Maven JVM importer UI or 
via `project-directory/.mvn/jvm.config`) will break the Maven sync. 
This bug is tracked by JetBrains as 
[IDEA-246963](https://youtrack.jetbrains.com/issue/IDEA-246963).
There is a `.mvn/Xjvm.config file` in this project - once the bug is fixed,
 or if you use a different editor, 
just try renaming that file to `jvm.config`. Or, presumably, when Java 16 
ships and jpackage is no longer in incubation.

## Usage

To generate an installer, just run...

`mvn clean install`

To do everything up until the actual installer generation...

`mvn clean package`