# Laboratory work V

Данная лабораторная работа посвящена изучению фреймворков для модульного тестирования на примере GoogleTest.

Цель работы — настроить сборку библиотеки `banking`, написать модульные тесты для классов `Account` и `Transaction` с использованием mock-объектов, проверить покрытие кода и настроить CI с отправкой покрытия в Coveralls.

Репозиторий: https://github.com/labubutrue/lab05

---

## 1. Подключение GoogleTest как git submodule

### Задание

Подключить GoogleTest к проекту. Библиотека должна быть добавлена как `git submodule`, а не скопирована в репозиторий.

### Команды

```bash
mkdir -p third-party
git submodule add https://github.com/google/googletest third-party/gtest
```

### Вывод

<details>
<summary>Вывод git submodule add</summary>

```text
Cloning into '/Users/mac/tp-labs/lab05/third-party/gtest'...
remote: Enumerating objects: 29360, done.
remote: Counting objects: 100% (540/540), done.
remote: Compressing objects: 100% (357/357), done.
Receiving objects: 100% (29360/29360), 14.12 MiB | 5.69 MiB/s, done.
remote: Total 29360 (delta 384), reused 189 (delta 178), pack-reused 28820 (from 3)
Resolving deltas: 100% (21768/21768), done.
```

</details>

```bash
cd third-party/gtest
git checkout release-1.8.1
```

### Вывод

<details>
<summary>Вывод git checkout release-1.8.1</summary>

```text
Note: switching to 'release-1.8.1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 2fe3bd99 Merge pull request #1433 from dsacre/fix-clang-warnings
```

</details>

```bash
cd ../..
git add .gitmodules third-party/gtest .gitignore
git commit -m "Add GoogleTest as submodule"
```

### Вывод

```text
[main 3ecc283] Add GoogleTest as submodule
 3 files changed, 13 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 third-party/gtest
```

Проверка submodule:

```bash
git submodule status
```

### Вывод

```text
 2fe3bd994b3189899d93f1d5a881e725e046fdc2 third-party/gtest (release-1.8.0-1329-g2fe3bd99)
```

```bash
git ls-files -s third-party/gtest
```

### Вывод

```text
160000 2fe3bd994b3189899d93f1d5a881e725e046fdc2 0	third-party/gtest
```

Режим `160000` подтверждает, что `third-party/gtest` является git submodule.

---

## 2. Настройка CMake

### Задание

Создать `CMakeLists.txt` для библиотеки `banking` и настроить сборку тестов.

### `banking/CMakeLists.txt`

```cmake
add_library(banking
    Account.cpp
    Transaction.cpp
)

target_include_directories(banking PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
)
```

### Корневой `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.10)

project(lab05 VERSION 1.0.0)

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    add_compile_options(-Wno-error=maybe-uninitialized)
endif()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_TESTS "Build tests" ON)
option(ENABLE_COVERAGE "Enable coverage" ON)

