# МАКСИМАЛЬНЫЙ ТЕСТ: GIT + CMD/BAT + PowerShell + bash/zsh + macOS Terminal

TEST_VERSION: 2.0

## ПРАВИЛА

Выполни все 32 задания одним ответом без интернета, запуска команд или уточняющих вопросов.

Очень важно:
- отличай CMD от PowerShell и bash/zsh;
- для CMD/BAT не используй Unix-синтаксис;
- расширение batch-файлов в заданиях — `.bat`;
- не выдумывай команды Git/system tools;
- destructive commands должны иметь safeguards;
- если команда зависит от ОС/версии — явно скажи;
- не предлагай опасные wildcard/rm/del без проверки путей.

Начало:
```text
MODEL: <название модели или UNKNOWN>
MODE: <thinking|non-thinking|UNKNOWN>
TEST_VERSION: 2.0
```

Конец:
```text
FINAL
Выполнено заданий: <число>
Пропущено заданий: <число>
Использованы внешние инструменты: НЕТ
END_OF_TEST
```

Всего заданий: 32.

---

## КОНТЕКСТ ПРОЕКТА ATLAS TOOLING

Запомни:
- Windows shell для проекта: ConEmu + `cmd.exe`;
- Windows commands должны быть совместимы с CMD;
- использовать Windows paths `A:\work\atlas`;
- переход диска через `cd /d`;
- batch files только `.bat`, не `.cmd`;
- пользователь распаковывает ZIP вручную: не давать команды unzip/delete-before-unpack;
- production Linux: Debian 13;
- macOS build machine: Apple Silicon;
- main Git branch: `main`;
- release branch pattern: `release/*`;
- secrets запрещены в Git;
- destructive scripts обязаны иметь dry-run или explicit confirmation guard;
- временный каталог проекта на Linux/macOS: `/var/tmp/atlas`;
- Windows temporary workspace: `A:\work\atlas\tmp`.

---

## 1. GIT RECOVERY / HISTORY / COLLABORATION

### TERM-1
Пользователь сделал `git reset --hard HEAD~5`, затем понял, что потерял 5 локальных коммитов. Дай безопасный recovery flow через reflog и создание recovery branch. Не переписывай main до проверки.

### TERM-2
Найти первый коммит, сломавший тест. Дай полный `git bisect` flow, включая автоматизацию через test command и корректное завершение.

### TERM-3
Объясни merge, rebase, cherry-pick, fast-forward и merge commit. Когда `git merge` НЕ создаёт merge commit?

### TERM-4
После rebase общей ветки нужно force-push. Почему `--force-with-lease` безопаснее `--force`, но не является абсолютной гарантией от ошибок?

### TERM-5
Нужно временно убрать незакоммиченные изменения, переключиться на `release/2.0`, сделать hotfix, вернуться на исходную ветку и восстановить изменения. Дай команды без потери stash и с проверкой текущей ветки.

### TERM-6
Коммит содержит секрет. Опиши правильную последовательность: revoke/rotate, audit, purge history, force update, clones/forks. Почему «удалить файл новым коммитом» недостаточно?

### TERM-7
Разница `git revert` и `git reset` для уже опубликованной shared history. Как откатить merge commit безопасно?

### TERM-8
Работа с несколькими версиями одновременно: покажи `git worktree` flow для `main` и `release/2.0`, включая удаление worktree без удаления branch.

---

## 2. WINDOWS CMD / BAT

### TERM-9
CMD: показать PATH, ERRORLEVEL, текущий каталог, найти `python.exe`, перейти в `A:\work\atlas` независимо от текущего диска.

### TERM-10
Почему это небезопасно в BAT?

```bat
set TARGET=%1
rmdir /s /q %TARGET%
```

Напиши безопасный `.bat` вариант с:
- обязательным аргументом;
- кавычками;
- запретом пустого/root drive/`A:\work\atlas`;
- разрешением удаления только внутри `A:\work\atlas\tmp`;
- dry-run по умолчанию;
- реальное удаление только при втором аргументе `--execute`.

### TERM-11
CMD delayed expansion: объясни разницу `%VAR%` и `!VAR!` внутри `for`/`if` block. Покажи пример бага и исправление через `setlocal EnableDelayedExpansion`.

### TERM-12
BAT: запустить 3 команды последовательно и остановиться при первой ошибке, сохранив её exit code. Покажи корректный шаблон с `errorlevel` и `exit /b`.

### TERM-13
CMD quoting: как корректно передать путь `C:\Program Files\My App\app.exe` и аргумент, содержащий `&`? Объясни metacharacters и почему простая конкатенация пользовательского ввода опасна.

