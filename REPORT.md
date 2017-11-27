# Laboratory work X

Данная лабораторная работа посвещена изучению систем управления пакетами на примере **Hunter**

```ShellSession
$ open https://github.com/ruslo/hunter
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab10** на сервисе **GitHub**
- [x] 2. Сгенирировать токен для доступа к сервису **GitHub** с правами **repo**
- [x] 3. Выполнить инструкцию учебного материала
- [x] 4. Ознакомиться со ссылками учебного материала
- [x] 5. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

Задаем переменные окружения : имя пользователя, токен гитхаба
```ShellSession
$ export GITHUB_USERNAME=<имя_пользователя>
$ export GITHUB_TOKEN=<сгенирированный_токен>
```


```ShellSession
$ cd ${GITHUB_USERNAME}/workspace      - переходим в директорию workspace
$ pushd .  - сохраняем текущую директорию в стек
$ source scripts/activate    - указываем источник
$ go get github.com/github/hub   - скачиваем и устанавливаем hub
```

```ShellSession
$ mkdir ~/.config   - созданем папку .config в домашней директории
$ cat > ~/.config/hub <<EOF    - создание файла hub и запись в него значения имя пользователя гитхаба, токена, протокола
github.com:
- user: ${GITHUB_USERNAME}
  oauth_token: ${GITHUB_TOKEN}
  protocol: https
EOF
$ git config --global hub.protocol https - просмотр и установка значения hub.protocol 
```


```ShellSession
$ wget https://github.com/${GITHUB_USERNAME}/lab09/archive/v0.1.0.0.tar.gz   - скачиваем архив
$ export PRINT_SHA1=`openssl sha1 v0.1.0.0.tar.gz | cut -d'=' -f2 | cut -c2-41`    - задаем переменную окружения PRINT_SHA1
$ echo $PRINT_SHA1   - выводим на экран значение PRINT_SHA1
$ rm -rf v0.1.0.0.tar.gz    - удаляем
```

```ShellSession
$ git clone https://github.com/ruslo/hunter projects/hunter     - кКлонируем hunter в projects
$ cd projects/hunter && git checkout v0.19.137      -  переходим в hunter на v0.19.137
$ git remote show      - показываем удаленные серверы
origin
$ hub fork              - делаем fork
$ git remote show       - показываем удаленные серверы
anasteyshakoshman
origin
$ git remote show ${GITHUB_USERNAME}    - узнаем информацию о сервере ${GITHUB_USERNAME}
```

```ShellSession
$ mkdir cmake/projects/print  - создаем папку print
$ cat > cmake/projects/print/hunter.cmake <<EOF    - включаем библиотеки hunter, настраиваем  
include(hunter_add_version)
include(hunter_cacheable)
include(hunter_cmake_args)
include(hunter_download)
include(hunter_pick_scheme)

hunter_add_version(
    PACKAGE_NAME
    print
    VERSION
    "0.1.0.0"
    URL
    "https://github.com/${GITHUB_USERNAME}/lab09/archive/v0.1.0.0.tar.gz"
    SHA1
    ${PRINT_SHA1}
)

hunter_pick_scheme(DEFAULT url_sha1_cmake)

hunter_cmake_args(
    print
    CMAKE_ARGS
    BUILD_EXAMPLES=NO
    BUILD_TESTS=NO
)
hunter_cacheable(print)
hunter_download(PACKAGE_NAME print)
EOF
```

```ShellSession
$ cat >> cmake/configs/default.cmake <<EOF    - выбираем версию для построения
hunter_config(print VERSION 0.1.0.0)
EOF

```

```ShellSession
$ git add .        - выбираем изменения
$ git commit -m"added print package" - коммитим
$ git push ${GIHUB_USERNAME} master    - добавляем сервер на master
$ git tag v0.19.137.1       - просматриваем версии
$ git push ${GIHUB_USERNAME} master --tags - добавляем ветку на сервер
$ cd ..  - выходим из текущей папки
```

```ShellSession
$ export HUNTER_ROOT=`pwd`/hunter - определение переменной окружения для добавления новго пакета
$ mkdir lab10 && cd lab10  - создаем папку lab10 И переходим туда
$ git init    - инициализируем
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab10  - ообщаем локальному репозиторию, где находится главный репозиторий
```

```ShellSession
$ mkdir sources   - создание папки sources
$ cat > sources/demo.cpp <<EOF - заполнение demo.cpp
#include <print.hpp>

