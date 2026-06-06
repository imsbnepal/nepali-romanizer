# Nepali Translator – Java Library

A production-ready, pluggable Java library for:

| Direction | Type | Engine |
|---|---|---|
| Roman Nepali → Nepali | Transliteration | Rule-based (offline) |
| Nepali → Roman Nepali | Transliteration | Rule-based (offline) |


## Installation via GitLab Maven

To use this library in your **Maven** project, add the following to your `pom.xml`:

```xml
<repositories>
     <repository>
            <id>github-mvn</id>
            <name>GitHub SBNepal Apache Maven Packages</name>
            <url>https://maven.pkg.github.com/SBNepal/nepali-romanizer</url>
        </repository>
</repositories>

<dependencies>
    <dependency>
       <groupId>nepaliromanis</groupId>
    <artifactId>nepali-romanis</artifactId>
    <version>1.0.1</version>
    </dependency>
</dependencies>
```


## Quick start

### Transliteration only (no API key required)

```java
NepaliRomanizer romanized = NepaliRomanizer.builder().build();

// Roman → Nepali
String nepali = facade.romanToNepali("namaste");   // → "नमस्ते"
String nepal  = facade.romanToNepali("nepal");      // → "नेपाल"
String city   = facade.romanToNepali("kathmandu");  // → "काठमाडौं" (dictionary hit)

// Nepali → Roman
String roman  = facade.nepaliToRoman("नमस्ते");    // → "namaste"
```


### Full metadata result

```java
TranslationResult result = facade.translateFull("namaste",
        LanguagePair.of(Language.ROMAN_NEPALI, Language.NEPALI));

System.out.println(result.getOutput());          // नमस्ते
System.out.println(result.getProvider());        // RuleBasedRomanToNepali+Cache
System.out.println(result.isCached());           // true/false
System.out.println(result.getProcessingTimeMs()); // 0
```
## Architecture decisions

### Why interfaces + registry?

The `Translator` interface lets you swap engines without changing call sites.
The `TranslatorRegistry` is just a `ConcurrentHashMap<LanguagePair, Translator>`,
so it is thread-safe and O(1) lookup.

### Why two separate concerns (transliteration vs translation)?

Transliteration (Roman ↔ Nepali) is deterministic and rule-based — it needs
no network, no API key, and runs in microseconds.
Translation (Nepali ↔ English) is probabilistic and requires either a
dictionary or an external model.
Mixing them in the same class would force callers to configure API keys even
when they only need script conversion.

### Why Caffeine for caching?

Caffeine is the fastest JVM-native cache (W-TinyLFU eviction policy, O(1)
read, no lock contention under high concurrency). It outperforms Guava Cache
and EhCache for the simple key→value workload that translation caching requires.
The `CachingTranslator` decorator follows the Decorator pattern, keeping caching
logic entirely separate from translation logic.



---
---

## Running tests

```bash
mvn test
```
Tests are self-contained — the transliteration tests require no network or API key.

---

## Building a fat JAR

```bash
mvn package -DskipTests
java -jar target/nepali-translator-1.0.0.jar
```
