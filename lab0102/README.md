# **Отчет по лабораторной работе №1: «Установка и настройка Git, создание SSH-ключа, работа с GitLab»**

## **Сведения о студенте**

**Дата:** 2026-10-11  
**Семестр:** 3 курс, 5 семестр  
**Группа:** ПИН-б-о-24-1  
**Дисциплина:** Системы контроля версий  
**Студент:** Макаров Роман Дмитриевич

### **Структура готового проекта**

```
hello-git/
├── .git/          # Локальная база данных репозитория Git
├── .gitignore     # Правила игнорирования файлов
└── README.md      # Описание проекта и сведения об авторе
```

### **Ссылка на репозиторий GitLab**

```
https://gitlab.com/drrrm0471/hello-git
```

### **1. ЦЕЛЬ РАБОТЫ**

> 1. Установить и настроить систему контроля версий Git.
> 2. Создать SSH-ключ для безопасного подключения к GitLab.
> 3. Зарегистрироваться на GitLab и добавить SSH-ключ в профиль.
> 4. Создать локальный репозиторий, выполнить базовые операции (add, commit, push).
> 5. Создать удалённый репозиторий на GitLab и загрузить проект.
> 6. Настроить игнорирование файлов с помощью `.gitignore`.

### **2. ЗАДАЧИ РАБОТЫ**

**Выполнены следующие задачи:**

> 1. Проверка установленной версии Git (Git for Windows, терминал Git Bash).
> 2. Настройка глобального профиля Git: `user.name`, `user.email`, `core.editor`, `color.ui`, `init.defaultBranch`; исправление дублирующегося значения `user.name`.
> 3. Проверка каталога `~/.ssh`; использование существующей пары ключей ed25519 без перезаписи.
> 4. Запуск `ssh-agent` и добавление приватного ключа командой `ssh-add`.
> 5. Добавление публичного ключа в профиль GitLab (User Settings → SSH Keys).
> 6. Проверка SSH-подключения к gitlab.com.
> 7. Инициализация локального репозитория `hello-git`, создание `README.md`.
> 8. Индексация и первый коммит по соглашению Conventional Commits; просмотр истории.
> 9. Создание пустого проекта `hello-git` на GitLab, связывание с локальным репозиторием (`origin`), отправка ветки `main`.
> 10. Создание `.gitignore`, проверка игнорирования, коммит и отправка на GitLab.

### **3. ХОД ВЫПОЛНЕНИЯ РАБОТЫ**

**Окружение:** Windows, Git for Windows 2.51.0, терминал Git Bash (MINGW64), платформа gitlab.com.

#### **3.1. Проверка установки Git**

```bash
git --version
```

Вывод команды в терминале:

```
MSI@drrrm MINGW64 ~
$ git --version
git version 2.51.0.windows.1
```

Git установлен, версия удовлетворяет требованию методички (2.30+).

#### **3.2. Настройка глобального профиля пользователя Git**

При попытке задать имя пользователя Git сообщил, что параметр `user.name` в глобальной конфигурации имеет несколько значений:

```
$ git config --global user.name "makarov"
warning: user.name has multiple values
error: cannot overwrite multiple values with a single value
       Use a regexp, --add or --replace-all to change user.name.
```

Проверка текущих значений:

```
$ git config --global --get-all user.name
drrrm
Roman
$ git config --global --get-all user.email
ctx2005ctx@yandex.ru
```

Параметр содержал две записи, поэтому обычная команда не может определить, какую из них заменить. Для замены всех значений одним использован ключ `--replace-all`, затем заданы остальные параметры:

```bash
git config --global --replace-all user.name "makarov"
git config --global --replace-all user.email "makarovr0471@gmail.com"
git config --global core.editor nano
git config --global color.ui auto
git config --global init.defaultBranch main
git config --global --list
```

Вывод команды в терминале:

```
$ git config --global --list
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
user.email=makarovr0471@gmail.com
user.name=makarov
core.editor=nano
safe.directory=C:/Windows/System32/python-git-lab1
credential.https://gitflic.ru.provider=generic
color.ui=auto
init.defaultbranch=main
```

