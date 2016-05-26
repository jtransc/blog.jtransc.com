---
layout: post
title: "New examples repository: #JTransc + #Gradle + #Libgdx + #Box2D + #Spine + #Kotlin + #HTML5"
---

<img src="/img/examples/spine_demo.png" width="450" height="450" />

I have added new libgdx and spine demos to the repository using gradle.
Also I have created a new repository with example binaries, so you can access an URL and test it yourself.

Also I have added the libgdx box2d extension to be used with jtransc.

You can access the repository here: [https://github.com/jtransc/jtransc-examples-binaries](https://github.com/jtransc/jtransc-examples-binaries)

Available HTML5 examples transcompiled using [JTransc](https://github.com/jtransc/jtransc) and [gdx-backend-jtransc](https://github.com/jtransc/gdx-backend-jtransc)):
Note that without modifications you can generate C++ native versions for: Windows, Linux , Mac, Android and iOS using jtransc backend (that uses [Haxe](http://haxe.org/) and [Lime](https://github.com/openfl/lime)).

* **[Libgdx Cuboc Demo](http://jtransc.github.io/jtransc-examples-binaries/cuboc/index.html)** - [(Source Code)](https://github.com/jtransc/jtransc-examples/tree/master/libgdx/cuboc)
* **[Libgdx Vector Pinball Demo (Box2D)](http://jtransc.github.io/jtransc-examples-binaries/vector-pinball/index.html)** - [(Source Code)](https://github.com/jtransc/jtransc-examples/tree/master/libgdx/vector-pinball)
* **[Spine+Libgdx Demo 1](http://jtransc.github.io/jtransc-examples-binaries/spine1/index.html)** - [(Source Code)](https://github.com/jtransc/jtransc-examples/tree/master/spine-demo)
* **[Spine+Libgdx Demo 2](http://jtransc.github.io/jtransc-examples-binaries/spine2/index.html)** - [(Source Code)](https://github.com/jtransc/jtransc-examples/tree/master/spine-demo)

<iframe src="http://jtransc.github.io/jtransc-examples-binaries/spine1/index.html" width="450" height="450" style="border:0;"></iframe>

Also note that those examples **do not require code modifications/extra mains at all!** The desktop (lwjgl version) is compiled with JTransc and it works.
And that the same gradle file allows you to run on JVM and JTransc with several targets.

Right now performance is not the best in class, but that will improve in future JTransc versions.

## Examples:
* `gradle run` will run on the JVM
* `gradle runHtml5` will generate HTML5 and would open a browser
* `gradle distWindows` will generate a release executable (.exe) for windows
* `gradle distAndroid` will generate a C++ (.apk) for android. So no method count limit! And predictable performance (no Dalvik or ART). And also supports Kotlin and some java8 features.

## How does it work?

There is a plugin to use jtransc with gradle. You just include it.
Later you define all your dependencies as if it was going to be libgdx for desktop (using lwjgl).
But there is a configuration called "jtransc" instead of "compile" that is prepended just when using JTransc.
So all the classes defined in jtransc will have priority over later ones. So one single build.gradle.
And one single source folder, and one single assets folder.

{% highlight gradle %}
buildscript {
  repositories {
    mavenLocal()
    mavenCentral()
  }
  dependencies {
    classpath "com.jtransc:jtransc-gradle-plugin:$jtranscVersion"
  }
}
dependencies {
  jtransc "com.jtransc.gdx:gdx-backend-jtransc:$jtranscVersion"
  jtransc "com.jtransc.gdx:gdx-box2d-jtransc:$jtranscVersion"

  compile "com.badlogicgames.gdx:gdx-backend-lwjgl:$libgdxVersion"
  compile "com.badlogicgames.gdx:gdx-platform:$libgdxVersion:natives-desktop"
  compile "com.badlogicgames.gdx:gdx-box2d:$libgdxVersion"
  compile "com.badlogicgames.gdx:gdx-box2d-platform:$libgdxVersion:natives-desktop"

  testCompile group: 'junit', name: 'junit', version: '4.+'
}
{% endhighlight %}

Then you can configure jtransc itself (application configuration and jtransc specific stuff):

{% highlight gradle %}
jtransc {
  // Optional properties (https://github.com/jtransc/jtransc/blob/master/jtransc-gradle-plugin/src/com/jtransc/gradle/JTranscExtension.kt)
  title = "Vector Pinball"
  name = "VectorPinball"
  version = "0.0.1"
  company = "Badlogic"
  package_ = "com.jtransc.gdx.examples"
  embedResources = true
  assets = ['libgdx-demo-vector-pinball/android/assets']
  vsync = true
  relooper = true
  minimizeNames = false
  analyzer = false

  customTarget("cpp", "haxe:cpp", "exe")
  customTarget("windows", "haxe:windows", "exe")
  customTarget("linux", "haxe:linux", "bin")
  customTarget("mac", "haxe:mac", "app")
  customTarget("android", "haxe:android", "apk")
  customTarget("ios", "haxe:ios", "ipa")
  customTarget("tizen", "haxe:tizen", "app")
  customTargetMinimized("html5", "haxe:html5", "js")
}
{% endhighlight %}

You can find [jtransc-gradle documentation here](https://github.com/jtransc/jtransc/wiki/Gradle).