int main(int argc, char** argv) {
	std::string text;
	while(std::cin >> text) {
		std::ofstream out("log.txt", std::ios_base::app);
		print(text, out);
		out << std::endl;
	}
}
EOF
```

```ShellSession
$ wget https://github.com/hunter-packages/gate/archive/v0.8.1.tar.gz  - получение пакета 
$ tar -xzvf v0.8.1.tar.gz gate-0.8.1/cmake/HunterGate.cmake
$ mkdir cmake  - создание папки mkdir
$ mv gate-0.8.1/cmake/HunterGate.cmake cmake   - перемещаем в cmake
$ rm -rf gate*/ - удаляем
$ rm *.tar.gz  - удаляем
```

```ShellSession
$ cat > CMakeLists.txt <<EOF   - устанавливаем минимальную обязательную версю cmake для проекта, свойства по умолчанию
cmake_minimum_required(VERSION 3.0)

set(CMAKE_CXX_STANDARD 11)
EOF
```

```ShellSession
$ wget https://github.com/${GITHUB_USERNAME}/hunter/archive/v0.19.137.1.tar.gz - получаем архив
$ export HUNTER_SHA1=`openssl sha1 v0.19.137.1.tar.gz | cut -d'=' -f2 | cut -c2-41` - определяем HUNTER_SHA1
$ echo $HUNTER_SHA1  - вывод на экран HUNTER_SHA1
$ rm -rf v0.19.137.1.tar.gz - удаляем
```

```ShellSession
$ cat >> CMakeLists.txt <<EOF - включаем HunterGate в CMakeLists.txt     

include(cmake/HunterGate.cmake)

HunterGate(
    URL "https://github.com/${GITHUB_USERNAME}/hunter/archive/v0.19.137.1.tar.gz"
    SHA1 "${HUNTER_SHA1}"
)
EOF
```

```ShellSession
$ cat >> CMakeLists.txt <<EOF    - установка target-объектов

project(demo)

hunter_add_package(print)
find_package(print)

add_executable(demo \${CMAKE_CURRENT_SOURCE_DIR}/sources/demo.cpp)
target_link_libraries(demo print)

install(TARGETS demo RUNTIME DESTINATION bin)
EOF
```

```ShellSession
$ cat > .gitignore <<EOF - создаем .gitignore
*build*/
*install*/
*.swp
EOF
```

```ShellSession
$ cat > README.md <<EOF     - помещаем значок MARKDOWN в README.md 
[![Build Status](https://travis-ci.org/${GITHUB_USERNAME}/lab10.svg?branch=master)](https://travis-ci.org/${GITHUB_USERNAME}/lab10)
the demo application redirects data from stdin to a file **log.txt** using a package **print**.
EOF
```

```ShellSession
$ cat > .travis.yml <<EOF - создаем файл для связи с travis
language: cpp

script:   
- cmake -H. -B_build
- cmake --build _build
EOF
```

```ShellSession
$ travis lint - проверка конфигурации
```

```ShellSession
$ git add .     - выбираем все изменения
$ git commit -m"first commit" - добавляем комментарий
$ git push origin master - выкладываем изменения на удаленный репозиторий
```

```ShellSession
$ travis login --auto        - авторизуемся
$ travis enable   - активируем ЛР10
```

Построение, создание папки artifacts, вывод текста
```ShellSession
$ cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install
$ cmake --build _build --target install
$ mkdir artifacts && cd artifacts
$ echo "text1 text2 text3" | ../_install/bin/demo
$ cat log.txt
```

## Report

```ShellSession
$ popd        - возвращаемся в сохраненную директорию
$ export LAB_NUMBER=10       - задаем переменную окружения
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER} - клонируем задание лр9 в директорию tasks/lab09
$ mkdir reports/lab${LAB_NUMBER} - создаем папку lab09 в reports
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md      - копируем из tasks README.md и сохраняем его в reports под названием REPORT.md
$ cd reports/lab${LAB_NUMBER}   - заходим в директорию reports/lab09
$ edit REPORT.md     - редактируем REPORT.md
$ gistup -m "lab${LAB_NUMBER}"     - cоздаем gist
```

## Links

- [hub](https://hub.github.com/)
- [polly](https://github.com/ruslo/polly)
- [conan](https://conan.io)

```
Copyright (c) 2017 Братья Вершинины
```
