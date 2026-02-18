unzip -d / -l

# Install
Download Gradle https://gradle.org/releases/
```
mkdir /usr/local
unzip -d /usr/local gradle-8.14-bin.zip
ls /usr/local/gradle-8.14

export PATH=$PATH:/usr/local/gradle-8.14/bin

or write in zsh:
echo 'export PATH=$PATH:/usr/local/gradle-8.14/bin' >> ~/.zshrc

gradle -v
```

# Environment Variables
```
export ORG_GRADLE_PROJECT_myProperty='Hi, world'
```
then Gradle will set a `myProperty` property on your project object, with the value of `Hi, world`.

# Object model

`gradle init` - инициализация проекта

Когда мы запускаем билд создается корневой объект Gradle в котором есть TaskGraph в этом интерфейсе храниться граф Task (на основе билд скриптов, build.gradle строится объектная модель)

Объект Settings описывается settings.gradle и служит для конфигурации иерархии Project объектов (не нужен в случае одномодульного проекта)

Объект Project описывается build.gradle файлом. Если у нас 1 модуль то будет создан 1 объект типа Project (или build.gradle)

Проект состоит из задач Task (init, test, etc.)

Задачи состоят из экшенов Action (минимум 1)

Объект Script описывает groovy скрипты (на основе settings.gradle создается groovy скрипт, который описыает определенную модель Project, Settings и тд)

# Lifecycle

Init Phase
Шаг 1
Считывает init*.gradle скрипты из init.d и создает объект типа Script для инициализации объекта типа Gradle (содержащим в себе структуру ациклического графа)

Шаг 2
Считывает settings.gradle и создает объект типа Script для инициализации объекта типа Settings (если несколько модулей). Описали все проекты где лежат build.gradle файлы

Config Phase
Шаг 3
Считывает build.gradle файлы создает объект типа Script для инициализации объекта типа Project
Строится граф проекта

Exec Phase
Шаг 4
В объекте Project вызываем все его Task 

# gradle wrapper
Запускает проект с той версией гредл с которой он создавался
- gradle-wrapper.jar - код для загрузки гредла
- gradle-wrapper.properties - описывает версию
- gradlew - загружает и запускает нужную версию гредл

# gradle.properties 
описывает свойства проекта
# build.gradle
Состоит из 
- Описания плагинов plugins. Градл может использоваться для разных целей, например собрать проект на скалке, встроенной поддержки скалы в нем нет, поэтому применяются расширения (или плагины)
	- id 'scala' - для сборки проекта на скале
	- id "io.spring.dependency-management" version "1.1.0" - управление зависимостями (гарантирует совместимость версий библиотек между собой)
	- id 'com.github.johnrengelman.shadow' version '7.1.2' - для сборки толстых джарников (uber-jar)
- Указания зависимостей dependencies
	- чтобы исключить зависимость из пакета
```gradle
implementation("org.apache.zookeeper:zookeeper:${versions.zookeeper}") {  
    exclude group: 'junit', module: 'junit'  
}
```
- Репозиторий repositories
```
buildscript {  
    repositories {  
        gradlePluginPortal()  
        mavenCentral()  
    }
```


Указали версии исходного и байт-кодов:
sourceCompatibility = JavaVersion.VERSION_17  
targetCompatibility = JavaVersion.VERSION_17
# Команды
- `./gradlew build` сборка проекта (результат в папке build)
- `./gradlew clean` очистит папку build
- `./gradlew test` запуск тестов
- `./gradlew projects` покажет структуру проекта
- `./gradlew tasks` покажет возможные задачи
- `./gradlew tiOrder assemble` покажет возможные задачи
- `./gradlew publishToMavenLocal` размещает локальную версию

Запуск градл из идеи запустит тот градл, который указан в настройках

# Task
Свои задачи Task можно описать в build/src

```gradle
tasks.register('hello').configure {  
    doFirst {  
        print "Hello "  
    }  
    doLast {  
        print "World"  
    }  
}
```

```
class HelloTask extends DefaultTask {  
    @TaskAction  
    def hello() {  
        print "Hello world"  
    }  
}

tasks.register('hello', HelloTask)
```

Зависимость через in & out
```
Provider<Task1> task1 = tasks.register("task1", Task1) {  
    outputFile.set(project.layout.buildDirectory.file("fff"))  
}  
tasks.register("task2", Task2) {  
    inputFile.set(task1.flatMap {it.outputFile})  
}
```

# Dependency tree
```
gradle dependencies --scan
```

Чтобы были доступны учетные данные создать файл gradle.properties в .gradle
с содержимым
```
ARTIFACTORY_USER=login
ARTIFACTORY_TOKEN=password
ARTIFACTORY_URL=https://artifactory.mts.ru/artifactory/
```


# Errors
- Если не выполняется команда `./gradlew build` с ошибкой permission denied, то нужно сделать файл исполняемым `chmod +x gradlew`