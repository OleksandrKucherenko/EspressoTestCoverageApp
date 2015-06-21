# EspressoTestCoverageApp

Sample project that demonstrate how to run Espresso tests with enabled code coverage.

# Android Test + Espresso + JaCoCo

If you try to run Espresso tests on device and at the same time grab the code coverage information, than your are looking on the right topic... Its all about this.

Unfortunately JaCoCo + Espresso does not work for everyone. 

Quick search will reveal a planty of reported issues and mostly no working solutions. I also ignore this problem till Michenux contact me and ask for help. (Good for us all in Sweden is a "red date" in calendar and I have some free time)

The issue and why it exists?
AndroidTest configuration produce a special APK. Inside it included source code of the original binary, libraries and instrumentation. Problem is that Android Dalvik is not 100% compatible JVM, so it does not support JVM javaagents (its the way how originally JaCoCo/Emma should be used with Java). So everything we need is inside the APK, but we cannot able to run it properly.           

That is why inside Android should be used a special way for code coverage instruments. Its known as a "JaCoCo offline mode". In two words - this is a pre-processing of *.class files and embedding into them of special Jacoco method, that executed from class constructor. So when you execute the tests, Jacoco is able to identify that and collect runtime information.

Example: ![JaCoCo Offline Mode][jacoco-offlin]

Android build system should do that all, but looks like it fails at the current moment. 

Steps that build system should do:
1) download and install dependency JaCoCo library - DONE
2) include JaCoCo Agent into APK - DONE
3) create classes with enabled jacoco (offline mode) - DONE
4) replace original files by "jacoco enabled" - NOT WORKING
5) define coverage result output path - NOT WORKING
6) run tests - DONE
7) PULL coverage file from device - NO
8) Generate HTML coverage report - NO

## The Solution
all in this Gradle file: [App Build Gradle File][build.gradle]

*Step #1:* use the latest version of JaCoCo ("0.7.5.201505241946")

*Step #2:* Destination path can be defined by resource file - jacoco-agent.properties (~/src/androidTest/resources/jacoco-agent.properties). [for alternative look inside the org.michenux.espressotestcoverageapp.AndroidJacocoTestRunner class]

> destfile=/data/data/{package.name}/coverage.ec

OR _(this way you should use only if you has a extra custom build logic)_

> destfile =/sdcard/coverage.ec

*Step #3:* Modify build sequence. Place own customization tasks between "preDex${flavor}${buildType}AndroidTest" and "dex${flavor}${buildType}AndroidTest". I name them: "fixJacocoAgentAndroidTest${flavor}${buildType}" and "fixJacocoAndroidTest${flavor}${buildType}"

*Step #3.1:*
"fixJacocoAgentAndroidTest${flavor}${buildType}"  - this task include jacocoagent.jar into final binary (simply copy JAR into "${project.buildDir}/intermediates/pre-dexed/androidTest/${flavor}/${buildType}" folder). After that other tasks will use it during the compilation. 

*Step #3.2:*
"fixJacocoAndroidTest${flavor}${buildType}" - copy "jacoco enabled" classes on top of old classes.

*Step #4:*
Run the project. Inside the app folder you will find a "coverage.ec" with non-zero size. 

If you run the tests from command line:
```bash
gradlew :app:connectedAndroidTest
```

than your coverage report will be in folder:
```
EspressoTestCoverageApp\app\build\reports\coverage\debug\ 
```

[jacoco-offlin]: https://raw.githubusercontent.com/OleksandrKucherenko/EspressoTestCoverageApp/master/jacoco-offline.png
[build.gradle]: https://github.com/OleksandrKucherenko/EspressoTestCoverageApp/blob/jacoco/app/build.gradle
