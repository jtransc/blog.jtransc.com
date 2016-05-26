---
layout: post
title:  "First sonatype release"
---

I have finally created a non-snapshot version of jtransc.
First release version is 0.0.1.

[You can find it here](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.jtransc%22l)

<!--more-->

Uploading releases to sonatype requires several steps.

And I'm going to document them so other people can benefit from what I have learned
in that time-consuming process.

## Creating user
First of all you have to create an user in [sonatype jira](https://issues.sonatype.org/)

## Requesting groupId
Then you have to create an issue requesting permissions for your groupId.
As reference, for JTransc I created this issue [OSSRH-18584](https://issues.sonatype.org/browse/OSSRH-18584)

You can buy a domain with the reversed order domain of your group id, or you can use github or similar as base.
You can find a guideline for this [here](http://central.sonatype.org/pages/choosing-your-coordinates.html).

## Configuring pom.xml

In order to publish to sonatype, you need to add their repositories to the distributionManagement section.

{% highlight xml %}
<distributionManagement>
    <repository>
        <id>sonatype-staging</id>
        <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
    </repository>
    <snapshotRepository>
        <id>sonatype-snapshot</id>
        <url>http://oss.sonatype.org/content/repositories/snapshots/</url>
    </snapshotRepository>
</distributionManagement>
{% endhighlight %}

In order to be able to download dependencies from there:

{% highlight xml %}
<repositories>
    <repository>
        <id>sonatype.oss.snapshots</id>
        <name>Sonatype OSS Snapshot Repository</name>
        <url>http://oss.sonatype.org/content/repositories/snapshots</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
{% endhighlight %}

## Configuring servers

You will have to edit `$user/.m2/settings.xml` in order to configure your user/password from sonatype.
You can use plain password or user tokens that you can grab from [Sonatype Nexus Repo](https://oss.sonatype.org/).

{% highlight xml %}
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository/>
    <interactiveMode/>
    <usePluginRegistry/>
    <offline/>
    <pluginGroups/>
    <servers>
      <server>
      	<id>sonatype-staging</id>
      	<username>username</username>
      	<password>password</password>
      </server>
      <server>
      	<id>sonatype-snapshot</id>
      	<username>username</username>
      	<password>password</password>
      </server>
    </servers>
    <mirrors/>
    <proxies/>
    <profiles/>
    <activeProfiles/>
</settings>
{% endhighlight %}

## Publishing snapshots

After validated, you will be able to use `mvn deploy` to upload snapshots.

## Publishing release builds

Publishing releases requires some more work.

You first need to upload to `sonatype-staging`, just using a non-snapshot version
with the configuration above and `mvn deploy` will do the job.

In order to promote a release version from staging you have to ensure your project
meets some requirements: extra pom sections, sign with pgp, include -sources and -javadoc packages.

You can find [a guide for these requirements, here](http://central.sonatype.org/pages/requirements.html).

### Extra sections in pom.xml

You must ensure that your `pom.xml` has at least these sections:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example.applications</groupId>
  <artifactId>example-application</artifactId>
  <version>1.4.7</version>  
  <packaging>jar</packaging>
  <name>Example Application</name>
  <description>A application used as an example on how to set up pushing
  its components to the Central Repository.</description>
  <url>http://www.example.com/example-application</url>
  <licenses>
    <license>
      <name>The Apache License, Version 2.0</name>
      <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
    </license>
  </licenses>
  <developers>
    <developer>
      <name>Manfred Moser</name>
      <email>manfred@sonatype.com</email>
      <organization>Sonatype</organization>
      <organizationUrl>http://www.sonatype.com</organizationUrl>
    </developer>
  </developers>
  <scm>
    <connection>scm:git:git@github.com:example/example-application.git</connection>
    <developerConnection>scm:git:git@github.com:example/example-application.git</developerConnection>
    <url>git@github.com:example/example-application.git</url>
  </scm>

  <dependencies>
    <dependency>
      <groupId>...</groupId>
      <artifactId>...</artifactId>
      <version>...</version>
    </dependency>
    ...
  </dependencies>

</project>
{% endhighlight %}

### sign with pgp

In my case never used PGP. In OSX you must install it `brew gpg`.

`gpg --gen-key`

Your key must be RSA 2048-bit

`gpg --armor --export alice@cyb.org`

{% highlight xml %}
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v0.9.7 (GNU/Linux)
Comment: For info see http://www.gnupg.org

[...]
-----END PGP PUBLIC KEY BLOCK-----
{% endhighlight %}

You will have to upload the public key block to one public server. Mine was:
[http://keyserver.ubuntu.com:11371/pks/add](http://keyserver.ubuntu.com:11371/pks/add)

In your pom.xml you will have to add:

{% highlight xml %}
<build><plugins>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-gpg-plugin</artifactId>
    <executions>
        <execution>
            <id>sign-artifacts</id>
            <phase>verify</phase>
            <goals>
                <goal>sign</goal>
            </goals>
        </execution>
    </executions>
</plugin>
</plugins></build>
{% endhighlight %}

It will request your password while performing `mvn deploy`.

To avoid having to type your password everytime, you can:

`mvn clean deploy -Dgpg.passphrase=yourpassphrase`

Though this will put your password in `.bash-history` and similar files.

### including -sources and -javadoc packages

In my case, with kotlin I have the following plugins:

{% highlight xml %}
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <executions>
        <execution>
            <id>attach-sources</id>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <executions>
        <execution>
            <id>empty-javadoc-jar</id>
            <phase>package</phase>
            <goals>
                <goal>jar</goal>
            </goals>
            <configuration>
                <classifier>javadoc</classifier>
                <classesDirectory>${basedir}/javadoc</classesDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

## releasing staged artifacts

You can see jtransc [pom.xml file](https://github.com/jtransc/jtransc/blob/master/pom.xml).

You have to go to sonatype nexus: [https://oss.sonatype.org/](https://oss.sonatype.org/)

At the sidebar: Sonatype (TM) -> Build Promotion -> Staging Repositories

After the `mvn deploy` you will be able to see there your release.
You have to select it, and check at the bottom file viewer if it's all ok.
In the activity tab you will be able to see if there were problems validating your release
(all the requirements I put above).

Then you have to select your release, and press the close button above.

After that, your release will be closed, and you will be able to press the release button.

In the case you did something wrong, you should press the drop button instead.