Назначение параметров:
- `user.name`, `user.email` — автор каждого коммита; email совпадает с адресом аккаунта GitLab, поэтому коммиты привязываются к профилю;
- `core.editor nano` — редактор для сообщений коммитов;
- `color.ui auto` — цветной вывод в терминале;
- `init.defaultBranch main` — имя первой ветки новых репозиториев (`main`, как на GitLab);

Строки `filter.lfs.*`, `safe.directory`, `credential.https://gitflic.ru...` остались от предыдущей работы (установленный Git LFS и настройки других проектов) и на выполнение работы не влияют.

#### **3.3. Проверка каталога SSH-ключей**

```bash
ls -la ~/.ssh
```

Вывод команды в терминале:

```
$ ls -la ~/.ssh
total 44
drwxr-xr-x 1 MSI 197121    0 Sep  7  2025 ./
drwxr-xr-x 1 MSI 197121    0 Oct 11 20:22 ../
-rw-r--r-- 1 MSI 197121  419 Sep  7  2025 id_ed25519
-rw-r--r-- 1 MSI 197121  108 Sep  7  2025 id_ed25519.pub
-rw-r--r-- 1 MSI 197121 2590 Sep  7  2025 id_rsa
-rw-r--r-- 1 MSI 197121  564 Sep  7  2025 id_rsa.pub
-rw-r--r-- 1 MSI 197121  831 Sep  7  2025 known_hosts
-rw-r--r-- 1 MSI 197121   93 Sep  7  2025 known_hosts.old
```

В каталоге уже имеются пары ключей ed25519 и RSA, созданные ранее.

#### **3.4. Генерация ключевой пары SSH**

```bash
ssh-keygen -t ed25519 -C "makarovr0471@gmail.com"
```

Вывод команды в терминале:

```
$ ssh-keygen -t ed25519 -C "makarovr0471@gmail.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/c/Users/MSI/.ssh/id_ed25519):
/c/Users/MSI/.ssh/id_ed25519 already exists.
Overwrite (y/n)? n
```

Так как ключ ed25519 уже существует и не был привязан к используемому аккаунту GitLab, при запросе перезаписи выбран ответ `n`, и далее используется существующий ключ. Алгоритм ed25519 выбран как современный, быстрый и рекомендуемый GitLab.

#### **3.5. Запуск ssh-agent и добавление ключа**

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
```

Вывод команд в терминале:

```
$ eval "$(ssh-agent -s)"
Agent pid 1125

$ ssh-add ~/.ssh/id_ed25519
Identity added: /c/Users/MSI/.ssh/id_ed25519 (makarov*******@gmail.com)

$ ssh-add -l
256 SHA256:hMjZ…e13s makarov*******@gmail.com (ED25519)
```

`ssh-agent` хранит расшифрованный ключ в памяти на время сеанса, `ssh-add -l` показывает загруженные ключи. Адрес в конце — комментарий, заданный при создании ключа; на аутентификацию он не влияет.

#### **3.6. Добавление публичного ключа в GitLab**

```bash
cat ~/.ssh/id_ed25519.pub
```

Вывод команды в терминале:

```
$ cat ~/.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA…OMw makarov*******@gmail.com
```

Публичный ключ добавлен в профиль GitLab: **User Settings → SSH Keys → Add new key**.

Параметры добавленного ключа:

| Параметр | Значение |
|:---|:---|
| Title | Windows MSI |
| Usage type | Authentication & Signing |
| Created | Oct 11, 2026 |
| Expires | Oct 11, 2027 |
| Fingerprint SHA256 | `hMjZ…e13s` |
| Fingerprint MD5 | `b2:9a:3c:ca:e3:f3:37:28:83:3b:f3:9d:36:75:3d:37` |

Отпечаток SHA256 совпадает с выводом `ssh-add -l`, то есть в GitLab добавлен тот же ключ. Комментарий ключа при добавлении заменён на актуальный адрес `makarovr0471@gmail.com`.

#### **3.7. Проверка SSH-подключения к GitLab**

```bash
ssh -T git@gitlab.com
```

Вывод команды в терминале:

```
$ ssh -T git@gitlab.com
The authenticity of host 'gitlab.com (172.65.251.78)' can't be established.
ED25519 key fingerprint is SHA256:eUXGGm1YGsMAS7vkcx6JOJdOGHPem5gQp4taiCfCLB8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'gitlab.com' (ED25519) to the list of known hosts.
Welcome to GitLab, @drrrm0471!
```

При первом подключении SSH запрашивает подтверждение подлинности сервера. Отпечаток совпадает с официально опубликованным отпечатком gitlab.com, поэтому введено `yes`; сервер добавлен в `~/.ssh/known_hosts`. Ответ `Welcome to GitLab, @drrrm0471!` подтверждает успешную аутентификацию по ключу.

#### **3.8. Создание локального репозитория**

```bash
cd /c/Users/MSI/Documents/Projects/Working/vcs/lab-01
mkdir hello-git
cd hello-git
git init
ls -la
```

Вывод команд в терминале:

```
Initialized empty Git repository in C:/Users/MSI/Documents/Projects/Working/vcs/lab-01/hello-git/.git/

