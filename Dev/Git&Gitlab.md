# Удалить все ветки, кроме master [Grep](Grep.md)
`git branch | grep -v "master" | xargs git branch -D;`

# Удалить submodule
- Delete the relevant section from the .gitmodules file.
- Stage the .gitmodules changes git add .gitmodules
- Delete the relevant section from .git/config.
- Run git rm --cached path_to_submodule (no trailing slash).
- Run rm -rf .git/modules/path_to_submodule (no trailing slash).
- Commit git commit -m "Removed submodule "
- Delete the now untracked submodule files rm -rf path_to_submodule

# Создать submodule
- `git submodule add git@gitlab.web3tech.ru:development/we/toolchain/transactions-factory.git`


```
git --version

git init // создать пустой репозиторий

git status // проверяет рабочую директорию и статус черновой области

git add .  // добавить все файлы в папке проекта в черновую область

git commit -m "message" // создать коммит

  

git config --global user.name "Alex Galin" // задать имя

git config --global user.email "agalin@web3tech.ru" // задать почту

git commit --amend --reset-author // применить новые имя и почту

  

git log // показывает все коммиты в текущей ветке

git ls-files // показывает информацию о файлах в черновой области

  

git checkout <hash> // переключиться на данный коммит (смещение указателя)

  

git checkout <branch> // переключиться на ветку

git switch third-branch // переключиться на ветку from v2.23.*

  

git branch // информация по всем веткам

  

git branch second-branch  // создание новой ветки

git checkout -b <branch> // создать ветку и переключиться на нее

git switch -c fourth-branch // создать ветку и переключиться на нее from v2.23.*

  

git merge <branch> // добавить указанный бранч в текущий

  

Удаление

  

Файлы Рабочей каталога

git rm <file> // удалит файл из рабочего каталога

git add <file> // зафиксирует факт удаления файла из черновой области

  

Неотслеживаемые изменения

git checkout .|<file> // отменить изменения в отслеживаемых файлах

git restore .|<file> // from v2.23.*

  

git clean -dn // вывести неотслеживаемые файлы которые удалим

git clean -df // удалить неотслеживаемые файлы

  

Черновая область

git reset <file> // удалить файлы 

git checkout <file> // из черновой области

  

git restore --staged .|<file> // from v2.23.*

git checkout <file>

  

Удалить комиты

git reset HEAD~1 // отменить последний коммит, и удалить его изменения из черновой области, изменения останутся в рабочей области

git reset --soft HEAD~1 // отменить последний коммит, его изменения останутся в рабочей директории и черновой области

git reset --hard HEAD~1 // отменить последний коммит, удалить изменения из рабочей директории и черновой области

  

Удалить ветку

git branch -d <branch> // удалить ветку если ее уже слили

git branch -D <branch> // удалить ветку независимо от того была она слита или нет

  

Можно сделать коммит в состоянии отсоединенного указателя HEAD, но потом нужно создать ветку в которую добавим этот коммит:

git branch <new_branch> // тут будет новый коммит

git switch master

git merge <new_branch>

  

Если после коммита в состоянии отсоединенного указателя HEAD мы переключились на другой бранч, то нужно сохранить коммит в новой ветке, чтобы не потерять его

git branch <new_branch> <hash_commit>

git switch master

git merge <new_branch>

  

.gitignore

*.log // игнорировать все файлы заканчивающиеся на .log

!test.log // не игнорировать файл test.log

web-app/* // игнорировать все файлы в папке

  

git stash // сохраняет неотслеживаемые и незакоммиченные во временное хранилище

git stash push -m 'message' // сохраняет неподготовленные изменения с сообщением

git stash pop <num> // получить изменения спрятанные под номером num и удалить их из стеша

git stash drop <num> // удалить запись в стеше с индексом num

git stash clear // очистить стеш

git stash list // вывод всех спрятанных изменений

git stash apply // получить последние спрятанные неподготовленные изменения

git stash apply <num> // получить изменения спрятанные под номером num

  

git reflog // список всех изменений в проекте, включая удаленные коммиты (хранятся 14 дней)

git reset --hard <hash> // вернуть удаленный коммит

  

git checkout <hash> // переключились на коммит с удаленной ветки

git switch -c <branch> // создали ветку

  

git merge <branch> // если в основной ветке после ветвления не было новых коммитов то применяется fast-forward слияние (просто перемещает указатель)

git merge --no-ff <branch> // рекурсивное слияние (моздает merge-коммит)

git merge --squash <branch> // все изменения с другой ветки добавит в черновую область текущей ветки

git commit -m 'merge'

  

git rebase <branch> // изменит родительский коммит для коммитов другой ветки (изменит hash существующих коммитов)

git switch master

git merge <branch> // слить изменения в мастер

  

git merge --abort // если хотим отменить слияние при конфликтах

git log --merge // покажет конфликтующие коммиты

git diff // сравнит конфликтующие коммиты

  

git cherry-pick <hash> // копирует коммит (с новым hash) в текущую ветку

  

git tag // вывести все теги

git tag 1.0 <hash> // создание легковесного тега

git tag -a <tag> -m "message" // создание аннотированного тега

git show <tag> // вывести коммит на который указывает тег

git checkout <tag> // сместить указатель на коммит с указанным тегом

git tag -d <tag> // удалить тег

  

git remote add origin URL // создаем соединение с удаленным репозиторием, где origin - алиас ссылки на удаленный репозиторий, URL - адрес репозитория на сервере

git push origin <branch> // отправляем локальные изменения в удаленный репозиторий и создадим там новую ветку

git push // переносит локальные данные в облако

git pull // получить изменения из удаленного репозитория (комбинация fetch+merge)

  

git branch // показать все ветки

git branch -a // локально и удаленно отслеживаемые ветки

git branch -r // удаленно отслеживаемые ветки

git branch -vv // список локально отслеживаемых и удаленных веток

  

git ls-remote // вывести список удаленных веток

git fetch origin // обновляет удаленно отслеживаемые ветки

  

git branch --track <branch> origin/<branch> // создание локально отслеживаемой ветки (связанной с удаленно отслеживаемой веткой)

  

git remote // показывает удаленные серверы

git remote show origin // показывает детальную конфигурацию

  

git clone <link > // копирование удаленного репозитория

  

git push origin <branch> // пуш ветки в удаленный репозиторий

git push -u origin <branch> // пуш ветки в удаленный репозиторий в режиме upstream (создает связь удаленно отслеживаемой ветки с локально отслеживаемой веткой)

  

git branch --delete --remotes origin/<branch> // удалить удаленно отслеживаемую ветку

git push origin --delete <branch> // удалить удаленную ветку (при удалении удаленной ветки удаленно отслеживаемая ветка тоже удаляется)

```

# Gitlab
```
.gitlab-ci.yml

image: golang:latest

stages:
 - build
 - test
 - deploy

prep:
 stage: build
 script:
   - echo "preparation"

feedback:
 image: golang:latest
 stage: privat
 script:
   - echo "How is it"
 except:
   - /^lover-*$/

install_gentoo:
 stage: privat
 script:
   - echo "installed"
 when: on_failure
 only:
   - main
   - /^lover-*$/
```
