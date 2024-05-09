## Maven

Maven uses an XML file to describe the software project being built, its dependencies on other external modules and components, the build order, directories, and required plugins. It comes with pre-defined targets for performing certain well-defined tasks such as compilation of code and its packaging.

Key Features:
- Simple project setup that follows best practices.
- Dependency management including automatic updating, dependency closures (also known as transitive dependencies)
- Able to easily work with multiple projects at the same time.
- Large and mature community with a large ecosystem of plugins and integrations.

``` shell
mvn clean package
```

``` shell
mvn clean install
```

``` shell
mvn clean package spring-boot:run
```

``` shell
mvn versions:display-dependency-updates
```

``` shell
mvn dependency:analyze
```

more about
[Maven dependency plugin](https://maven.apache.org/plugins/maven-dependency-plugin/usage.html){target="_blank"}

## Gradle

Gradle is another build automation tool that builds upon the concepts of Apache Ant and Apache Maven and introduces a Groovy-based domain-specific language (DSL) instead of the XML form used by Apache Maven for declaring the project configuration. Gradle provides a platform to support the entire development lifecycle of a software project.

Key Features:
- Declarative builds and build-by-convention.
- Language for dependency-based programming.
- Structure your build.
- Deep API.
- Multi-project builds.
- Many ways to manage dependencies.
- Integration with existing structures.
- Ease of migration.
