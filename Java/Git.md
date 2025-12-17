## 1. Устройство коммитов, веток, merge

- **Коммит** — это объект, содержащий снимок дерева файлов, метаданные и ссылку на родительский коммит. Он идентифицируется SHA-1-хешем [Википедия](https://ru.wikipedia.org/wiki/Git?utm_source=chatgpt.com).
    
- **Ветка** — это всего лишь указатель (именованная ссылка) на коммит [Википедия](https://ru.wikipedia.org/wiki/Git?utm_source=chatgpt.com).
    
- **Merge**:
    
    - **Явное слияние** создаёт _merge-коммит_ с двумя родительскими коммитами: из текущей и целевой ветки [smartiqa.ru](https://smartiqa.ru/courses/git/lesson-5?utm_source=chatgpt.com)[git-scm.com](https://git-scm.com/book/ru/v2/%D0%92%D0%B5%D1%82%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B2-Git-%D0%9E%D1%81%D0%BD%D0%BE%D0%B2%D1%8B-%D0%B2%D0%B5%D1%82%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F-%D0%B8-%D1%81%D0%BB%D0%B8%D1%8F%D0%BD%D0%B8%D1%8F?utm_source=chatgpt.com)[atlassian.com](https://www.atlassian.com/ru/git/tutorials/using-branches/git-merge?utm_source=chatgpt.com).
        
    - **Fast-forward merge** просто сдвигает указатель ветки, если история линейна (никаких новых коммитов между ветками) [smartiqa.ru](https://smartiqa.ru/courses/git/lesson-5?utm_source=chatgpt.com)[atlassian.com](https://www.atlassian.com/ru/git/tutorials/using-branches/git-merge?utm_source=chatgpt.com).
        

---

## 2. Разница между `git fetch` и `git pull`

- `git fetch` загружает данные с удалённого репозитория и обновляет локальный индекс веток (например, `origin/main`), **не изменяя** текущую рабочую ветку [Википедия](https://ru.wikipedia.org/wiki/Git?utm_source=chatgpt.com)[timeweb.cloud](https://timeweb.cloud/tutorials/git/git-fetch-i-git-pull-chem-otlichayutsya-i-kak-rabotayut?utm_source=chatgpt.com)[selectel.ru](https://selectel.ru/blog/tutorials/git-fetch-command-how-is-it-different-from-git-pull/?utm_source=chatgpt.com).
    
- `git pull` — это shorthand для `git fetch` + `git merge` (или `--rebase`). Он сразу подтягивает и объединяет изменения в текущую ветку, что может вызвать конфликты [GeeksforGeeks](https://www.geeksforgeeks.org/git/git-difference-between-git-fetch-and-git-pull/?utm_source=chatgpt.com)[about.gitlab.com](https://about.gitlab.com/blog/git-pull-vs-git-fetch-whats-the-difference/?utm_source=chatgpt.com)[timeweb.cloud](https://timeweb.cloud/tutorials/git/git-fetch-i-git-pull-chem-otlichayutsya-i-kak-rabotayut?utm_source=chatgpt.com).
    

---

## 3. Что такое `rebase` и когда его использовать

- `git rebase` переписывает историю, перенося ваши коммиты на вершину целевой ветки — получаем более линейный и чистый лог без merge-коммитов [git-scm.com](https://git-scm.com/book/ru/v2/%D0%92%D0%B5%D1%82%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B2-Git-%D0%9E%D1%81%D0%BD%D0%BE%D0%B2%D1%8B-%D0%B2%D0%B5%D1%82%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F-%D0%B8-%D1%81%D0%BB%D0%B8%D1%8F%D0%BD%D0%B8%D1%8F?utm_source=chatgpt.com)[struchkov.dev](https://struchkov.dev/blog/ru/git-branches-merge-rebase/?utm_source=chatgpt.com)[Habr](https://habr.com/ru/companies/yandex_praktikum/articles/728302/?utm_source=chatgpt.com).
    
- **Когда использовать**:
    
    - Когда хотите сохранить чистую линейную историю.
        
    - Часто применяется при обновлении feature-ветки из `main` перед мёржем.
        

---

## 4. Откат изменений: `revert`, `reset`

- **`git reset`**:
    
    - Перемещает указатель `HEAD` — по сути, отменяет коммиты в локальной истории.
        
    - Опции:
        
        - `--soft`: оставляет изменения в индексе;
            
        - `--mixed` (по умолчанию): снимает индексацию, оставляя изменения в рабочем каталоге;
            
        - `--hard`: полностью возвращает состояние к указанному коммиту, включая файлы [timeweb.cloud](https://timeweb.cloud/tutorials/git/kak-otmenit-izmeneniya-v-git?utm_source=chatgpt.com)[cloud.ru](https://cloud.ru/blog/kak-primenyat-git-reset-i-git-revert?utm_source=chatgpt.com).
            
    - Подходит для локальных, необратимых изменений.
        
- **`git revert`**:
    
    - Создаёт новый коммит, который **отменяет** указанный ранее коммит, не изменяя историю [atlassian.com](https://www.atlassian.com/git/tutorials/undoing-changes/git-revert?utm_source=chatgpt.com)[FoxmindEd](https://foxminded.ua/ru/git-revert-chto-delaet/?utm_source=chatgpt.com).
        
    - Безопасен для использования в общих (pulled/pushed) ветках.
        

---

## 5. Как решать конфликты при слиянии

1. При `git merge` Git останавливается и отмечает файлы с конфликтами [atlassian.com](https://www.atlassian.com/ru/git/tutorials/using-branches/merge-conflicts?utm_source=chatgpt.com)[learn.microsoft.com](https://learn.microsoft.com/ru-ru/azure/devops/repos/git/merging?view=azure-devops&utm_source=chatgpt.com).
    
2. Используйте `git status`, чтобы увидеть список конфликтующих файлов, затем откройте их в редакторе и найдите маркеры:
    
    markdown
    
    КопироватьРедактировать
    
    `<<<<<<< HEAD ваша версия ======= версия из другой ветки >>>>>>> other-branch`
    
3. Разрешите конфликт вручную — оставьте нужный код и удалите маркеры [GitHub Docs](https://docs.github.com/ru/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line?utm_source=chatgpt.com)[Merion Academy](https://wiki.merionet.ru/articles/kak-ulavlivat-i-razresat-konflikty-v-git?utm_source=chatgpt.com).
    
4. Добавьте файл в индексацию: `git add <file>`.
    
5. Завершите мёрдж:
    
    - `git commit` — если мёрдж не в автоматическом режиме;
        
    - или `git merge --continue` / `git merge --abort` — если используется Git с поддержкой мердж-флоу [learn.microsoft.com](https://learn.microsoft.com/ru-ru/azure/devops/repos/git/merging?view=azure-devops&utm_source=chatgpt.com)[githowto.com](https://githowto.com/ru/resolving_conflicts?utm_source=chatgpt.com).
        
6. Для облегчения процесса можно использовать `git mergetool` + визуальные дифф-инструменты (например, `meld`) [Habr](https://habr.com/ru/articles/323234/?utm_source=chatgpt.com).
    

---

### Резюме:

| Пункт                 | Описание                                                               |
| --------------------- | ---------------------------------------------------------------------- |
| Коммиты, ветки, merge | Объекты, указатели, явное/fast-forward слияние                         |
| Fetch vs Pull         | Fetch безопасно загружает, Pull делает автоматически merge             |
| Rebase                | Линейное переписывание истории, cleaner log                            |
| Revert vs Reset       | Revert — безопасный отменяющий коммит; Reset — перенос ветки           |
| Конфликты merge       | Ручное разрешение, использование mergetool, `--continue/--abort` опции |
## 📝 Шпаргалка по Git

### 1. Коммиты, ветки, merge

bash

Копировать/Редактировать

`# Создать новый коммит git add file.txt git commit -m "Сообщение коммита"  # Создать ветку и переключиться git branch feature git checkout feature      # старый способ git switch feature        # новый способ  # Слияние (merge) git merge feature         # из main: git merge feature`

**Ключевые моменты:**

- Коммит = снимок файлов + ссылка на родителя.
    
- Ветка = просто указатель на коммит.
    
- Merge бывает:
    
    - **Fast-forward** — просто двигает указатель.
        
    - **Merge commit** — создаёт коммит с двумя родителями.
        

---

### 2. Fetch vs Pull

bash

КопироватьРедактировать

`git fetch origin main   # обновляет только ссылки (origin/main) git pull origin main    # fetch + merge в текущую ветку`

- **Fetch** — подтягивает изменения, но не трогает рабочую ветку.
    
- **Pull** — подтягивает и сразу сливает.
    

---

### 3. Rebase

bash

КопироватьРедактировать

`# Перенос своей ветки на main git checkout feature git fetch origin git rebase origin/main`

- Держит историю линейной.
    
- Удобно перед merge в main.
    
- **Не делай rebase веток, которые уже пушил в общий репозиторий.**
    

---

### 4. Откаты (reset, revert)

bash

КопироватьРедактировать

`# reset (отмена локально) git reset --soft <commit>   # оставить в индексе git reset --mixed <commit>  # по умолчанию git reset --hard <commit>   # удалить все изменения  # revert (отмена с сохранением истории) git revert <commit>`

- **Reset** — переписывает историю (локально).
    
- **Revert** — создаёт новый коммит, отменяющий изменения.
    

---

### 5. Решение конфликтов merge

bash

КопироватьРедактировать

`git merge feature # Конфликты в файлах: # <<<<<<< HEAD # ваш код # ======= # код из feature # >>>>>>> feature  # Редактируем, сохраняем: git add file.txt git merge --continue`

- Конфликты появляются, если изменения пересекаются.
    
- Разрешаем вручную или через `git mergetool`.
    

---

### 🔥 Быстрые команды для собеседования

bash

КопироватьРедактировать

`# Создание ветки и переход в неё git checkout -b mybranch  # Слияние без merge-коммита (fast-forward) git merge --ff-only branch  # Откат до состояния удалённой ветки git reset --hard origin/main  # Обновить feature ветку через rebase git pull --rebase origin main  # Отменить последний коммит, но оставить изменения git reset --soft HEAD~1`