if(ENABLE_COVERAGE AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_compile_options(--coverage)
    add_link_options(--coverage)
endif()

add_subdirectory(banking)

if(BUILD_TESTS)
    enable_testing()

    if(MSVC)
        set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
    endif()

    add_subdirectory(third-party/gtest)
    add_subdirectory(tests)
endif()
```

### `tests/CMakeLists.txt`

```cmake
add_executable(check
    test_account.cpp
    test_transaction.cpp
)

target_link_libraries(check
    banking
    gtest_main
    gmock
)

add_test(NAME check COMMAND check)
```

### Конфигурация

```bash
cmake -S . -B build \
  -DBUILD_TESTS=ON \
  -DENABLE_COVERAGE=ON \
  -DCMAKE_POLICY_VERSION_MINIMUM=3.5
```

### Вывод

<details>
<summary>Вывод CMake</summary>

```text
-- The C compiler identification is AppleClang 17.0.0.17000013
-- The CXX compiler identification is AppleClang 17.0.0.17000013
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
CMake Deprecation Warning at third-party/gtest/CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.10 will be removed from a future version of
  CMake.

CMake Deprecation Warning at third-party/gtest/googlemock/CMakeLists.txt:42 (cmake_minimum_required):
  Compatibility with CMake < 3.10 will be removed from a future version of
  CMake.

CMake Deprecation Warning at third-party/gtest/googletest/CMakeLists.txt:49 (cmake_minimum_required):
  Compatibility with CMake < 3.10 will be removed from a future version of
  CMake.

CMake Warning (dev) at third-party/gtest/googletest/cmake/internal_utils.cmake:239 (find_package):
  Policy CMP0148 is not set: The FindPythonInterp and FindPythonLibs modules
  are removed.
Call Stack (most recent call first):
  third-party/gtest/googletest/CMakeLists.txt:84 (include)
This warning is for project developers.  Use -Wno-dev to suppress it.

-- Found PythonInterp: /opt/homebrew/bin/python3.14 (found version "3.14.7")
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Success
-- Found Threads: TRUE
-- Configuring done (1.3s)
-- Generating done (0.0s)
-- Build files have been written to: /Users/mac/tp-labs/lab05/build
```

</details>

### Сборка

```bash
cmake --build build
```

### Вывод

```text
[  7%] Building CXX object banking/CMakeFiles/banking.dir/Account.cpp.o
[ 14%] Building CXX object banking/CMakeFiles/banking.dir/Transaction.cpp.o
[ 21%] Linking CXX static library libbanking.a
[ 21%] Built target banking
[ 28%] Building CXX object third-party/gtest/googlemock/gtest/CMakeFiles/gtest.dir/src/gtest-all.cc.o
[ 35%] Linking CXX static library libgtest.a
[ 35%] Built target gtest
[ 42%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock.dir/src/gmock-all.cc.o
[ 50%] Linking CXX static library libgmock.a
[ 50%] Built target gmock
[ 57%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock_main.dir/src/gmock_main.cc.o
[ 64%] Linking CXX static library libgmock_main.a
[ 64%] Built target gmock_main
[ 71%] Building CXX object third-party/gtest/googlemock/gtest/CMakeFiles/gtest_main.dir/src/gtest_main.cc.o
[ 78%] Linking CXX static library libgtest_main.a
[ 78%] Built target gtest_main
[ 85%] Building CXX object tests/CMakeFiles/check.dir/test_account.cpp.o
[ 92%] Building CXX object tests/CMakeFiles/check.dir/test_transaction.cpp.o
[100%] Linking CXX executable check
[100%] Built target check
```

---

## 3. Модульные тесты `Account` и `Transaction`

### Задание

Создать модульные тесты для `Transaction` и `Account`:
- использовать mock-объекты;
- обеспечить покрытие кода 100%.

В тестах `Transaction` используется `MockAccount` через Google Mock для проверки вызовов `Lock`, `Unlock`, `ChangeBalance` и `GetBalance`.

### Запуск тестов

```bash
ctest --test-dir build --output-on-failure
```

### Вывод

```text
Test project /Users/mac/tp-labs/lab05/build
    Start 1: check
1/1 Test #1: check ............................   Passed    0.86 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.87 sec
```

```bash
./build/tests/check
```

### Вывод

<details>
<summary>Полный вывод тестов</summary>

```text
Running main() from /Users/mac/tp-labs/lab05/third-party/gtest/googletest/src/gtest_main.cc
[==========] Running 14 tests from 2 test cases.
[----------] Global test environment set-up.
[----------] 5 tests from Account
[ RUN      ] Account.Constructor
[       OK ] Account.Constructor (0 ms)
[ RUN      ] Account.ChangeBalanceThrowsIfNotLocked
[       OK ] Account.ChangeBalanceThrowsIfNotLocked (0 ms)
[ RUN      ] Account.ChangeBalanceWorksIfLocked
[       OK ] Account.ChangeBalanceWorksIfLocked (0 ms)
[ RUN      ] Account.DoubleLockThrows
[       OK ] Account.DoubleLockThrows (0 ms)
[ RUN      ] Account.UnlockWorks
[       OK ] Account.UnlockWorks (0 ms)
[----------] 5 tests from Account (0 ms total)

[----------] 9 tests from Transaction
[ RUN      ] Transaction.DefaultFee
[       OK ] Transaction.DefaultFee (0 ms)
[ RUN      ] Transaction.SetFee
[       OK ] Transaction.SetFee (0 ms)
[ RUN      ] Transaction.SameAccountThrows
[       OK ] Transaction.SameAccountThrows (0 ms)
[ RUN      ] Transaction.NegativeSumThrows
[       OK ] Transaction.NegativeSumThrows (0 ms)
[ RUN      ] Transaction.TooSmallSumThrows
[       OK ] Transaction.TooSmallSumThrows (0 ms)
[ RUN      ] Transaction.FeeTooBigReturnsFalse
[       OK ] Transaction.FeeTooBigReturnsFalse (0 ms)
[ RUN      ] Transaction.SuccessTransfer
1 send to 2 $200
Balance 1 is 1000
Balance 2 is 499
[       OK ] Transaction.SuccessTransfer (0 ms)
[ RUN      ] Transaction.CallsAccountMethods
1 send to 2 $200
Balance 1 is

GMOCK WARNING:
Uninteresting mock function call - returning default value.
    Function call: GetBalance()
          Returns: 0
NOTE: You can safely ignore the above warning unless this call should not happen.  Do not suppress it by blindly adding an EXPECT_CALL() if you don't mean to enforce the call.  See https://github.com/google/googletest/blob/master/googlemock/docs/CookBook.md#knowing-when-to-expect for details.
0
Balance 2 is 1000
[       OK ] Transaction.CallsAccountMethods (0 ms)
[ RUN      ] Transaction.DebitFailureReturnsFalse
1 send to 2 $200
Balance 1 is 1000
Balance 2 is 0
[       OK ] Transaction.DebitFailureReturnsFalse (0 ms)
[----------] 9 tests from Transaction (0 ms total)

[----------] Global test environment tear-down
[==========] 14 tests from 2 test cases ran. (0 ms total)
[  PASSED  ] 14 tests.
```

</details>

---

## 4. Проверка покрытия кода

### Задание

Покрытие кода должно составлять 100%.

```bash
xcrun llvm-cov gcov build/banking/CMakeFiles/banking.dir/Account.cpp.gcno
```

### Вывод

```text
File '/Users/mac/tp-labs/lab05/banking/Account.cpp'
Lines executed:100.00% of 13
Creating 'Account.cpp.gcov'
```

```bash
xcrun llvm-cov gcov build/banking/CMakeFiles/banking.dir/Transaction.cpp.gcno
```

### Вывод

<details>
<summary>Полный вывод для Transaction.cpp</summary>

```text
File '/Users/mac/tp-labs/lab05/banking/Transaction.cpp'
Lines executed:100.00% of 34
Creating 'Transaction.cpp.gcov'

File '/Users/mac/tp-labs/lab05/banking/Account.h'
Lines executed:0.00% of 1
Creating 'Account.h.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/stdexcept'
Lines executed:100.00% of 1
Creating 'stdexcept.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/__ostream/basic_ostream.h'
Lines executed:22.22% of 27
Creating 'basic_ostream.h.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/__string/char_traits.h'
Lines executed:0.00% of 3
Creating 'char_traits.h.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/locale'
Lines executed:0.00% of 29
Creating 'locale.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/__iterator/ostreambuf_iterator.h'
Lines executed:0.00% of 2
Creating 'ostreambuf_iterator.h.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/ios'
Lines executed:0.00% of 23
Creating 'ios.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/streambuf'
Lines executed:0.00% of 2
Creating 'streambuf.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/string'
Lines executed:0.00% of 14
Creating 'string.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/__memory/compressed_pair.h'
Lines executed:0.00% of 10
Creating 'compressed_pair.h.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/__memory/allocator.h'
Lines executed:0.00% of 2
Creating 'allocator.h.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/__memory/pointer_traits.h'
Lines executed:0.00% of 4
Creating 'pointer_traits.h.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/__locale'
Lines executed:0.00% of 3
Creating '__locale.gcov'

File '/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1/__string/constexpr_c_functions.h'
Lines executed:0.00% of 2
Creating 'constexpr_c_functions.h.gcov'
```

</details>

Проверка отсутствия непокрытых исполняемых строк:

```bash
grep -n -E '#####|=====' Account.cpp.gcov Transaction.cpp.gcov
```

Команда не вывела строк.

Итоговая сводка LCOV только по `banking`:

```bash
lcov --summary coverage.info
```

### Вывод

```text
Reading tracefile coverage.info.

Summary coverage rate:
  source files: 4
  lines.......: 100.0% (50 of 50 lines)
  functions...: 92.0% (23 of 25 functions)

Message summary:
  no messages were reported
```

Покрытие строк исходного кода `banking` составляет **100% (50 из 50 строк)**. Показатель функций в локальной AppleClang-сборке учитывает сгенерированные компилятором варианты виртуальных деструкторов.

---

## 5. CI и Coveralls

### Задание

Настроить процедуру автоматической сборки и тестирования в CI и настройку Coveralls.

Вместо Travis CI используется GitHub Actions.

### `.github/workflows/tests.yml`

```yaml
name: Lab05 Tests and Coverage

on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y cmake g++ lcov

      - name: Configure
        run: |
          cmake -S . -B build \
            -DBUILD_TESTS=ON \
            -DENABLE_COVERAGE=ON \
            -DCMAKE_POLICY_VERSION_MINIMUM=3.5

      - name: Build
        run: cmake --build build

      - name: Run tests
        run: ctest --test-dir build --output-on-failure

      - name: Capture coverage
        run: |
          lcov --capture \
            --directory build \
            --output-file coverage.info \
            --ignore-errors mismatch

      - name: Filter coverage
        run: |
          lcov --extract coverage.info \
            '*/banking/*' \
            --output-file coverage.info \
            --ignore-errors unused

      - name: Show coverage
        run: lcov --summary coverage.info

      - name: Upload coverage to Coveralls
        uses: coverallsapp/github-action@v2
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          file: coverage.info
          format: lcov
```

Успешный запуск тестов и покрытия:

https://github.com/labubutrue/lab05/actions/runs/33705263299

Статус: **Success**.

Дополнительно настроена matrix-сборка:
- Linux GCC;
- Linux Clang;
- Windows MSVC.

Успешный запуск:

https://github.com/labubutrue/lab05/actions/runs/33705263300

Статус: **Success**, завершены 3 job.

---

## 6. Проверка структуры репозитория

Сборочные артефакты и результаты покрытия исключены через `.gitignore`.

Проверка:

```bash
git ls-files | grep -E '(^|/)(_?build|coverage)(/|$)|\.gcda$|\.gcno$|\.gcov$|coverage\.info$'
```

Команда не вывела строк.

Финальное состояние:

```bash
git status
```

### Вывод

```text
On branch main

Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

---

## Итог

В ходе лабораторной работы:

- GoogleTest подключён как **git submodule**;
- создана CMake-конфигурация библиотеки `banking`;
- написаны модульные тесты для `Account` и `Transaction`;
- используется `MockAccount` на базе Google Mock;
- все 14 тестов проходят успешно;
- достигнуто **100% покрытие строк `banking` (50/50)**;
- настроен GitHub Actions для сборки, тестирования, проверки покрытия и отправки результата в Coveralls;
- дополнительная CI-матрица успешно собирает проект на Linux GCC, Linux Clang и Windows MSVC;
- сборочные каталоги и coverage-файлы не отслеживаются Git.
