# swe-metrics



## Cyclic





## LCOM

https://www.ndepend.com/api/NDepend.API~NDepend.CodeModel.IType~LCOM.html

https://blog.ndepend.com/lack-of-cohesion-methods/#:~:text=LCOM%20for%20a%20class%20will,mean%20a%20lot%20of%20cohesion.

https://github.com/Mkohm/detekt-hint/blob/master/src/main/kotlin/io/github/mkohm/detekt/hint/rules/LackOfCohesionMethods.kt#L29



Build 



```
git clone https://github.com/Mkohm/detekt-hint ./detekt/detekt-hint/ 
git clone https://github.com/arturbosch/detekt ./detekt/detekt/
cd ./detekt/detekt-hint/ 
git checkout 959d3c2ca4bd65ee300ba4f4d71ea59183444113
./gradlew jar 
cd ../detekt/ 
git checkout 9a35a3aec418d234d9c889a91895e19a9f9b567e
./gradlew build shadowJar
cd ../..
```

Run

Original

```bash
java -jar ./detekt/detekt/detekt-cli/build/libs/detekt-cli-1.22.0-RC1-all.jar --plugins ./detekt/detekt-hint/build/libs/detekt-hint.jar --config ./config/detekt-hint.yml --classpath ./original/Kotlin/src --input ./original/Kotlin/src --report txt:reports/original.txt --report html:reports/original.html
```

Refactored

```bash
java -jar ./detekt/detekt/detekt-cli/build/libs/detekt-cli-1.22.0-RC1-all.jar --plugins ./detekt/detekt-hint/build/libs/detekt-hint.jar --config ./config/detekt-hint.yml --classpath ./refactored/Kotlin/src --input ./refactored/Kotlin/src --report txt:reports/refactored.txt --report html:reports/refactored.html
```

reports are found under `/reports`, threshold can be adapted under 

### Changes to `detekt-hint`

`gradle/wrapper/gradle-wrapper.properties`:

```
- distributionUrl=https\://services.gradle.org/distributions/gradle-5.2.1-bin.zip
+ distributionUrl=https\://services.gradle.org/distributions/gradle-7.5.1-all.zip
```

`build.gradle`:

```
- id "com.vanniktech.maven.publish" version "0.8.0"
- A new dokka task for generating github flavoured markdown to the docs directory
- task customDokkaTask(type: org.jetbrains.dokka.gradle.DokkaTask) {
-    outputFormat = "jekyll"
-    outputDirectory = "$projectDir/docs"
- }
```



## Size





Maintainability Index

* Definition
* Nur auf Beispiel von Gilded Rose
* Erklären, dass es kein Program gibt (wie für Java order C#), Cyclic Complexity und Lines of Code kriegen wir aber
* Händisch kalkulieren
* Profit

https://learn.microsoft.com/en-us/visualstudio/code-quality/code-metrics-maintainability-index-range-and-meaning?view=vs-2022

https://stackoverflow.com/questions/2936814/visual-studio-code-metrics-and-the-maintainability-index-of-switch-case

https://www.youtube.com/watch?v=Wdq8YOiLF7o

https://en.wikipedia.org/wiki/Halstead_complexity_measures

https://en.wikipedia.org/wiki/Halstead_complexity_measures

https://docplayer.org/66261984-7-metriken-idee-von-masssystemen-halstead-live-variables-variablenspanne-mccabe-zahl-lcom.html
