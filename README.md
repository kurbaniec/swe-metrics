### Kacper Urbaniec | SWE | 24.09.2022

# Assignment 1b: Metrics

*Apply a CC, LCOM, and Size Metric on the original code of the previous exercise, and apply the same metrics also on your refactored version of it. In addition research the Maintainability Index, and calculate it. Provide your interpretation of the given results.*

## Cyclic Complexity



static code analysis tool for Kotlin *[detekt](https://github.com/detekt/detekt)*

## LCOM

LCOM stands for *Lack of Cohesion Of Methods* and is a measure of cohesion of methods in a class. [i](https://blog.ndepend.com/lack-of-cohesion-methods/#:~:text=LCOM%20for%20a%20class%20will,mean%20a%20lot%20of%20cohesion.)

LCOM is associated with the *Single Responsibility Principle* (SRP) in the sense that a cohesive class will use all its properties regularly. For example, if half of the methods use a property *a* and the other half use a property *b*, then the cohesion of the class may not be very strong and splitting the class into two classes may be a better choice. [i](https://blog.ndepend.com/lack-of-cohesion-methods/#:~:text=LCOM%20for%20a%20class%20will,mean%20a%20lot%20of%20cohesion.)

### Formula

There are many different LCOM versions and calculation formulas. The one described here is based the algorithm used in *[NDepend](https://www.ndepend.com/api/NDepend.API~NDepend.CodeModel.IType~LCOM.html)* and *[detekt-hint](https://github.com/Mkohm/detekt-hint/blob/master/src/main/kotlin/io/github/mkohm/detekt/hint/rules/LackOfCohesionMethods.kt#L29)*.

* LCOM = 1 - (sum(MP)/M*P)

Where:

- M is the number of methods in class (it includes also constructors).
- P is the number of instance properties in the class.
- MP is the number of methods of the class accessing a particular instance property.
- Sum(MP) is the sum of MP over all instance properties of the class.

A high LCOM value generally pinpoints a poorly cohesive class.

### Results

Both original and refactored code contain only two classes with instance properties: `GlidedRose` & `Item`.

The LCOM value was calculated once again using *detekt*, however the plugin *[detekt-hint](https://github.com/Mkohm/detekt-hint)* was needed, as LCOM calculation is not part of the core library.

These are the results:

* Original
  * `GildedRose`
    * LCOM value: 0.0
    * M: 2
    * P: 1
    * Sum(MP): 2
  * `Item`
    * LCOM value: 0.0
    * M: 2
    * P: 3
    * Sum(MP): 6
* Refactored
  * `GildedRose`
    * LCOM value: 0.75
    * M: 2
    * P: 4
    * Sum(MP): 2
  * `Item`
    * LCOM value: 0.0
    * M: 2
    * P: 3
    * Sum(MP): 6

Detekt-hint counts private and public properties; static properties (via companion objects) are also counted. 

However, when inspecting the refactored results I realized that private methods where not counted by detekt-hint. I am not sure if it is the correct approach for LCOM, but it seems intended (according to the [source code](https://github.com/Mkohm/detekt-hint/blob/master/src/main/kotlin/io/github/mkohm/detekt/hint/rules/LackOfCohesionMethods.kt#L82)). For sake of completion, I also run the analyser with the private methods set to public and got comparable results:

* Refactored (including private methods)
  * `GildedRose`
    * LCOM value: 0.75
    * M: 4
    * P: 4
    * Sum(MP): 4

### Interpretation

When one looks only at the LCOM metric, then one would argue that no refactoring of the original code is needed as both classes have perfect cohesion.

However, when looking at the original code one instantly see that the code is not in a good shape. But it has more of a complexity problem than one with cohesion so the metric is right in this sense.

When looking at the LCOM metric for my refactored `GildedRose` class I was quite suprised how "low cohesive" it was declared.

```kotlin
class GildedRose(var items: Array<Item>) {
    private val factory = SimpleItemUpdaterFactory()

    companion object Constants {
        const val MAX_QUALITY = 50
        const val MIN_QUALITY = 0
    }

    fun updateQuality() {
        items.forEach { item ->
            update(item)
        }
    }

    private fun update(item: Item) {
        getItemUpdater(item)
            .updateSellIn(item)
            .updateQuality(item)
    }

    private fun getItemUpdater(item: Item): ItemUpdater {
        return factory.create(item)
    }
}
```

I double checked the formula, calculated by hand, but the results are right: The LCOM metric for the class is truly 0.75. 

I thought about it a little bit and came up with some simple adjustments to the class that affect the metric quite heavily.

* Move `Constants` out of the class (P -= 2)
* Add `factory` property to the constructor (Sum(MP) += 1)

```kotlin
object Constants {
	const val MAX_QUALITY = 50
    const val MIN_QUALITY = 0
}

class GildedRose(
	var items: Array<Item>,
	private val factory = SimpleItemUpdaterFactory()
) { 
	...
}
```

With these adjustments the LCOM metric for `GildedRose` is now as follows:

* LCOM value: 1 - (3 / (2 x 2)) = 0.25
* M: 2
* P: 2
* Sum(MP): 3

I do not think that these adjustments change the maintainability of the code in any really meaningful way, however the LCOM value is drastically changed.

In my opinion, this could lead to many classes being declared non-cohesive, even though this is not the case. If such simple restructuring has such a drastic effect on the metric, then I don't think it is that stable. During research I also found out that Sonarqube deprecated the LCOM4 metric because of too many false-positives. [1](https://sonarsource.atlassian.net/browse/SONAR-4853)

It is also interesting to note that in Kotlin, properties are abstractions over instance fields. That is, in classical Java, each Kotlin property consists of a private field with a setter and a getter method. This means six additional methods and would make the class `item` not so cohesive according to LCOM:

* LCOM value: 1 - (12 / (8 x 3)) = 0.5
* M: 8
* P: 3
* Sum(MP): 12

That means that the LCOM metric is not suitable for pure data classes (DTOs/Models/DAOs) and would often lead to non-cohesive assessments.

## Size





## Maintainability Index

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







## Build

```
git submodule init
```

### LCOM

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



