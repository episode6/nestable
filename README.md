Nestable for Gradle Plugins
===========================
Easier nestable extensions for gradle plugins.

### Why?
When creating a gradle plugin, you often want to define an extension for users to configure your plugin. If you want this extension to be a struct that contains other structs, then this is a PITA. If you want these properties to also be settable via gradle.properties, it's an even bigger PITA.

Nestable tries to make this a little bit easier.

### Usage
Lets say we want to create a plugin extension that is definable in the following 2 ways
```
# gradle.properties
rootnamespace.subspace1.param1=myParam1
rootnamespace.subspace1.param2=myParam2
rootnamespace.subspace1.subspace3.yParam=myYParam
rootnamespace.subspace2.xParam=myXParam
```
OR
```
// build.gradle
// NOTE: setting extension vars in gradle will override gradle.properties
rootnamespace {
  subspace1 {
    param1 "myParam1" // notice that the equals signs are optional
    param2 "myParam2"
    subspace3 {
      yParam = "myYParam"
    }
  }
  subspace2 {
    xParam = "myXParam"
  }
}
```

Here's how we do it...

Add `compile 'com.episode6.hackit.nestable:nestable:0.0.1'` to your plugin's dependencies.

Implement your own root plugin extension. You'll have to define some special methods here for the top-level nestable extensions, but only in the root plugin
```groovy
class RootExtension extends NestablePluginExtension {

    // both of these are NestablePluginExtensions we define below
    SubGroup1 subgroup1
    SubGroup2 subgroup2

    RootExtension(Project project) {
        super(project, "rootnamespace")
        subgroup1 = new SubGroup1(this)
        subgroup2 = new SubGroup2(this)
    }

    // you only need the following methods in your root-level plugin

    SubGroup1 subgroup1(Closure closure) {
        return subgroup1.applyClosure(closure)
    }

    SubGroup2 subgroup2(Closure closure) {
        return subgroup2.applyClosure(closure)
    }
}
```

Then define your child plugins
 ```groovy
class SubGroup1 extends NestablePluginExtension {
    String param1
    String param2
    SubGroup3 subgroup3 // another nestable extension

    SubGroup1(NestablePluginExtension parent) {
        // Here i define the sub-namespace of this plugin. If you want to
        // re-use the same struct in multiple places, simply pass the
        // newNamespace String to the constructor
        super(parent, "subspace1")
    }

    // Notice, we don't need those extra closure methods in a child-extension
}

class SubGroup2 extends NestablePluginExtension {
    String xParam

    SubGroup2(NestablePluginExtension parent) {
        super(parent, "subspace2")
    }
}

class SubGroup3 extends NestablePluginExtension {
    String yParam

    SubGroup3(NestablePluginExtension parent) {
        super(parent, "subspace3")
    }
}
```

In your plugin code, add your extension (it's recommended that your extension be named the same as the root namespace of your extension so that the syntax matches)
```groovy
class MyPlugin implements Plugin<Project> {
    void apply(Project project) {
        RootExtension rootExtension = project.extensions.create(
            "rootnamespace",
            RootExtension,
            project)
    }
}
```

That's about it.

### License
MIT: https://github.com/episode6/nestable/blob/master/LICENSE