# Maven — Build, Dependencies, and Test Orchestration for the JVM

> Subject #14 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: file 06 (Java). Estimated time: ~25 hours.
> Target outcome: Run, debug, and extend Maven builds in real projects — multi-module layouts, dependency hygiene, test plugin configuration (Surefire, Failsafe, JaCoCo), CI wiring, and the ability to answer "why does Maven do X?" in interviews without guessing.

---

## 1. Overview & Learning Outcomes

Maven is the de-facto build tool for enterprise Java and for most QA automation frameworks you'll join (REST-assured, Selenium, Cucumber, Playwright-Java, Spring Boot services). You don't need to love it — you need to *read a POM fluently*, diagnose dependency problems fast, and extend the build without cargo-culting.

By the end of this subject you should be able to:

- Read any `pom.xml` top-to-bottom and explain every tag.
- Resolve dependency conflicts with `mvn dependency:tree` and `<dependencyManagement>`.
- Split unit vs integration tests across Surefire and Failsafe, wire code-coverage with JaCoCo, and add mutation testing with PIT.
- Design a multi-module project (parent POM, shared BOM, child modules) and know when *not* to.
- Use profiles and `settings.xml` to vary behavior per environment without polluting the POM.
- Publish artifacts to a private Nexus/Artifactory and pull them from CI.
- Compare Maven vs Gradle honestly.

---

## 2. Detailed Syllabus

### Part A — Foundations (6 h)

1. What Maven is, the "convention over configuration" pitch — 0.5h
2. Installing Maven, the Maven Wrapper (`mvnw`) — 0.5h
3. Directory layout, `pom.xml` anatomy — 1h
4. Coordinates (groupId / artifactId / version / packaging / classifier) — 1h
5. Local repo (`~/.m2`), Maven Central, private repos — 1h
6. The build lifecycle, phases, and goals — 2h

### Part B — Dependencies (5 h)

7. `<dependencies>` vs `<dependencyManagement>` — 1h
8. Scopes: compile / provided / runtime / test / system / import — 1h
9. Transitive dependencies, exclusions, conflict resolution — 1h
10. BOMs (Bill of Materials), Spring Boot BOM, JUnit BOM — 1h
11. SNAPSHOT vs RELEASE, version ranges (and why to avoid them) — 1h

### Part C — Plugins & Testing (6 h)

12. Plugins, goals, executions, `<configuration>` — 1h
13. Compiler plugin, release flag, toolchains — 0.5h
14. Surefire (unit tests) vs Failsafe (integration tests) — 1.5h
15. JaCoCo code coverage (report + thresholds) — 1h
16. PIT mutation testing — 0.5h
17. Spotless / Checkstyle / SpotBugs for quality gates — 0.5h
18. Maven Enforcer for build rules — 0.5h
19. Maven Shade / Assembly for fat JARs — 0.5h

### Part D — Structure & Environments (4 h)

20. Multi-module projects, parent POMs, aggregator vs parent — 1.5h
21. Profiles (env, OS, JDK, file presence) — 1h
22. `settings.xml` — servers, mirrors, proxies — 1h
23. Properties, resource filtering — 0.5h

### Part E — CI & Release (4 h)

24. CI-friendly versions, `-q -B -ntp` flags — 0.5h
25. GitHub Actions with Maven cache — 1h
26. Publishing to Nexus/Artifactory, `deploy` phase — 1h
27. Maven Release Plugin vs Maven-CI-Friendly approach — 1h
28. Maven vs Gradle — honest comparison — 0.5h

---

## 3. Topics — Explanations with Examples

### 3.1 Coordinates and the POM

**Concept.** Every Maven artifact is uniquely identified by `groupId:artifactId:version` (GAV). Optional `packaging` (default `jar`; also `war`, `pom`, `maven-plugin`) and `classifier` (e.g., `sources`, `javadoc`, `tests`) extend that. The POM (Project Object Model) is the declarative build file.

**Minimal POM.**
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.govind</groupId>
  <artifactId>qa-toolkit</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <properties>
    <maven.compiler.release>21</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