### TERM-14
Напиши `.bat`, который рекурсивно находит `*.log` старше 7 дней без PowerShell. Если это невозможно надёжно чистым CMD без locale-sensitive парсинга дат, скажи это прямо и предложи безопасный внешний системный вариант, не притворяясь, что pure CMD легко решает задачу.

---

## 3. POWERSHELL

### TERM-15
PowerShell: найти 20 процессов с максимальным WorkingSet, вывести Name, Id, WorkingSetMB и отсортировать численно.

### TERM-16
PowerShell: безопасно удалить только `.tmp` старше 3 дней внутри `A:\work\atlas\tmp`, сначала dry-run, затем параметр `-Execute`. Использовать `-LiteralPath` там, где это важно, и `ShouldProcess`/аналогичный guard.

### TERM-17
Объясни `$ErrorActionPreference`, `-ErrorAction Stop`, terminating/non-terminating errors, `try/catch/finally`. Почему `catch` не всегда ловит ошибку cmdlet без `-ErrorAction Stop`?

### TERM-18
PowerShell pipeline передаёт объекты. Дай пример, где текстовый подход ломается, а объектный фильтр по `Length`/`LastWriteTime` корректен.

### TERM-19
Разница single quotes и double quotes в PowerShell, subexpression `$()`, literal `$`, here-string. Покажи безопасную сборку аргументов внешней программе без `Invoke-Expression`.

### TERM-20
PowerShell jobs/runspaces не нужны: нужно параллельно проверить TCP connectivity к 50 hosts с limit=8. Предложи корректный современный подход и явно отметь зависимость от версии PowerShell, если используешь `ForEach-Object -Parallel`.

---

## 4. BASH / ZSH

### TERM-21
Почему `rm -rf $DIR/*` при пустом `DIR` может стать `rm -rf /*`? Почему `rm -rf "$DIR"/*` тоже не достаточно безопасен? Напиши guarded cleanup только содержимого `/var/tmp/atlas/<job-id>`, с проверкой realpath и запретом корня.

### TERM-22
Посчитать число обычных файлов непосредственно в `/var/tmp/atlas` безопасно для пробелов и newlines. Нужен NUL-safe вариант.

### TERM-23
Разница `set -e`, `set -u`, `set -o pipefail`. Назови минимум 5 случаев, где `set -euo pipefail` не делает shell script автоматически корректным.

### TERM-24
Надёжный cleanup temp dir через `trap` в bash. Нужно сохранить original exit status и не скрыть основную ошибку cleanup-ошибкой.

### TERM-25
Bash arrays: безопасно передать список файлов с пробелами/newlines другой команде без string concatenation. Покажи пример с массивом.

### TERM-26
Отличия zsh globbing от bash, особенно unmatched glob. Почему скрипт, работающий в bash, может завершиться ошибкой в zsh на `*.log` без совпадений?

### TERM-27
`find ... -print0 | xargs -0`: где безопасно, а где `-exec ... {} +` проще? Покажи оба варианта для удаления файлов старше 30 дней после предварительного dry-run.

---

## 5. macOS TERMINAL / SYSTEM

### TERM-28
На Apple Silicon определить архитектуру, версию macOS, путь python3 и архитектуру конкретного бинарника. Дай команды `uname`, `sw_vers`, `command -v`, `file`/`lipo` там, где уместно.

### TERM-29
Приложение `.app` не запускается после скачивания из-за quarantine. Объясни Gatekeeper/quarantine. Покажи диагностические команды (`xattr`, `spctl`) и осторожный путь действий. Не советуй бездумно отключать Gatekeeper глобально.

### TERM-30
Разница `launchd` LaunchAgent и LaunchDaemon. Где размещаются plist, от чьего имени запускаются и когда нужен каждый вариант?

---

## 6. CROSS-PLATFORM / LONG CONTEXT

### TERM-31
По памяти перечисли:
- Windows shell;
- Windows root path;
- обязательный способ `cd` между дисками;
- допустимое расширение batch;
- Linux distro;
- macOS architecture;
- main branch;
- Linux/macOS temp dir;
- Windows temp dir;
- правило распаковки ZIP.

### TERM-32
Напиши три отдельные безопасные процедуры очистки временного workspace:
A. Windows CMD/BAT;
B. PowerShell;
C. bash/zsh/macOS Terminal.

Во всех случаях:
- dry-run default;
- exact allowed root;
- запрет empty/root/project-root;
- защита от пробелов;
- явный execute switch/confirmation;
- никаких предположений о symlink/junction safety без проверки;
- не смешивать синтаксис оболочек.

После этого проведи короткий self-audit каждой процедуры и назови минимум по 2 residual risks.

