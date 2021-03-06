# Руководство по работе с GIT

## Начальная настройка GIT

Необходимо указать наше имя пользователя и адрес электронной почты:

```
    git config --global user.name "My Name"
    git config --global user.email myEmail@example.com
```

Для того, чтобы посмотреть все настройки системы, используйте команду:

```
    git config --list
```

## Создание и инициализация нового репозитория

Для создания и инициализации нового репозитория введите команды:

```
    mkdir Desktop/git_exercise/
    cd Desktop/git_exercise/
    git init
```

## Как сделать коммит

Для сохранения изменений, их необходимо закоммитить. Но сначала, мы должны обозначить эти файлы для Гита, при помощи команды git add, добавляющей (или подготавливающей) их к коммиту. Добавлять их можно по отдельности:

```
    git add <имя файла>
```

Теперь создадим непосредственно сам коммит:

```
    git commit -m 'Add some code'
```

## Как посмотреть коммиты

Для просмотра все выполненных фиксаций можно воспользоваться историей коммитов. Она содержит сведения о каждом проведенном коммите проекта. Запросить ее можно при помощи команды:

```
    git log
```

Если вдруг нам нужно переделать commit message и внести туда новый комментарий, можно написать следующую конструкцию:

```
    git commit --amend -m 'Новый комментарий'
```

## Настройка .gitignore

В большинстве проектов есть файлы или целые директории, в которые мы не хотим (и, скорее всего, не захотим) коммитить. Мы можем удостовериться, что они случайно не попадут в git add -A при помощи файла .gitignore

1. Создайте вручную файл под названием .gitignore и сохраните его в директорию проекта.
2. Внутри файла перечислите названия файлов/папок, которые нужно игнорировать, каждый с новой строки.
3. Файл .gitignore должен быть добавлен, закоммичен и отправлен на сервер, как любой другой файл в проекте.

## Ветвление

Во время разработки новой функциональности считается хорошей практикой работать с копией оригинального проекта, которую называют веткой. Ветви имеют свою собственную историю и изолированные друг от друга изменения до тех пор, пока вы не решаете слить изменения вместе. Это происходит по набору причин:

* Уже рабочая, стабильная версия кода сохраняется.
* Различные новые функции могут разрабатываться параллельно разными программистами.
* Разработчики могут работать с собственными ветками без риска, что кодовая база поменяется из-за чужих изменений.
* В случае сомнений, различные реализации одной и той же идеи могут быть разработаны в разных ветках и затем сравниваться.

### 1. Создание новой ветки

Основная ветка в каждом репозитории называется master. Чтобы создать еще одну ветку, используем команду branch <name>:

```
    git branch drafts
```

Это создаст новую ветку, пока что точную копию ветки master.

### 2. Переключение между ветками

Сейчас, если мы запустим branch, мы увидим две доступные опции:

```
    git branch
        drafts
        * master
``` 

master — это активная ветка, она помечена звездочкой. Но мы хотим работать с нашей “drafts”, так что нам понадобится переключиться на другую ветку. Для этого воспользуемся командой checkout, она принимает один параметр — имя ветки, на которую необходимо переключиться.

```
    git checkout drafts
``` 

### 3. Слияние веток

Наши "черновики" будут текстовым файлом под названием drafts.txt. Мы создадим его, добавим и закоммитим:

```
    git add drafts.txt
    git commit -m "New drafts”
```

Изменения завершены, теперь мы можем переключиться обратно на ветку master.

```
    git checkout master
```

Теперь, если мы откроем наш проект в файловом менеджере, мы не увидим файла drafts.txt, потому что мы переключились обратно на ветку master, в которой такого файла не существует. Чтобы он появился, нужно воспользоваться merge для объединения веток (применения изменений из ветки amazing_new_feature к основной версии проекта).

```
    git merge drafts
```

Теперь ветка master актуальна. Ветка drafts больше не нужна, и ее можно удалить.

```
    git branch -d drafts
```

### 4. Слияние веток через Rebase

Rebase — один из двух способов объединить изменения, сделанные в одной ветке, с другой веткой. Начинающие и даже опытные пользователи git иногда испытывают нежелание пользоваться ей, так как не видят смысла осваивать еще один способ объединять изменения, когда уже и так прекрасно владеют операцией merge. В этой статье я бы хотел подробно разобрать теорию и практику использования rebase.

#### Теория

Итак, освежим теоретические знания о том, что же такое rebase. Для начала вкратце — у вас есть две ветки — master и feature, обе локальные, feature была создана от master в состоянии A и содержит в себе коммиты C, D и E. В ветку master после отделения от нее ветки feature был сделан 1 коммит B.

![Схема слияния по Rebase](/Pictures/d15f9c605c1701890cdd8a9b3a1f9d80.png)

После применения операции rebase master в ветке feature, дерево коммитов будет иметь вид:

![Схема слияния по Rebase](/Pictures/0259c0a2acf089365cc677c6c2824473.png)

Обратите внимание, что коммиты C', D' и E' — не равны C, D и E, они имеют другие хеши, но изменения (дельты), которые они в себе несут, в идеале точно такие же. Отличие в коммитах обусловлено тем, что они имеют другую базу (в первом случае — A, во втором — B), отличия в дельтах, если они есть, обусловлены разрешением конфликтных ситуаций, возникших при rebase. Об этом чуть подробнее далее.

