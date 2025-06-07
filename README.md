# Игра Змейка

## Обзор
Мы реализовали простую версию классической игры "Змейка". Игра предназначена для запуска в терминале и написана на языке C++. Игрок управляет змейкой, которая увеличивается в длину, поедая еду. Цель игры — избегать столкновений со стенами и собственным телом змейки.

## Структура проекта
Наш проект состоит из следующих файлов и папок:

- **.github/workflows/**: Содержит файлы конфигурации GitHub Actions для сборки проекта на разных платформах.
  - **build-linux.yml**: Конфигурация для сборки проекта в среде Linux с использованием Clang.
  - **build-windows.yml**: Конфигурация для сборки проекта в среде Windows.

- **src/**: Содержит исходный код игры.
  - **main.cpp**: Точка входа в приложение, инициализация игры и запуск игрового цикла.
  - **game.cpp**: Реализация игровой логики, включая обновление состояния и рендеринг.
  - **game.h**: Заголовочный файл, содержащий объявления классов и функций, используемых в game.cpp.

- **CMakeLists.txt**: Конфигурационный файл для CMake, задающий имя проекта, требуемый стандарт C++ и исходные файлы.

- **README.md**: Документация проекта, включая инструкции по сборке и запуску.

- **LICENSE**: Лицензионная информация о проекте.

## Инструкции по сборке

### Linux
1. Убедитесь, что на вашей системе установлены Clang и CMake.
2. Клонируйте репозиторий на локальный компьютер.
3. Перейдите в директорию проекта.
4. Выполните следующие команды для сборки проекта:
   ```bash
   mkdir build
   cd build
   cmake ..
   make
   ```

### Windows
1. Убедитесь, что у вас установлен совместимый компилятор C++ и CMake.
2. Клонируйте репозиторий на локальный компьютер.
3. Перейдите в директорию проекта.
4. Выполните следующие команды для сборки проекта:
   ```cmd
   mkdir build
   cd build
   cmake ..
   cmake --build . --config Release
   ```

## Запуск игры
После сборки проекта вы можете запустить игру, выполнив скомпилированный бинарный файл:
- На Linux:
  ```bash
  ./snake-game
  ```
- На Windows:
  ```cmd
  snake-game.exe
  ```

###Подробное описание работы build.yml , который создан в .github/workflows

##Job для Linux (build-linux)

##1. Подготовка окружения

```
- uses: actions/checkout@v4
```

##2. Установка зависимостей

```
- name: Install dependencies
  run: |
    sudo apt-get update
    sudo apt-get install -y clang cmake libsfml-dev
```

Обновляет пакетный менеджер

Устанавливает:

Компилятор Clang

Систему сборки CMake

Библиотеку SFML (графика, аудио, ввод)

##3. Сборка проекта

```
- name: Build with CMake
  run: |
    mkdir build
    cd build
    cmake -DCMAKE_CXX_COMPILER=clang++ ..
    make
```

Создает директорию для сборки

Настраивает проект через CMake с указанием компилятора

Компилирует проект с помощью make

##4. Создание DEB-пакета

```
- name: Create DEB package
  run: |
    mkdir -p snake-game/usr/games
    mkdir -p snake-game/usr/share/snake-game/images
    cp build/snake snake-game/usr/games/
    cp -r images/* snake-game/usr/share/snake-game/images/
    mkdir -p snake-game/DEBIAN
    echo "Package: snake-game" > snake-game/DEBIAN/control
    echo "Version: 1.0" >> snake-game/DEBIAN/control
    echo "Section: games" >> snake-game/DEBIAN/control
    echo "Priority: optional" >> snake-game/DEBIAN/control
    echo "Architecture: amd64" >> snake-game/DEBIAN/control
    echo "Maintainer: Kirill207 <kirya.sherstyuk.05@mail.ru>" >> snake-game/DEBIAN/control
    echo "Description: Simple Snake Game" >> snake-game/DEBIAN/control
    dpkg-deb --build snake-game
    mv snake-game.deb snake-game_1.0_amd64.deb
```

Создает структуру пакета:

/usr/games/ для исполняемого файла

/usr/share/snake-game/images/ для ресурсов

Генерирует контрольный файл с метаданными пакета

Собирает DEB-пакет с помощью dpkg-deb

Переименовывает пакет с указанием версии и архитектуры

##5. Загрузка артефакта
```
- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: snake-game-deb
    path: snake-game_1.0_amd64.deb
```
Сохраняет DEB-пакет как артефакт сборки

###Job для Windows (build-windows)

##1. Подготовка окружения

```
- uses: actions/checkout@v4
```

Проверяет исходный код из репозитория

##2. Установка зависимостей

```
- name: Install dependencies
  run: |
    choco install cmake --installargs 'ADD_CMAKE_TO_PATH=System' -y
    choco install wixtoolset -y
```

Устанавливает через Chocolatey:

CMake (добавляя в PATH)

WiX Toolset (для создания MSI-установщиков)

##3. Загрузка SFML

```
- name: Download and extract SFML
```

Создает директорию C:/tools при необходимости

Скачивает SFML 2.6.1 для Windows (64-bit)

Распаковывает архив в C:/tools/SFML

Добавляет путь к бинарникам SFML в PATH

##4. Проверка установки SFML

```
- name: Verify SFML installation
```

Проверяет наличие обязательных директорий:

bin (исполняемые файлы и DLL)

lib (библиотеки)

include (заголовочные файлы)

##5. Сборка проекта

```
- name: Build with CMake
  run: |
    mkdir build
    cd build
    cmake -DSFML_DIR="C:/tools/SFML/lib/cmake/SFML" ..
    cmake --build . --config Release
```

Создает директорию для сборки

Настраивает проект с указанием пути к SFML

Компилирует в конфигурации Release

##6. Подготовка файлов для упаковки

```
- name: Prepare package files
```

Копирует:

Исполняемый файл snake.exe

Необходимые DLL из SFML:

sfml-graphics-2.dll

sfml-window-2.dll

sfml-system-2.dll

sfml-audio-2.dll

openal32.dll

Графические ресурсы из папки `images`

##7. Создание MSI-пакета

```
- name: Create MSI package
```

Генерирует XML-файл (product.wxs) с описанием установщика:

Метаданныe продукта

Структура директорий

Компоненты (исполняемый файл, DLL, ресурсы)

Компилирует установщик с помощью WiX:

`candle.exe` - компиляция WXS-файла

`light.exe` - линковка с созданием MSI

##8. Загрузка артефакта

```
- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: snake-game-msi
    path: snake-game.msi
```

Сохраняет MSI-установщик как артефакт сборки