$ ls -la
total 4
drwxr-xr-x 1 MSI 197121 0 Oct 11 21:52 ./
drwxr-xr-x 1 MSI 197121 0 Oct 11 21:52 ../
drwxr-xr-x 1 MSI 197121 0 Oct 11 21:52 .git/
```

Команда `git init` создала скрытый каталог `.git` — локальную базу данных репозитория.

#### **3.9. Создание файла README.md**

Файл создан командами `echo`:

```bash
echo "# Hello Git — первое знакомство с системой контроля версий" > README.md
echo "" >> README.md
echo "## Описание проекта" >> README.md
echo "Этот проект создан в рамках лабораторной работы по дисциплине 'Системы контроля версий'." >> README.md
echo "" >> README.md
echo "## Автор" >> README.md
echo "Макаров Роман Дмитриевич, группа ПИН-б-о-24-1" >> README.md
```

Содержимое файла:

```
$ cat README.md
# Hello Git — первое знакомство с системой контроля версий

## Описание проекта
Этот проект создан в рамках лабораторной работы по дисциплине 'Системы контроля версий'.

## Автор
Макаров Роман Дмитриевич, группа ПИН-б-о-24-1
```

#### **3.10. Индексация и первый коммит**

```bash
git status
git add README.md
git status
git commit -m "feat: add README.md with project description"
```

Вывод команд в терминале:

```
$ git status
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        README.md

nothing added to commit but untracked files present (use "git add" to track)

$ git add README.md

$ git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   README.md

$ git commit -m "feat: add README.md with project description"
[main (root-commit) 0eb5c91] feat: add README.md with project description
 1 file changed, 7 insertions(+)
 create mode 100644 README.md
```

До `git add` файл находится в состоянии *Untracked* (Git его видит, но не отслеживает), после — в индексе (*Changes to be committed*). Коммит создан с сообщением по соглашению Conventional Commits (`feat:` — новая функциональность). Пометка `root-commit` означает первый коммит в истории (без родителя). Ветка называется `main` благодаря настройке `init.defaultBranch`.

#### **3.11. Просмотр истории коммитов**

```bash
git log
git log --oneline
git log --oneline --graph --all
```

Вывод команд в терминале:

```
$ git log
commit 0eb5c915f949f63fd8aca2e68111779d57d5e638 (HEAD -> main)
Author: makarov <makarovr0471@gmail.com>
Date:   Sun Oct 11 22:03:38 2026 +0300

    feat: add README.md with project description

$ git log --oneline
0eb5c91 (HEAD -> main) feat: add README.md with project description

$ git log --oneline --graph --all
* 0eb5c91 (HEAD -> main) feat: add README.md with project description
```

`git log` выводит полный хеш SHA-1, автора, дату и сообщение; `--oneline` — сокращённый хеш и сообщение; `--graph --all` — граф всех веток.

#### **3.12. Создание удалённого репозитория и отправка изменений**

На GitLab создан пустой публичный проект `hello-git` (**New project → Create blank project**) без инициализации файлом README — иначе на сервере появился бы собственный первый коммит, не связанный с локальной историей, и отправка была бы отклонена.

```bash
git remote add origin git@gitlab.com:drrrm0471/hello-git.git
git remote -v
git push -u origin main
```

Вывод команд в терминале:

```
$ git remote add origin git@gitlab.com:drrrm0471/hello-git.git