Такое состояние имеет одно важное преимущество перед первым, при слиянии ветки feature в master ветка может быть объединена по fast-forward, что исключает возникновение конфликтов при выполнении этой операции, кроме того, код в ветке feature более актуален, так как учитывает изменения сделанные в ветке master в коммите B.

#### Процесс rebase-а детально

Давайте теперь разберемся с механикой этого процесса, как именно дерево 1 превратилось в дерево 2?

Напомню, перед rebase вы находтесь в ветке feature, то есть ваш HEAD смотрит на указатель feature, который в свою очередь смотрит на коммит E. Идентификатор ветки master вы передаете в команду как аргумент:

```
    git rebase master
```

Для начала git находит базовый коммит — общий родитель этих двух состояний. В данном случае это коммит A. Далее двигаясь в направлении вашего текущего HEAD git вычисляет разницу для каждой пары коммитов, на первом шаге между A и С, назовем ее ΔAC. Эта дельта применяется к текущему состоянию ветки master. Если при этом не возникает конфликтное состояние, создается коммит C', таким образом C' = B + ΔAC. Ветки master и feature при этом не смещаются, однако, HEAD перемещается на новый коммит (C'), приводя ваш репозитарий состояние «отделеной головы» (detached HEAD).

![Схема слияния по Rebase](/Pictures/6c3d10f335d1612aba10fdecfc6ee291.png)

Успешно создав коммит C', git переходит к переносу следующих изменений — ΔCD. Предположим, что при наложении этих изменний на коммит C' возник конфликт. Процесс rebase останавливается (именно в этот момент, набрав git status вы можете обнаружить, что находитесь в состоянии detached HEAD). Изменения, внесенные ΔCD находятся в вашей рабочей копии и (кроме конфликтных) подготовлены к коммиту (пунктиром обозначена stage-зона):

![Схема слияния по Rebase](/Pictures/ad5e57930d4d4e490017b2b17cb23e84.png)

Далее вы можете предпринять следующие шаги:

1. Отменить процесс rebase набрав в консоли

```
    git rebase --abort
```

При этом маркер HEAD, будет перенесен обратно на ветку feature, а уже добавленные коммиты повиснут в воздухе (на них не будет указывать ни один указатель) и будут вскоре удалены.

2. Разрешить конфликт в вашем любимом merge-tool'е, подготовить файлы к коммиту, набрав git add %filename%. Проделав это со всеми конфликтными файлами, продолжить процесс rebase-а набрав в консоли

```
    git rebase --continue 
```

При этом, если все конфликты действительно разрешены, будет создан коммит D' и rebase перейдет к следующему, в данном примере последнему шагу.

3. Если изменения, сделанные при формировании коммита B и коммита D являются полностью взаимоисключающими, причем «правильные» изменения сделаны в коммите B, то вы не сможете продолжить набрав git rebase --continue, так как разрешив конфликты обнаружите, что изменений в рабочей копии нет. В данном случае вам надо пропустить создание коммита D', набрав команду

```
    git rebase --skip
```

После применения изменений ΔDE будет создан последний коммит E', указатель ветки feature будет установлен на коммит E', а HEAD станет показывать на ветку feature — теперь, вы находитесь в состоянии на втором рисунке, rebase окончен. Старые коммиты C, D и E вам больше не нужны.

![Схема слияния по Rebase](/Pictures/36fb32fc0cd649665022df80fb00dcf5.png)

При этом коммиты, созданные в процессе rebase-а, будут содержать данные как об оригинальном авторе и дате изменений (Author), так и о пользователе, сделавшем rebase.

## Работа с GitHub

Для начала работы необходимо зарегистрировать на
```
    https://github.com/
```
К примеру создадим репозиторий "geek-git-tutorial"

Проделайте следующие операции у себя на компьютере(инструкция взята с github.com):

    * или создайте новый репозиторий в командной строке 

    ```
        echo "# geek-git-tutorial" >> README.md
        git init
        git add README.md
        git commit -m "first commit"
        git branch -M main
        git remote add origin https://github.com/zversko/geek-git-tutorial.git
        git push -u origin main
    ```

    * или поместите существующий репозиторий из командной строки

    ```
        git remote add origin https://github.com/zversko/geek-git-tutorial.git
        git branch -M main
        git push -u origin main
    ```

### Для примера попробуем поработать с готовым репозиторием и внести в нем изменения.

Заходим в интересующий нас репозиторий на Github нажимаем на зеленую кнопку "Code" где копируем URL ссылку на нужный нам репозиторий.

![Кнопка Code](/Pictures/button_code.png)

Создаем необходимую папку для клонирования репозитория, заходим в эту папку.

```
    mkdir <название_папки>
    cd <название_папки>
```

Затем выполняем команду в терминале (или командной строке Windows):

```
    git clone <вставляем_URL>
```

Ветка по умолчанию — master(или main). Чтобы изменениями было проще управлять и они не смешивались друг с другом, создадим отдельную ветку, где и будем работать. При этом ветку стоит назвать так, чтобы имя говорило о её назначении.

```
    git checkout -b new-feature
```

Теперь приступаем к работе. Редактируем код, обновляем документацию. Эти изменения мы коммитим в нашу ветку.

```
    git add <имя файла>
    git commit -m 'Add some code'
```

После чего мы осуществляем отправку ветки обратно на GitHub

```
    git push
```

Так же в процессе с работой в команде не забываем делать синхронизацию с удаленным репозиторием

```
    git pull
```