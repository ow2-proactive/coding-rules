# coding-rules

Configuration files used to enforce our coding guidelines / formatting

## Clone this project
* git clone this project somewhere on your PC

## Intellij User

* Use File > Import settings and select settings.jar to import the settings in IntelliJ IDEA.
* (Optional if you have already the "Eclipse code formatter" plugin installed) Install Eclipse Code Formatter plugin from the Plugins dialog screen:

  * on MacOS: Preferences -> Plugins
  * on GNU/Linux: File -> Settings -> Plugins
  * on Microsoft Windows: File -> Settings
  
  Click on Browse repositories, search for "Eclipse Code Formatter" and click on Install.
  To load this new plugin, you will need to restart IntelliJ.
* Open up the Eclipse Code Formatter dialog screen:

  * on MacOS: Preferences -> Eclipse Code Formatter
  * on GNU/Linux: File -> Settings -> Eclipse Code Formatter
  * on Microsoft Windows: File -> Settings -> Other Settings -> Eclipse Code Formatter
  
* Enable the plugin by ticking "Use the Eclipse code formatter"
* Setting "Eclipse Java Formatter config file" by using the `src/main/resource/ProactiveCodeFormatter.xml`
* Make sure the ProActive Rules is selected as "Java formatter profile"
* Tick "Optimize Imports"
* Select "Import order from file", which uses `src/main/resources/proactive.importorder`
* Click on Apply or OK
* Make sure that the selected Code Style Scheme is "Proactive" by opening the Code Style dialog screen:

  * on MacOS: Preferences -> Editor -> Code Style
  * on GNU/Linux: File -> Settings -> Editor -> Code Style

## Eclipse User

Download Eclipse_Preferences.epf file (https://github.com/ow2-proactive/coding-rules/blob/master/Eclipse_Preferences.epf), and import it by doing in Eclipse: File > Import > General > Preferences, browse to file and Finish. This epf file contains already the ProactiveCodeFormatter.xml, no need to import the formatter again.


## Code format during the project build

1.Merge the following buildscript into your project's build.gradle file

```
buildscript {
    repositories {
        maven { 
            url "https://plugins.gradle.org/m2/"
        }
        maven {        
            url "http://repository.activeeon.com/content/groups/proactive/"
        }       
    }

    dependencies {
        classpath "com.diffplug.gradle.spotless:spotless:2.4.0"
        classpath "org.ow2.proactive:coding-rules:1.0.0"
        delete "gradle/ext"
        ant.unjar src: configurations.classpath.find { it.name.startsWith("coding-rules") }, dest: 'gradle/ext'
    }
}
```

2.Apply the code format plugin in your project's build.gradle file with

```
apply from: "$rootDir/gradle/ext/coding-format.gradle"
```

3.Update your project's .gitignore by adding the line below to ignore the temporary folder

```
gradle/ext/
```
4.Build your project as usual, if any java files are bad formatted, build will fail with messages telling you to build again with task "spotlessApply".

5.Build again with either the custom task "formatCode" (defined in coding-format.gradle) or plugin task "spotlessApply" will automatically format your whole project.

## Issue you may have after having applied the changes

### 1. gradle build failure
Some projects may have a build exception after having applied the changes above, the exception message looks like the following

```
An exception occurred applying plugin request [id: 'org.sonarqube', version: '2.2.1']
> Cannot change dependencies of configuration ':classpath' after it has been resolved.
```

This is because the sonarqube plugin is applied within the "plugins" block like this

```
plugins {
    id 'org.sonarqube' version '2.2.1'
}
```

The only way to solve this issue so far is to apply the plugin in the traditional manner, add dependencies in the "buildscript" block like this

```
dependencies {
        classpath "org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:2.2.1"
    }
```

then apply the plugin like this

```
apply plugin: 'org.sonarqube'
```

### 2. Code format check failure in windows
Due to the fact that the line ending character used in windows and in linux is different, the code format check might be failed in Windows environment.

The fix is to create `.gitattributes` file in the project's root path with the following content, this will force git to use the fix line ending character while checking out the project.

```
*.java eol=lf
*.gradle eol=lf
*.sh eol=lf
*.md eol=lf

*.bat eol=crlf

*.png binary
*.jpg binary
```

Run the following git commands on the jenkins windows slave machine to apply the `.gitattributes` rules inside the git local working directory, otherwise the current working directory will still have the windows line ending character.

1. `git rm --cached -r .`
2. `git reset --hard`

The jenkins workspace directory is kind of like `C:\jenkins\workspace\${project_name}\jdk\JDK8\label\Windows`