</project>
```

**Standard layout** (don't fight it):
```
src/main/java          # production code
src/main/resources     # classpath resources
src/test/java          # tests
src/test/resources     # test resources
target/                # build output
```

### 3.2 The Maven Wrapper

**Concept.** `mvnw` / `mvnw.cmd` pin a specific Maven version per project — anyone cloning the repo gets the same build, no global install needed. Prefer it over system Maven.

```bash
# Add wrapper to a project
mvn -N wrapper:wrapper -Dmaven=3.9.9

./mvnw -v          # prints pinned version
./mvnw clean verify
```

### 3.3 Lifecycle, phases, goals

**Concept.** A **lifecycle** is an ordered list of **phases**. Running a phase runs all earlier phases too. **Goals** (provided by plugins) are bound to phases.

Three built-in lifecycles: `clean`, `default`, `site`. The default lifecycle's key phases:

```
validate → compile → test → package → verify → install → deploy
```

- `test` — runs **unit tests** via Surefire; fails the build if any fail.
- `package` — produces the JAR/WAR.
- `verify` — runs **integration tests** via Failsafe.
- `install` — copies the artifact to `~/.m2/repository`.
- `deploy` — uploads to a remote repo (needs `<distributionManagement>` + credentials).

```bash
./mvnw clean verify            # typical CI command
./mvnw -pl api -am test        # test one module + its deps
./mvnw -o package              # offline mode
./mvnw -DskipTests verify      # skip all tests (rare, and a smell)
./mvnw -Dtest=OrderTest test   # run a single test class
./mvnw -Dtest='Order*#happy*'  # class + method pattern
```

### 3.4 Dependencies and scope

**Concept.** Maven resolves a transitive graph. Scope limits where a dep is visible.

| Scope      | Compile | Test | Runtime | In final JAR |
|------------|:-------:|:----:|:-------:|:------------:|
| compile    | Y       | Y    | Y       | Y            |
| provided   | Y       | Y    | N       | N            |
| runtime    | N       | Y    | Y       | Y            |
| test       | N       | Y    | N       | N            |
| system     | Y       | Y    | N       | N            |
| import     | (BOM only, in dependencyManagement)                |

**Example — typical mix for a Spring Boot service.**
```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <dependency>            <!-- ships in Tomcat; not in fat jar for WAR deploys -->
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <scope>provided</scope>
  </dependency>

  <dependency>            <!-- only on runtime classpath -->
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
  </dependency>

  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

### 3.5 Dependency management and BOMs

**Concept.** `<dependencyManagement>` lets a parent POM declare *versions* without forcing *inclusion*. Child POMs then declare the dep without a version.

A **BOM** (Bill of Materials) is a POM with packaging `pom` used via `scope=import` to pull in a *curated matrix* of compatible versions — the killer feature for Spring, JUnit, Jackson, Testcontainers, etc.

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>3.3.4</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
    <dependency>
      <groupId>org.junit</groupId>
      <artifactId>junit-bom</artifactId>
      <version>5.11.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

**Interview tip.** The right answer to "how do you manage dep versions across 20 modules?" is: a parent POM with `<dependencyManagement>` + imported BOMs — not version pinning in every child.

### 3.6 Conflict resolution

