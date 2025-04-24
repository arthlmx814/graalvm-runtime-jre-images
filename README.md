# GraalVM - Java Runtime Environment (JRE) Docker Images

[![GraalVM - Java Runtime Environment - Docker Hub Publish](https://github.com/arthlmx814/graalvm-runtime-jre-images/actions/workflows/docker-publish.yaml/badge.svg?branch=main)](https://github.com/arthlmx814/graalvm-runtime-jre-images/actions/workflows/docker-publish.yaml)

This repository contains the source for building GraalVM Java JRE Docker images as Oracle does not provide official JRE Docker images.

The images are using the official [GraalVM Oracle JDK](https://www.graalvm.org/downloads/) binaries to build the images with the jlink tool. (<https://www.youtube.com/watch?v=3UCBmdbeYm4>)

The images are based on the official [Ubuntu](https://hub.docker.com/_/ubuntu) images and contains the ca-certificates package to support SSL/TLS connections.

All the images have the java executable in the PATH and the JAVA_HOME environment variable set. The entrypoint is the java executable. You can use it with the [Google JIB](https://github.com/GoogleContainerTools/jib) tool on your maven or gradle project.

## Supported Tags

The images are available on [Docker Hub](https://hub.docker.com/r/arthlmx814/graalvm-jre) with the following tags:

| Tags | Java Version | Ubuntu Version | Architecture | LTS |
| ---- | ------------ | -------------- | ------------ | --- |
| `21`,`21.0.7`,`21-noble`,`21.0.7-noble`,`21-jammy`,`21.0.7-jammy` | 21.0.7+8.1 | 24.04 (Noble Numbat), 22.04 (Jammy Jellyfish) | x86_64, aarch64 | Yes |
| `24`,`24.0.2`,`24-noble`,`24.0.1-noble`,`24-jammy`,`24.0.1-jammy` | 24.0.1+9.1 | 24.04 (Noble Numbat), 22.04 (Jammy Jellyfish) | x86_64, aarch64 | No |

The tags without the suffix `-jammy` or `-noble` are based on the latest Ubuntu LTS version (Ubuntu 24.04) and the tags with the suffix `-jammy` or `-noble` are based on their respective Ubuntu version.

## Usage

You can use the GraalVM Java JRE Docker images like any other Docker image. For example, you can use the following command get the version of the java executable:

```shell
docker run --rm arthlmx814/graalvm-jre:24 -version
```

## FAQ

### What is GraalVM?

GraalVM is an universal virtual machine for running applications written in JavaScript, Python, Ruby, R, JVM-based languages like Java, Scala, Groovy, Kotlin, Clojure, and LLVM-based languages such as C and C++.

For Java developers, GraalVM provides a high-performance Java runtime with a number of additional features, such as the ability to compile Java code ahead-of-time into a native executable. (with the native-image tool)

The JIT compiler (JVMCI) in GraalVM is written in Java and can be used by any JVM-based language. (As a replacement for the C2 compiler in OpenJDK)

For more information, see the [GraalVM website](https://www.graalvm.org/) page.

### Why distribute GraalVM Java JRE Docker images?

#### 1. Because GraalVM provides a high-performance Java runtime

GraalVM provides the GraalVM JIT compiler (JVMCI) that provides very good performance for Java applications. (better than the C2 compiler in OpenJDK for my personal experience with the applications that I develop in my company or for my personal projects)

And I think that some people (or companies) might want to use GraalVM for that reason.

#### 2. Because not everyone can use jlink or jpackage to create a custom JRE

You might know that since Java 9, the JDK provides modularization, which means that you can create a custom JRE with the jlink tool based on the module-info.java file.
That is great and you can create a very small JRE with only the modules of the JDK that you need!

But not everyone can use jlink or jpackage to create a custom JRE. For example, if you are using a third-party library that is not modularized, you cannot create a custom JRE with jlink. (like flyway or rxjava). The jlink rule is that all the dependencies of the modules that you want to include in the custom JRE must be modularized. (and sub-modules, and sub-sub-modules, and so on)

You can still use the jmod tool to know which modules are required by the third-party library and generate a module-info.java file for the library, but that is not always easy to do and some libraries are not helpful for that. (the kubernetes java-client library as a great example, for my personal experience)

For that reason, some company might need to use the "legacy" JRE with all the modules of the JDK without the javac, jlink, or other tools that are not required to run the application during the production phase.

For example, the Eclipse Temurin project provides great JRE docker images that I use here to know which modules can be used for a production JRE. (<https://adoptium.net/temurin/>)

### Is the native-image tool included in the GraalVM Java JRE Docker images?

No, the native-image tool is not included in the images. The native-image tool is used to compile Java code ahead-of-time into a native executable. (A binary file that can be executed without the JVM)

Once compiled, you can deploy it on ubuntu-based, alpine, microsoft, or other distributions without the need to install the JVM. (depending on the target platform you choose). Or "scratch" if you use --static --libc=musl options with the native-image tool.

### What kind of modules are included in the GraalVM Java JRE Docker images?

Simple answer: The same modules that are included in the Eclipse Temurin JRE Docker images. (<https://hub.docker.com/_/eclipse-temurin>) based on the target java version of the JDK I use to build the images.

But, I only add the jdk.jcmd module to provide support for the jfr and jcmd tools. (for flight recorder, not included in the temurin images) (for more information: <https://www.baeldung.com/java-flight-recorder-monitoring>)

If you have any suggestions or improvements about the modules that should be included in the GraalVM Java JRE Docker images, feel free to open an issue.

### What architectures are supported by the GraalVM Java JRE Docker images?

As GraalVM provides binaries for the x86_64 architecture and aarch64 architecture, the GraalVM Java JRE Docker images are available both for the x86_64 and aarch64 architectures. (linux/amd64 and linux/arm64 respectively)

### Do you provide windows or macos images?

Not, I have no plan to provide windows or macos images. (I only provide linux images for the x86_64 and aarch64 architectures)

### Why using ubuntu as based image?

I use the ubuntu-based images because they are the most popular images and they are the smallest images that I can use for the glibc-based distributions. (like ubuntu, debian, centos, fedora, etc)

If you know a (secured and official) smaller image that I can use as a base image, please let me know. (debian is higher than ubuntu, alpine is smaller but it is not glibc-based, scratch is the smallest but the binaries require glibc and other dependencies)

### Why not using alpine as based image?

Like I said before, alpine is smaller than ubuntu but it is not glibc-based. (it uses musl as the standard C library) and, for now, the binaries do not provide support for musl-based distributions.

I could use the "gcompat" package to provide glibc compatibility for alpine-based images, but I cannot be 100% sure that it will work as expected.

### What frequency do you update the GraalVM Java JRE Docker images?

I use a GitHub action to build the GraalVM Java JRE Docker images every week on Sunday at 00:00 UTC. (if there is a new version of the GraalVM JDK available or if ubuntu updates the base image)

This is to avoid any security issues that might be present in the JDK or in the base image.

My GitHub action fetches automatically the latest version of the GraalVM Java JDK (21 and 24) and builds them.

### Do you provide a "latest" tag for the GraalVM Java JRE Docker images?

No, bacause there is no sense to use a "latest" tag with java versions.
Even if the JVM is backward compatible, some applications might not work as expected with a newer version of the JVM.
The "latest" is the major version of the JDK. (like 21 or 24 in the case of GraalVM Java)

### What versions of GraalVM Java do you provide?

I provide the latest and long-term support (LTS) versions of GraalVM Java.
Today, the (supported and free) LTS version is 21. The latest version is 24.

I do not provide the early access (EA) versions of GraalVM Java because they are not recommended for production use.

Be warned that once a version of GraalVM Java reaches the end of its (free) support, I will no longer provide the GraalVM Java JRE images for that version but it will still be available in the DockerHub registry (but will not receive any updates)

### How do you handle the security of the GraalVM Java JRE Docker images?

I use the official GraalVM Java binaries to build the GraalVM Java JRE Docker images and I use the official Ubuntu images as the base image. I do not provide extra security features in the images. In fact, I do not have the capability to provide security updates for the GraalVM binaries because I am not the maintainer of the GraalVM project.

### Do you provide JDK Docker images?

No, because the official GraalVM Java JDK Docker images are available on the [Oracle Container Registry](https://container-registry.oracle.com/ords/ocr/ba/graalvm/jdk) and you can use them directly.

### Can I make suggestions?

Of course! Feel free to open an issue if you have any suggestions or improvements that you would like to see in the GraalVM Java JRE Docker images.

## License

GraalVM is a trademark of Oracle Corporation and/or its affiliates. Oracle is a registered trademark of Oracle Corporation and/or its affiliates. Other names may be trademarks of their respective owners.

Ubuntu is a trademark of Canonical Ltd.

Oracle GraalVM is licensed under the [GraalVM Free Terms and Conditions (GFTC)](https://www.oracle.com/downloads/licenses/graal-free-license.html) and are free to use in production and free to redistribute at no cost.

All my work in this repository is licensed under the [MIT License](https://opensource.org/licenses/MIT).

This repository is not affiliated with Oracle Corporation.

The project is provided as-is, without any warranty.

## Links

My GitHub repository: <https://github.com/arthlmx814/graalvm-runtime-jre-images>

My Docker Hub repository: <https://hub.docker.com/r/arthlmx814/graalvm-jre>

Oracle GraalVM: <https://www.graalvm.org/>

Oracle GraalVM Free Terms and Conditions (GFTC): <https://www.oracle.com/downloads/licenses/graal-free-license.html>

Eclipse Temurin: <https://adoptium.net/temurin/>

Oracle GraalVM Free Terms and Conditions (GFTC): <https://www.oracle.com/downloads/licenses/graal-free-license.html>

Eclipse Temurin: <https://adoptium.net/temurin/>
