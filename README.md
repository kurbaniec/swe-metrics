### Kacper Urbaniec | SWE | 24.09.2022

# Assignment 1b: Metrics

*Apply a CC, LCOM, and Size Metric on the original code of the previous exercise, and apply the same metrics also on your refactored version of it. In addition research the Maintainability Index, and calculate it. Provide your interpretation of the given results.*

## Cyclic Complexity

Cyclomatic complexity (CC or MCC) is a software metric used to indicate the complexity of a program. It is a quantitative measure of the number of linearly independent paths through a program's source code. It was developed by Thomas J. McCabe, Sr. in 1976. [i](https://en.wikipedia.org/wiki/Cyclomatic_complexity)

### Formula

Mathematically, the cyclomatic complexity of a structured program is defined with reference to the control-flow graph of the program, a directed graph containing the basic blocks of the program, with an edge between two basic blocks if control may pass from the first to the second.  The complexity M is then defined as follows. [i](https://en.wikipedia.org/wiki/Cyclomatic_complexity)

* M = E - N + 2P

where

* E = the number of edges of the graph
* N = the number of nodes of the graph
* P = the number of connected components

Practically, the CC of a method is 1 + the number of expressions such as `if`, `while`, `for`, `continue` and so on. Expressions such as `else`, `try`, `return`, etc. are not counted for the CC. Method calls or property accesses are also not considered. [i](https://www.ndepend.com/docs/code-metrics#CC), [ii](https://detekt.dev/docs/rules/complexity/#complexmethod)

Methods where CC is higher than 15 are hard to understand and maintain. Methods where CC is higher than 30 are extremely complex and should be split into smaller methods (except if they are automatically generated by a tool). [i](https://www.ndepend.com/docs/code-metrics#CC)

### Results

The CC was calculated using the static code analysis tool for Kotlin *[detekt](https://github.com/detekt/detekt)*. The library can calculate many metrics. The CC can be processed using its *[ComplexMethod](https://detekt.dev/docs/rules/complexity/#complexmethod)* rule. I configured the threshold for the rule to 0 to display the CC metric for all methods.

These are the results that containin the total Cyclomatic Complexity (all methods added together) and the method with the highest complexity:

* Original
  * Total Cyclic Complexity: 20
  * Highest: 19, `GildedRose.updateQuality`
* Refactored
  * Total Cyclic Complexity: 42
  * Highest: 7, `ItemExtensions.limitQuality`
  * `GildedRose.updateQuality` has now a CC of 2

> 🪧 Autogenerated reports with complete results can be found in the `reports` directory.

### Interpretation

The CC of the original code, especially the `updateQuality`, was determined quite high which is exactly correct. The original code is mostly unreadable because the many if statements and the recommendation to split into smaller methods is spot on.

The disadvantage of the metric, in my opinion, is that when code is split into smaller methods or extracted into its own classes, the total CC increases. In comparison, the new total CC is 42, while the old one was 20. The highest value of the new single method is 7, but if one only looks at the total number, one cannot derive that. Therefore, the metric is only useful on a method-to-method basis and not really on whole project levels.

I have also another gripe about the metric. Look at the following method. Its CC is reported as 6.

```kotlin
private fun getItemUpdater(lowercaseName: String): ItemUpdater {
	return when {
        lowercaseName.contains("aged brie") -> AgedBrieUpdater
        lowercaseName.contains("sulfuras") -> SulfurasUpdater
        lowercaseName.contains("backstage passes") -> BackstagePassesUpdater
        lowercaseName.contains("conjured") -> ConjuredUpdater
        else -> BaseItemUpdater
	}
}
```

The `when` statement and the four conditional branches are counted to the baseline 1, resulting in a CC of 6. But isn't that a rather high number for something that can't really be simplified any more? 

For this reason, there are new metrics that try to address the shortcomings of CC and provide a better way to quantify the whole project. One of these is the Cognitive Complexity introduced by SonarSource. The above code snippet would be rated 1 for Cognitive Complexity as it is the simplest idiomatic approach to the problem. [i](https://www.sonarsource.com/docs/CognitiveComplexity.pdf), [ii](https://github.com/detekt/detekt/blob/main/detekt-metrics/src/main/kotlin/io/github/detekt/metrics/CognitiveComplexity.kt#L31)

Detekt supports the Cognitive Complexity metric and the results show that the refactored process is indeed more readable:

* Original
  * Total Cognitive Complexity: 66
* Refactored
  * Total Cognitive Complexity: 7

## LCOM

LCOM stands for *Lack of Cohesion Of Methods* and is a measure of cohesion of methods in a class. [i](https://blog.ndepend.com/lack-of-cohesion-methods/#:~:text=LCOM%20for%20a%20class%20will,mean%20a%20lot%20of%20cohesion.)

LCOM is associated with the *Single Responsibility Principle* (SRP) in the sense that a cohesive class will use all its properties regularly. For example, if half of the methods use a property *a* and the other half use a property *b*, then the cohesion of the class may not be very strong and splitting the class into two classes may be a better choice. [i](https://blog.ndepend.com/lack-of-cohesion-methods/#:~:text=LCOM%20for%20a%20class%20will,mean%20a%20lot%20of%20cohesion.)

### Formula

There are many different LCOM versions and calculation formulas. The one described here is based the algorithm used in *[NDepend](https://www.ndepend.com/api/NDepend.API~NDepend.CodeModel.IType~LCOM.html)* and *[detekt-hint](https://github.com/Mkohm/detekt-hint/blob/master/src/main/kotlin/io/github/mkohm/detekt/hint/rules/LackOfCohesionMethods.kt#L29)*.

* LCOM = 1 - (sum(MP) / (M * P))

Where:

- M is the number of methods in class (it includes also constructors).
- P is the number of instance properties in the class.
- MP is the number of methods of the class accessing a particular instance property.
- Sum(MP) is the sum of MP over all instance properties of the class.

A high LCOM value generally pinpoints a poorly cohesive class.

Also, when counting the LCOM value in a method, one must take into account all the methods that are called in it. For example, if a method `foo` does not use any properties but calls `bar` which accesses a property, then `foo` has MP=1 because of the `bar` invocation.

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

With these adjustments the LCOM metric for `GildedRose` is now as follows:

* LCOM value: 1 - (3 / (2 * 2)) = 0.25
* M: 2
* P: 2
* Sum(MP): 3

I do not think that these adjustments change the maintainability of the code in any really meaningful way, however the LCOM value is drastically changed.

In my opinion, this could lead to many classes being declared non-cohesive, even though this is not the case. If such simple restructuring has such a drastic effect on the metric, then I don't think it is that stable. During research I also found out that Sonarqube deprecated the LCOM4 metric because of too many false-positives. [i](https://sonarsource.atlassian.net/browse/SONAR-4853)

It is also interesting to note that in Kotlin, properties are abstractions over instance fields. That is, in classical Java, each Kotlin property consists of a private field with a setter and a getter method. This means six additional methods and would make the class `item` not so cohesive according to LCOM:

* LCOM value: 1 - (12 / (8 * 3)) = 0.5
* M: 8
* P: 3
* Sum(MP): 12

That means that the LCOM metric is not suitable for pure data classes (DTOs/Models/DAOs) and would often lead to non-cohesive assessments.

## Size

There are many metrics in the size domain. The simplest size metrics for software projects are probably *Lines of Code* (LOC), *Source Lines of Code* (SLOC) and *Logical Lines of Code* (LLOC). [i](https://en.wikipedia.org/wiki/Source_lines_of_code), [ii](https://de.wikipedia.org/wiki/Lines_of_Code), [iii](https://www.codingninjas.com/codestudio/library/size-oriented-metrics-in-software-engineering)

### Types

LOC measures all lines including blank lines and comments.

SLOC measures all lines excluding blank lines and comments.

LLOC is somewhat special, it does not measure lines but statements like loops or invocations.

However, one should be careful when comparing two identical size metrics counted by different tools, as the implementations may have a large number of different definitions. For example, who decides what a statement is in LLOC? [i](https://en.wikipedia.org/wiki/Source_lines_of_code)

The following shows two examples and what one can expect for the size metrics.

```kotlin
for (i in 0 until 100) println(i) // How many lines?
```

* 1 LOC
* 1 SLOC
* 2 LLOC (`for` and `println` statements)

```kotlin
/* How many lines? */
for (i in 0 until 100) {
	println(1)
}
```

* 4 LOC
* 3 SLOC
* 2 LLOC (`for` and `println` statements)

### Results

The previously size metrics were calculated once again using *detekt*. 

These are the results:

* Original
  * LOC: 66 
  * SLOC: 56
  * LLOC: 33
* Refactored
  * LOC: 266
  * SLOC: 179
  * LLOC: 99

This leads to the following size increases (from the original to the refactored project):

* LOC: 303 % increase
* SLOC: 220 % increase
* LLOC: 200 % increase

### Interpretation

I could imagine that these size metrics, especially SLOC and LLOC, could be useful in determining the size or cost of a product. From  historic data, a company could conclude that a small feature requires *x* lines, a medium one requires *y* lines and a large one requires *z* lines. Combining all the features gives a size metric that can be converted into development time and development cost. 

In my opinion, however, this has the flaw that a large or sophisticated feature does not automatically mean many lines of code. Sometimes sophisticated solutions are just a few lines of code that take hours, but there are also generic data transfer objects with almost a hundred lines of code that might be automatically generated.

I have the same feeling about the results related to the original and the refactored project. What does a 200% increase in SLOC or LLOC mean? Is the code more complex now? According to Cognitive Complexity, it is not. So without knowing the code, the metrics don't really provide any meaningful data, except that it seems that the refactored project consist of more lines, which could mean more classes.

## Maintainability Index

The Maintainability Index was introduced in 1992 by Paul Oman and Jack Hagemeister.  It is a single-value indicator for the maintainability of a software system. [i](https://www.cqse.eu/en/news/blog/maintainability-index/#ref-1)

### Formula

The Maintainability Index is a blend of several metrics, including *Halstead’s Volume* (HV), *McCabe’s Cylcomatic Complexity* (CC), *Lines of Code* (LOC), and *Percentage of Comments* (COM). For these metrics, the average per module is taken, and combined into a single formula. [i](https://avandeursen.com/2014/08/29/think-twice-before-using-the-maintainability-index/), [ii](https://www.cqse.eu/en/news/blog/maintainability-index/#ref-1)

* Maintainability Index = 171 - 5.2 * ln(Halstead Volume) - 0.23 * (Cyclomatic Complexity) - 16.2 * ln(Lines of Code) + 50 * sin(sqrt(2.46 * Percentage of Comments))

However, the Maintainability Index known today is the version widely used in Visual Studio, where the *Percentage of Comments* metric is omitted and other adjustments are made so that the value returned is in a fixed range between 0 and 100. [i](https://learn.microsoft.com/en-us/visualstudio/code-quality/code-metrics-maintainability-index-range-and-meaning?view=vs-2022), [ii](https://avandeursen.com/2014/08/29/think-twice-before-using-the-maintainability-index/)

* Maintainability Index = MAX(0,(171 - 5.2 * ln(Halstead Volume) - 0.23 * (Cyclomatic Complexity) - 16.2 * ln(Lines of Code))*100 / 171)

Visual Studio interprets the returned Maintainability Index as follows:

* MI >= 20: High Maintainability 
* 10 <= MI < 20: Moderate Maintainability
* MI < 10:  Low Maintainability

### Results

The Maintainability Index is the only metric the *detect* analysis tool does not support. In fact, I found no working tool for Kotlin which calculates the Maintainability Index or even just the Halstead’s Volume. 

In the following, my solution for manually calculating the maintainability index for the original and the refactored `GlidedRose` class is presented step by step.

#### Halstead Volume

The Halstead metric makes use of the assumption that executable programme parts are made up of operators and operands. Defining what the operators and operands to be considered are is one of the tasks before using a Halstead metric. Typically, variables and constants, for example, are considered operands; keywords, logical and comparison operators, etc. are considered operators. [i](https://de.wikipedia.org/wiki/Halstead-Metrik)

##### Formula [i](https://en.wikipedia.org/wiki/Halstead_complexity_measures)

For a given problem, let:

* n1 = the number of distinct operators
* n2 = the number of distinct operands
* N1 = the total number of operators
* N2 = the total number of operands

From these numbers, several measures can be calculated:

* Program vocabulary: n = n1 + n2
* Program length: N = N1 + N2
* Volume: V = N * log2(n)

##### Results

The following shows my take on calculating the Halstead Volume for the original and refactored `GildedRose` class.

* Original

  ```kotlin
  package com.gildedrose
  
  class GildedRose(var items: Array<Item>) {
  
      fun updateQuality() {
          for (i in items.indices) {
              if (items[i].name != "Aged Brie" && items[i].name != "Backstage passes to a TAFKAL80ETC concert") {
                  if (items[i].quality > 0) {
                      if (items[i].name != "Sulfuras, Hand of Ragnaros") {
                          items[i].quality = items[i].quality - 1
                      }
                  }
              } else {
                  if (items[i].quality < 50) {
                      items[i].quality = items[i].quality + 1
  
                      if (items[i].name == "Backstage passes to a TAFKAL80ETC concert") {
                          if (items[i].sellIn < 11) {
                              if (items[i].quality < 50) {
                                  items[i].quality = items[i].quality + 1
                              }
                          }
  
                          if (items[i].sellIn < 6) {
                              if (items[i].quality < 50) {
                                  items[i].quality = items[i].quality + 1
                              }
                          }
                      }
                  }
              }
  
              if (items[i].name != "Sulfuras, Hand of Ragnaros") {
                  items[i].sellIn = items[i].sellIn - 1
              }
  
              if (items[i].sellIn < 0) {
                  if (items[i].name != "Aged Brie") {
                      if (items[i].name != "Backstage passes to a TAFKAL80ETC concert") {
                          if (items[i].quality > 0) {
                              if (items[i].name != "Sulfuras, Hand of Ragnaros") {
                                  items[i].quality = items[i].quality - 1
                              }
                          }
                      } else {
                          items[i].quality = items[i].quality - items[i].quality
                      }
                  } else {
                      if (items[i].quality < 50) {
                          items[i].quality = items[i].quality + 1
                      }
                  }
              }
          }
      }
  }
  ```

  | Operators     | Occurrences | Operands                                    | Occurrences |
  | ------------- | ----------- | ------------------------------------------- | ----------- |
  | class         | 1           | items                                       | 36          |
  | GildedRose    | 1           | i                                           | 35          |
  | ()            | 19          | indices                                     | 1           |
  | Array         | 1           | name                                        | 8           |
  | <>            | 1           | quality                                     | 21          |
  | Item          | 1           | name                                        | 8           |
  | {}            | 22          | 1                                           | 9           |
  | fun           | 1           | 0                                           | 3           |
  | updateQuality | 1           | 50                                          | 4           |
  | for           | 1           | 11                                          | 1           |
  | in            | 1           | 6                                           | 1           |
  | if            | 16          | "Aged Brie"                                 | 2           |
  | else          | 3           | "Backstage passes to a TAFKAL80ETC concert" | 3           |
  | !=            | 8           | "Sulfuras, Hand of Ragnaros"                | 3           |
  | &&            | 1           |                                             |             |
  | >             | 2           |                                             |             |
  | <             | 7           |                                             |             |
  | []            | 34          |                                             |             |
  | +             | 4           |                                             |             |
  | -             | 4           |                                             |             |
  | .             | 35          |                                             |             |
  | ==            | 1           |                                             |             |
  | =             | 8           |                                             |             |
  | :             | 1           |                                             |             |
  | n1=24         | N1=174      | n2=14                                       | N2=135      |

  Program vocabulary: n = n1 + n2 = 38

  Program length: N = N1 + N2 = 309

  Volume: V = N * log2(n) = 309 * 5.25 = 1622.25

  

* Refactored

  ```kotlin
  package com.gildedrose
  
  import com.gildedrose.updater.ItemUpdater
  import com.gildedrose.updater.SimpleItemUpdaterFactory
  
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

  | Operators      | Occurrences | Operands    | Occurrences |
  | -------------- | ----------- | ----------- | ----------- |
  | class          | 1           | items       | 2           |
  | GildedRose     | 1           | MAX_QUALITY | 1           |
  | ()             | 10          | MIN_QUALITY | 1           |
  | Array          | 1           | item        | 8           |
  | <>             | 1           | factory     | 2           |
  | Item           | 3           | 50          | 1           |
  | {}             | 6           | 0           | 1           |
  | private        | 3           |             |             |
  | companion      | 1           |             |             |
  | object         | 1           |             |             |
  | Constants      | 1           |             |             |
  | fun            | 3           |             |             |
  | updateQuality  | 2           |             |             |
  | forEach        | 1           |             |             |
  | update         | 2           |             |             |
  | getItemUpdater | 2           |             |             |
  | updateSellIn   | 1           |             |             |
  | ItemUpdater    | 1           |             |             |
  | return         | 1           |             |             |
  | create         | 1           |             |             |
  | .              | 4           |             |             |
  | :              | 4           |             |             |
  | =              | 3           |             |             |
  | n1=23          | N1=54       | n2=7        | N2=16       |

  Program vocabulary: n = n1 + n2 = 30

  Program length: N = N1 + N2 = 70

  Volume: V = N * log2(n) = 70 * 4.91 = 343.7

#### Cyclomatic Complexity

* Original
  * Method `updateQuality`: 19
  * Total Cyclomatic Complexity: 19
* Refactored
  * Method `updateQuality`: 2
  * Method `update`: 1
  * Method `getItemUpdater`: 1
  * Total Cyclomatic Complexity: 4

#### Lines Of Code

* Original
  * LOC: 56
* Refactored
  * LOC: 29

#### Maintainability Index

Calculated using the Visual Studio formula:

* Original
  * Halstead Volume:  1622.25
  * Cyclomatic Complexity: 19
  * LOC: 56
  * Maintainability Index = MAX(0,(171 - 5.2 * ln(1622.25) - 0.23 * 19 - 16.2 * ln(56))*100 / 171) = 36.83
* Refactored
  * Halstead Volume:  343.7
  * Cyclomatic Complexity: 4
  * LOC: 29
  * Maintainability Index = MAX(0,(171 - 5.2 * ln(343.7) - 0.23 * 4 - 16.2 * ln(29))*100 / 171) = 49.80

### Interpretation





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
java -jar ./detekt/detekt/detekt-cli/build/libs/detekt-cli-1.22.0-RC1-all.jar --plugins ./detekt/detekt-hint/build/libs/detekt-hint.jar --config ./config/detekt-hint.yml --classpath ./original/Kotlin/src/main --input ./original/Kotlin/src/main --report txt:reports/original.txt --report html:reports/original.html --report pdf:reports/original.pdf --report md:reports/original.md
```

Refactored

```bash
java -jar ./detekt/detekt/detekt-cli/build/libs/detekt-cli-1.22.0-RC1-all.jar --plugins ./detekt/detekt-hint/build/libs/detekt-hint.jar --config ./config/detekt-hint.yml --classpath ./refactored/Kotlin/src/main --input ./refactored/Kotlin/src/main --report txt:reports/refactored.txt --report html:reports/refactored.html --report pdf:reports/refactored.pdf --report md:reports/refactored.md
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