**Concept.** When two paths bring in different versions, Maven picks the **nearest** one in the tree (shortest path from root), breaking ties by **order in the POM**. This is called *nearest-wins*, not *highest-wins* (that's Gradle).

```bash
./mvnw dependency:tree -Dverbose
./mvnw dependency:tree -Dincludes=com.fasterxml.jackson.core
./mvnw dependency:analyze          # unused declared, used undeclared
```

**Fix patterns.**

- Pin the version in `<dependencyManagement>`.
- `<exclusions>` on the transitive caller.
- Use `maven-enforcer-plugin` to fail the build on convergence violations.

```xml
<dependency>
  <groupId>org.example</groupId>
  <artifactId>leaky</artifactId>
  <exclusions>
    <exclusion>
      <groupId>log4j</groupId><artifactId>log4j</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

### 3.7 SNAPSHOT vs RELEASE

**Concept.** `-SNAPSHOT` versions are mutable — Maven checks the remote repo for updates on each build (default: daily; override with `-U`). Release versions are immutable — once published, never rewritten.

Rules of thumb: SNAPSHOTs during development; promote to a release version (e.g., `1.4.0`) for every deployable build. Never release a dependency on a SNAPSHOT.

### 3.8 Surefire vs Failsafe — unit vs integration

**Concept.** Surefire runs in the `test` phase, Failsafe in `integration-test` + `verify`. They're separate so you can run fast unit tests independently of slow integration tests.

**Naming conventions** (default):

- Surefire picks up `*Test.java`, `*Tests.java`, `Test*.java`.
- Failsafe picks up `*IT.java`, `IT*.java`, `*ITCase.java`.

**Example — full test config with JUnit 5.**
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>3.5.0</version>
      <configuration>
        <parallel>classes</parallel>
        <threadCount>4</threadCount>
        <forkCount>1C</forkCount>            <!-- 1 fork per CPU -->
        <reuseForks>true</reuseForks>
        <trimStackTrace>false</trimStackTrace>
      </configuration>
    </plugin>

    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-failsafe-plugin</artifactId>
      <version>3.5.0</version>
      <executions>
        <execution>
          <goals>
            <goal>integration-test</goal>
            <goal>verify</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

**Critical nuance.** Failsafe's `verify` goal is what fails the build on integration-test failure. If you only bind `integration-test`, failing ITs will still let `mvn install` succeed.

### 3.9 JaCoCo code coverage

```xml
<plugin>
  <groupId>org.jacoco</groupId>
  <artifactId>jacoco-maven-plugin</artifactId>
  <version>0.8.12</version>
  <executions>
    <execution>
      <goals><goal>prepare-agent</goal></goals>
    </execution>
    <execution>
      <id>report</id>
      <phase>verify</phase>
      <goals><goal>report</goal></goals>
    </execution>
    <execution>
      <id>check</id>
      <goals><goal>check</goal></goals>
      <configuration>
        <rules>
          <rule>
            <element>BUNDLE</element>
            <limits>
              <limit>
                <counter>LINE</counter>
                <value>COVEREDRATIO</value>
                <minimum>0.80</minimum>
              </limit>
            </limits>
          </rule>
        </rules>
      </configuration>
    </execution>
  </executions>
</plugin>
```

Open `target/site/jacoco/index.html` after `mvn verify`.

### 3.10 PIT mutation testing

**Concept.** JaCoCo tells you which lines ran; PIT tells you whether your tests would *notice* if the code were wrong. It mutates bytecode (flip `>` to `>=`, remove conditionals) and re-runs tests.

```xml
<plugin>
  <groupId>org.pitest</groupId>
  <artifactId>pitest-maven</artifactId>
  <version>1.17.0</version>
  <dependencies>
    <dependency>
      <groupId>org.pitest</groupId>
      <artifactId>pitest-junit5-plugin</artifactId>
      <version>1.2.1</version>
    </dependency>
  </dependencies>
  <configuration>
    <targetClasses><param>com.govind.*</param></targetClasses>
    <mutationThreshold>70</mutationThreshold>
  </configuration>
</plugin>
```

Run: `./mvnw org.pitest:pitest-maven:mutationCoverage`.

### 3.11 Quality gates — Spotless, Checkstyle, Enforcer

```xml
<plugin>
  <groupId>com.diffplug.spotless</groupId>
  <artifactId>spotless-maven-plugin</artifactId>
  <version>2.43.0</version>
  <configuration>
    <java>
      <googleJavaFormat><version>1.23.0</version></googleJavaFormat>
      <removeUnusedImports/>
    </java>
  </configuration>
  <executions>
    <execution>
      <goals><goal>check</goal></goals>
    </execution>
  </executions>
</plugin>

<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-enforcer-plugin</artifactId>
  <version>3.5.0</version>
  <executions>
    <execution>
      <id>enforce</id>
      <goals><goal>enforce</goal></goals>
      <configuration>
        <rules>
          <requireMavenVersion><version>[3.9.0,)</version></requireMavenVersion>
          <requireJavaVersion><version>[21,)</version></requireJavaVersion>
          <dependencyConvergence/>
          <banDuplicatePomDependencyVersions/>
        </rules>
      </configuration>
    </execution>
  </executions>
</plugin>
```

### 3.12 Multi-module layout

**Concept.** One parent aggregates many child modules. The parent is `packaging=pom` and lists `<modules>`; children inherit via `<parent>`.

```
library/
├── pom.xml                     (parent)
├── domain/pom.xml              (shared types)
├── api/pom.xml                 (depends on domain)
├── worker/pom.xml              (depends on domain)
└── integration-tests/pom.xml   (depends on api + worker)
```

**Parent POM.**
```xml
<project>
  <groupId>com.govind</groupId>
  <artifactId>library-parent</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <modules>
    <module>domain</module>
    <module>api</module>
    <module>worker</module>
    <module>integration-tests</module>
  </modules>

  <dependencyManagement> … </dependencyManagement>
  <build><pluginManagement> … </pluginManagement></build>
</project>
```

**Child POM.**
```xml
<project>
  <parent>
    <groupId>com.govind</groupId>
    <artifactId>library-parent</artifactId>
    <version>0.1.0-SNAPSHOT</version>
  </parent>
  <artifactId>api</artifactId>
  <dependencies>
    <dependency>
      <groupId>com.govind</groupId>
      <artifactId>domain</artifactId>
      <version>${project.version}</version>
    </dependency>
  </dependencies>
</project>
```

**Reactor tricks.**
```bash
./mvnw -pl api -am verify          # build api + required deps (also-make)
./mvnw -pl api -amd verify         # build api + everything that depends on it
./mvnw -rf integration-tests verify # resume-from after a failure
```

### 3.13 Profiles

**Concept.** Profiles let you vary config per environment / OS / JDK / property / file without duplicating POMs.

```xml
<profiles>
  <profile>
    <id>integration</id>
    <build>
      <plugins>
        <plugin>
          <artifactId>maven-failsafe-plugin</artifactId>
          <configuration>
            <groups>integration</groups>
          </configuration>
        </plugin>
      </plugins>
    </build>
  </profile>

  <profile>
    <id>release</id>
    <activation>
      <property><name>performRelease</name></property>
    </activation>
    <build>
      <plugins>…sign, javadoc, sources…</plugins>
    </build>
  </profile>
</profiles>
```

```bash
./mvnw -Pintegration verify
./mvnw -P'!release' install      # disable a profile
```

### 3.14 settings.xml

**Concept.** Not a POM file — user-level (`~/.m2/settings.xml`) or global (`$MAVEN_HOME/conf/settings.xml`). Holds credentials, mirrors, proxies, and *active* profiles. **Never commit credentials to a POM.**

```xml
<settings>
  <servers>
    <server>
      <id>nexus-releases</id>
      <username>${env.NEXUS_USER}</username>
      <password>${env.NEXUS_PASS}</password>
    </server>
  </servers>
  <mirrors>
    <mirror>
      <id>company-central</id>
      <mirrorOf>*</mirrorOf>
      <url>https://nexus.company.com/repository/maven-public/</url>
    </mirror>
  </mirrors>
</settings>
```

### 3.15 Resource filtering and properties

Swap `${env}` tokens in resource files at build time:

```xml
<build>
  <resources>
    <resource>
      <directory>src/main/resources</directory>
      <filtering>true</filtering>
    </resource>
  </resources>
</build>
```

```properties
# application.properties
app.build.version=${project.version}
app.env=${env}
```

Invoke: `./mvnw -Denv=prod package`.

### 3.16 Shade / Assembly — building fat JARs

**Concept.** `maven-shade-plugin` repackages dependencies into one JAR (optionally relocating packages to dodge conflicts). Spring Boot has its own `spring-boot-maven-plugin` that does the same with a layered format.

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <version>3.6.0</version>
  <executions>
    <execution>
      <phase>package</phase>
      <goals><goal>shade</goal></goals>
      <configuration>
        <transformers>
          <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <mainClass>com.govind.Main</mainClass>
          </transformer>
        </transformers>
      </configuration>
    </execution>
  </executions>
</plugin>
```

### 3.17 Deploying artifacts

```xml
<distributionManagement>
  <repository>
    <id>nexus-releases</id>
    <url>https://nexus.company.com/repository/maven-releases/</url>
  </repository>
  <snapshotRepository>
    <id>nexus-snapshots</id>
    <url>https://nexus.company.com/repository/maven-snapshots/</url>
  </snapshotRepository>
</distributionManagement>
```

`./mvnw deploy -s settings.xml`.

### 3.18 Releasing — old way vs CI-friendly way

**Old way (Maven Release Plugin).** `mvn release:prepare release:perform` — edits POMs, tags, deploys. Fine for solo or small teams.

**Modern way (CI-friendly versions).** Use `${revision}` in `<version>` and set on the CLI:

```xml
<version>${revision}</version>
<properties><revision>0.1.0-SNAPSHOT</revision></properties>
```

```bash
./mvnw -Drevision=1.4.0 deploy
```

Pairs well with GitHub Actions that sets the version from a tag.

### 3.19 CI recipe — GitHub Actions

```yaml
name: build
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: '21', cache: maven }
      - run: ./mvnw -B -ntp verify
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: surefire-reports
          path: '**/target/*-reports'
```

- `-B` / `--batch-mode` — non-interactive.
- `-ntp` / `--no-transfer-progress` — hide download noise.
- `cache: maven` — GitHub's native `~/.m2` cache.

### 3.20 Maven vs Gradle

| Dimension       | Maven                               | Gradle                           |
|-----------------|--------------------------------------|----------------------------------|
| Config          | XML, declarative                    | Groovy/Kotlin DSL, imperative    |
| Learning curve  | Gentle, predictable                 | Steep, very flexible             |
| Speed           | Fine; incremental is weak           | Fast — incremental + build cache |
| Conflict rule   | Nearest-wins                        | Highest-wins                     |
| Ecosystem       | Enormous, stable                    | Large, Android default           |
| Customization   | Via plugins                         | Via any code                     |

Prefer Maven when you want uniformity across 50 services. Prefer Gradle when build time hurts or you want DSL-level customization.

---

## 4. Exercises (Hands-On)

### Beginner

- **A1.** Init a project with `mvn archetype:generate -DgroupId=com.govind -DartifactId=hello`; build it with the Wrapper.
  - *Acceptance:* `./mvnw -v` shows pinned version; `./mvnw verify` passes.
- **A2.** Add JUnit 5 + AssertJ; write `OrderTest`; run a single test by name.
- **A3.** Add JaCoCo; fail the build below 80% line coverage.

### Intermediate

- **B1.** Create a parent POM with three modules: `domain`, `api`, `integration-tests`. Share versions via a BOM-style `<dependencyManagement>`.
- **B2.** Split unit (`*Test`) and integration (`*IT`) tests; bind Failsafe to `verify`; run only ITs via `-Pintegration`.
- **B3.** Add Spotless (Google Java Format) and fail the build on style violations. Add Enforcer rules: `dependencyConvergence` + required Java version.
- **B4.** Resolve a fake conflict: introduce two Jackson versions via transitive deps; fix with `<dependencyManagement>`; prove with `dependency:tree`.

### Advanced

- **C1.** Add PIT mutation testing; set a 70% threshold; iterate on tests until it passes.
- **C2.** Configure CI-friendly versions via `${revision}`; build `1.0.0-SNAPSHOT`, `1.0.0` and `1.0.1` from the same POM.
- **C3.** Stand up a local Nexus (Docker); deploy snapshots and releases; consume from a separate consumer project via `settings.xml`.
- **C4.** Convert the project to Spring Boot: use the Spring Boot BOM, the `spring-boot-maven-plugin`, and a multi-stage Dockerfile that uses the layered JAR.

### Capstone — "Library API Build Pipeline"

Take the Library API capstone from file 06-Java.md and professionalize its build:

1. Multi-module layout (`domain`, `api`, `worker`, `integration-tests`).
2. Shared parent POM with BOM imports (Spring Boot, JUnit, Testcontainers).
3. Surefire + Failsafe split, JaCoCo 80% gate, PIT 70% mutation gate, Spotless clean.
4. GitHub Actions build + cache + artifact upload.
5. Release with CI-friendly `${revision}` driven from a git tag.
6. One-line README explaining every non-default plugin.

---

## 5. Additional Reading & Resources

### Books
- *Maven: The Complete Reference* — Sonatype (free online).
- *Maven by Example* — Sonatype (gentle intro, free online).
- *Mastering Apache Maven* — Packt.

### Official docs
- Apache Maven — <https://maven.apache.org/>
- Maven Central Search — <https://central.sonatype.com/>
- Surefire — <https://maven.apache.org/surefire/maven-surefire-plugin/>
- Failsafe — <https://maven.apache.org/surefire/maven-failsafe-plugin/>
- JaCoCo — <https://www.jacoco.org/jacoco/trunk/doc/>
- Maven Enforcer — <https://maven.apache.org/enforcer/maven-enforcer-plugin/>

### Courses
- Baeldung's Maven guide series — <https://www.baeldung.com/maven>.
- Tim Buchalka's *Maven Crash Course* (Udemy).
- JetBrains Academy — Maven track.

### YouTube / talks
- "Maven in 5 minutes" (Apache) and the official "Why Maven?" talks.
- "Modern Maven" — Devoxx talks (search by year).

### Practice
- Spring Initializr (<https://start.spring.io/>) — always generates a clean, idiomatic POM to study.
- *Real-world* POMs: Spring Boot, JUnit 5, Testcontainers, Playwright-Java.

---

## 6. Index

- `-am` / `-amd` (also-make) — §3.12
- archetype — exercise A1
- BOM — §3.5
- CI-friendly versions — §3.18
- conflict resolution — §3.6
- coordinates (GAV) — §3.1
- `dependencyManagement` — §3.5
- `dependency:tree` — §3.6
- deploy phase — §3.17
- Enforcer — §3.11
- Failsafe — §3.8
- fat JAR (Shade) — §3.16
- GitHub Actions cache — §3.19
- JaCoCo — §3.9
- lifecycle / phases — §3.3
- Maven Wrapper — §3.2
- multi-module — §3.12
- nearest-wins rule — §3.6
- parent POM — §3.12
- PIT — §3.10
- profiles — §3.13
- resource filtering — §3.15
- scopes — §3.4
- settings.xml — §3.14
- SNAPSHOT vs RELEASE — §3.7
- Spotless — §3.11
- Surefire — §3.8
- `verify` phase — §3.3, §3.8

---

## 7. References

1. Apache Maven Project — <https://maven.apache.org/>
2. Introduction to the Build Lifecycle — <https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html>
3. Introduction to Dependency Mechanism — <https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html>
4. Surefire Plugin — <https://maven.apache.org/surefire/maven-surefire-plugin/>
5. Failsafe Plugin — <https://maven.apache.org/surefire/maven-failsafe-plugin/>
6. JaCoCo Maven Plugin — <https://www.jacoco.org/jacoco/trunk/doc/maven.html>
7. PIT Mutation Testing — <https://pitest.org/>
8. Spotless Maven Plugin — <https://github.com/diffplug/spotless/tree/main/plugin-maven>
9. Maven Enforcer Plugin — <https://maven.apache.org/enforcer/maven-enforcer-plugin/>
10. Maven Shade Plugin — <https://maven.apache.org/plugins/maven-shade-plugin/>
11. Spring Boot Maven Plugin — <https://docs.spring.io/spring-boot/maven-plugin/>
12. CI-friendly versions — <https://maven.apache.org/maven-ci-friendly.html>
13. *Maven: The Complete Reference* — Sonatype — <https://books.sonatype.com/mvnref-book/>
14. Baeldung — Maven category — <https://www.baeldung.com/maven>

---

*End of Subject 14 — Maven.*
