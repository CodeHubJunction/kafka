## Gradle Multi-Project Setup with Eclipse

### Goal

Create a clean Gradle multi-project structure with:

- Root project: `kafka-beginners-course`
- Subproject/module: `kafka-basic`
- Java + Kafka dependencies
- Eclipse-compatible project structure

---

### 📁 Final Folder Structure

```
kafka-beginners-course/
├── build.gradle.kts
├── gradle.properties
├── settings.gradle.kts
├── gradle/
├── gradlew
├── gradlew.bat
└── kafka-basic/
    ├── build.gradle.kts
    └── src/
        ├── main/java/
        └── test/java/
```

---

### 🔧 Step-by-Step Setup

#### 1. Create root project folder

The folder kafkaprojects is for eclipse workspace.

```bash
mkdir kafkaprojects
mkdir kafka-beginners-course
cd kafka-beginners-course
gradle init
```

Choose:

- Project type: basic or application
- Project name (default: kafka-beginners-course): ↵
- DSL: Kotlin
- Generate build using new APIs yes
- Generate wrapper: Yes

#### 2. Create subproject

```bash
mkdir -p kafka-basic/src/main/java
mkdir -p kafka-basic/src/test/java
touch kafka-basic/build.gradle.kts
```

#### 3. Update `settings.gradle.kts` in the root project

```kotlin
rootProject.name = "kafka-beginners-course"
include("kafka-basic")
```

#### 4. Add to `kafka-basic/build.gradle.kts`

```kotlin
plugins {
    java
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.apache.kafka:kafka-clients:3.7.0")
    implementation("org.slf4j:slf4j-api:2.0.12")
    runtimeOnly("org.slf4j:slf4j-simple:2.0.12")
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.0")
}

java {
    toolchain {
        languageVersion.set(JavaLanguageVersion.of(17))
    }
}

tasks.test {
    useJUnitPlatform()
}
```

#### 5. Initialize Gradle wrapper (if not already done)

```bash
cd kafka-beginners-course
gradle wrapper
```

---

### 🎮 Build the project

From the root:

```bash
./gradlew build
```

---

### 🚀 Run in Eclipse (Buildship)

1. Right-click the root project in Eclipse
2. Choose **Gradle** → **Refresh Gradle Project**
3. If not imported:
   - File → Import → Gradle → Existing Gradle Project
   - Select `kafka-beginners-course`

---

### ⚡ Fix: Sources not detected in Eclipse?

Make sure:

- Your folder structure is correct (`src/main/java`, etc.)
- `java` plugin is applied in `build.gradle.kts`
- Run:
  ```bash
  ./gradlew clean build
  ```
- Then refresh the project in Eclipse

---