$ git remote -v
origin  git@gitlab.com:drrrm0471/hello-git.git (fetch)
origin  git@gitlab.com:drrrm0471/hello-git.git (push)

$ git push -u origin main
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 16 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 484 bytes | 484.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To gitlab.com:drrrm0471/hello-git.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

Удалённый репозиторий добавлен под стандартным именем `origin` по SSH-адресу. Ключ `-u` связал локальную ветку `main` с `origin/main`, поэтому далее достаточно команды `git push`. Отправлено 3 объекта: содержимое файла (blob), структура каталога (tree) и коммит (commit).

Проверка в веб-интерфейсе GitLab: в проекте `drrrm0471/hello-git` отображается 1 коммит, 1 ветка `main`, последний коммит `0eb5c915` «feat: add README.md with project description» и содержимое `README.md`.

#### **3.13. Настройка игнорирования файлов (.gitignore)**

Файл `.gitignore` создан командами `echo`:

```bash
echo "# Системные файлы" > .gitignore
echo ".DS_Store" >> .gitignore
echo "Thumbs.db" >> .gitignore
echo "" >> .gitignore
echo "# Временные файлы" >> .gitignore
echo "*.tmp" >> .gitignore
echo "*.log" >> .gitignore
echo "" >> .gitignore
echo "# Файлы IDE" >> .gitignore
echo ".vscode/" >> .gitignore
echo ".idea/" >> .gitignore
```

Содержимое файла:

```gitignore
# Системные файлы
.DS_Store
Thumbs.db

# Временные файлы
*.tmp
*.log

# Файлы IDE
.vscode/
.idea/
```

Для проверки правил в каталог проекта помещён пустой файл `test.log`:

```
$ ls -la
total 14
drwxr-xr-x 1 MSI 197121   0 Oct 11 22:07 ./
drwxr-xr-x 1 MSI 197121   0 Oct 11 21:59 ../
drwxr-xr-x 1 MSI 197121   0 Oct 11 22:06 .git/
-rw-r--r-- 1 MSI 197121 131 Oct 11 22:07 .gitignore
-rw-r--r-- 1 MSI 197121 393 Oct 11 22:00 README.md
-rw-r--r-- 1 MSI 197121   0 Oct 11 22:07 test.log

$ git status
On branch main
Your branch is up to date with 'origin/main'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore

nothing added to commit but untracked files present (use "git add" to track)
```

Файл `test.log` присутствует в каталоге, но `git status` его не показывает — он исключён правилом `*.log`. Сам `.gitignore` виден как новый файл: его необходимо закоммитить, чтобы правила действовали у всех участников проекта.

```bash
git add .gitignore
git commit -m "chore: add .gitignore"
git push
git log --oneline
rm test.log
```

Вывод команд в терминале:

```
[main 68a4024] chore: add .gitignore
 1 file changed, 11 insertions(+)
 create mode 100644 .gitignore
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 387 bytes | 387.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To gitlab.com:drrrm0471/hello-git.git
   0eb5c91..68a4024  main -> main
68a4024 (HEAD -> main, origin/main) chore: add .gitignore
0eb5c91 feat: add README.md with project description
```

Тип коммита `chore:` — служебное изменение. Метка `origin/main` рядом с `HEAD -> main` показывает, что локальная и удалённая ветки совпадают — все изменения отправлены на GitLab.

### **4. ОТВЕТЫ НА КОНТРОЛЬНЫЕ ВОПРОСЫ**

#### **1. Что такое распределённая система контроля версий и чем она отличается от централизованной?**

В централизованной системе (SVN, CVS) история изменений хранится только на сервере: без доступа к нему нельзя создать коммит или просмотреть историю, а потеря сервера означает потерю истории. В распределённой системе (Git, Mercurial) каждый участник имеет полную копию репозитория со всей историей. Коммиты, ветки и просмотр истории выполняются локально и без сети, сервер (например, GitLab) используется для обмена изменениями, а любая копия может служить резервной.

#### **2. Для чего нужен SSH-ключ при работе с GitLab?**

Для аутентификации без пароля. Используется пара ключей: публичный хранится в профиле GitLab, приватный — на компьютере пользователя. При подключении сервер проверяет, что клиент владеет приватным ключом, соответствующим публичному; пароль по сети не передаётся. Вводить учётные данные при каждой операции `push`/`pull` не требуется, а доступ можно отозвать, удалив ключ из профиля.

#### **3. В чём разница между `git add` и `git commit`?**

`git add` переносит изменения из рабочего каталога в индекс (staging area) — подготавливает их к сохранению. `git commit` сохраняет содержимое индекса в историю репозитория как новый снимок с автором, датой, сообщением и уникальным хешем.

#### **4. Что такое staging area (индекс) в Git?**

Промежуточная область между рабочим каталогом и историей коммитов (файл `.git/index`), в которой формируется состав следующего коммита. Индекс позволяет включать в коммит только часть изменений и разбивать работу на логически завершённые коммиты.

#### **5. Как проверить, какие файлы были изменены, но ещё не добавлены в коммит?**

Командой `git status`: раздел *Changes not staged for commit* — изменённые, но не добавленные в индекс файлы, *Untracked files* — новые неотслеживаемые. Построчные изменения показывает `git diff`, а уже добавленные в индекс — `git diff --staged`.

#### **6. Для чего нужен файл `.gitignore`? Приведите примеры.**

Файл задаёт шаблоны файлов и каталогов, которые Git не отслеживает и не предлагает добавить в репозиторий. Это исключает попадание в историю системного мусора, временных файлов, результатов сборки и секретов. Примеры: `*.log`, `*.tmp` (журналы и временные файлы), `.vscode/`, `.idea/` (настройки IDE), `Thumbs.db`, `.DS_Store` (системные файлы), `bin/`, `obj/`, `node_modules/` (сборка и зависимости), `.env` (пароли и ключи).

#### **7. Как связать локальный репозиторий с удалённым?**

Командой `git remote add origin <адрес>` (в работе — `git@gitlab.com:drrrm0471/hello-git.git`), проверка — `git remote -v`. Первая отправка с установкой связи веток — `git push -u origin main`. Альтернатива — `git clone <адрес>`, при котором связь с `origin` настраивается автоматически.

#### **8. Чем отличается `git fetch` от `git pull`?**

`git fetch` загружает новые коммиты с сервера и обновляет удалённые ветки (`origin/main`), не изменяя локальные ветки и файлы. `git pull` выполняет `git fetch` и затем сливает (merge или rebase) загруженные изменения в текущую ветку, изменяя рабочие файлы.

### **5. ИСПОЛЬЗУЕМЫЕ КОМАНДЫ**

```bash
# Проверка установки
git --version

# Глобальная настройка Git
git config --global --get-all user.name
git config --global --get-all user.email
git config --global --replace-all user.name "makarov"
git config --global --replace-all user.email "makarovr0471@gmail.com"
git config --global core.editor nano
git config --global color.ui auto
git config --global init.defaultBranch main
git config --global --list

# SSH-ключи
ls -la ~/.ssh
ssh-keygen -t ed25519 -C "makarovr0471@gmail.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
ssh -T git@gitlab.com

# Локальный репозиторий
mkdir hello-git && cd hello-git
git init
git status
git add README.md
git commit -m "feat: add README.md with project description"
git log
git log --oneline
git log --oneline --graph --all

# Удалённый репозиторий
git remote add origin git@gitlab.com:drrrm0471/hello-git.git
git remote -v
git push -u origin main

# .gitignore
git add .gitignore
git commit -m "chore: add .gitignore"
git push
```

### **6. ВЫВОДЫ**

> 1. Проверена установка Git for Windows 2.51.0 и настроен глобальный профиль пользователя; при этом устранено дублирование параметра `user.name` с помощью `git config --replace-all`.
> 2. Организовано безопасное подключение к GitLab по протоколу SSH: использован существующий ключ ed25519, ключ загружен в `ssh-agent`, публичная часть добавлена в профиль, подключение подтверждено ответом `Welcome to GitLab, @drrrm0471!`.
> 3. Освоен базовый цикл работы с Git: инициализация репозитория, индексация, создание коммитов по соглашению Conventional Commits и просмотр истории.
> 4. Локальный репозиторий связан с удалённым проектом `drrrm0471/hello-git` на GitLab, изменения отправлены командой `git push`.
> 5. Настроен файл `.gitignore`, его работа проверена на тестовом файле `test.log`.
