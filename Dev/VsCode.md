# Shortcuts
Cmd+X - cut line
Cmd+Shift+K - remove line
Cmd+B - Скрыть/развернуть сайдбар
Cmd+~ - Скрыть/развернуть терминал
Ctrl+` - open terminal
Cmd+P - Переход к файлу
Cmd+| - Открыть контекстное меню во время ввода
Cmd+/ - comment
Shift+Opt+A - block comment
Opt+ArrowUp - move line up
Cmd+Shift+P - command line
Cmd+K+T - choose color theme
Cmd+K, then O (without holding CMD) - open new VS Code window

# Commands
Cmd+Shift+P - exec command (zen, minimap, markdown,theme, snippet)
Cmd+P - find file
Cmd+B - open/close side bar
Cmd+D - add selection to next find much
Cmd+~ - open/close terminal
Cmd+K - clear terminal
Ctrl+Tab - switch open file
Cmd+W - close file
CMD + Ctrl + space // add emoji
Cmd+Shift+L - select all occurrences of find match

Replace all occurrences
- Highlight the word.
- Use the Cmd + Shift + L keyboard shortcut on macOS.
- Start typing to replace the selected text.

# Launching from the command line
- Launch VS Code.
- Open the Command Palette (Cmd+Shift+P) and type 'shell command' to find the Shell Command: Install 'code' command in PATH command.

Alternative path:
- cat << EOF >> ~/.zprofile
- Add Visual Studio Code (code)
- export PATH="\$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
- EOF

# Extensions
Reload - Add reload button to status bar right bottom
Error Lens - Improve highlighting of errors, warnings and other language diagnostics
Diasable ligatures - Disable ligatures at the cursor position, or disable all ligatures on the line
Code Spell Checker (проверка орфографии) - https://marketplace.visualstudio.com/...
Russian - Code Spell Checker - https://marketplace.visualstudio.com/...
Color Highlight (подсветка синтаксиса html, css) - https://marketplace.visualstudio.com/...
CSS Peek (навигация из html в css) - https://marketplace.visualstudio.com/...
DotENV (описание переменных) - https://marketplace.visualstudio.com/...
ESLint (советы по синтаксису js) - https://marketplace.visualstudio.com/...
stylelint (советы по синтаксису css, вкладка problems)
GitLens — Git supercharged - https://marketplace.visualstudio.com/...
JavaScript (ES6) code snipets
npm (обновление плагинов) - https://marketplace.visualstudio.com/...
Open in browser (открыть html в браузере)
Path Intellisense (подсказки при импорте или автоимпорт) - https://marketplace.visualstudio.com/...

Prettier (Code formatter) https://marketplace.visualstudio.com/…
можно создать свой конфиг файл .prettierrc с описанием
{
  "singleQuote": true,
  "arrowParens": "avoid"
}


Auto Close Tag - https://marketplace.visualstudio.com/...
Auto Rename Tag - https://marketplace.visualstudio.com/...
Bracket pair colorizer
Live server (локальный сервер)
Import cost (показывает размер импортируемой библиотеки)
Settings Sync
Code Runner Extension (ctrl+opt+N, ctrl+opt+M)
YAML - поддержка YAML
Docker - для работы с докером

Темы:
One Dark Pro (тема оформления) - https://marketplace.visualstudio.com/...
Material Theme (тема оформления)
vscode-icons / Material Icon Theme (иконки для дерева проекта) - https://marketplace.visualstudio.com/...

Built-in:
Emmet ()


Конфиги:
keybindings.json

Настройки - settings.json
```json
{
    "editor.detectIndentation": true,
    "editor.fontSize": 14,
    "editor.formatOnPaste": true,
    "editor.formatOnSave": true,
    "editor.formatOnSaveMode": "file",
    "editor.formatOnType": true,
    "editor.indentSize": 2,
    "editor.insertSpaces": true,
    "editor.renderWhitespace": "trailing",
    "editor.rulers": [
        100
    ],
    "editor.snippetSuggestions": "bottom",
    "editor.stickyScroll.enabled": true,
    "editor.stickyScroll.maxLineCount": 10,
    "editor.tabSize": 2,
    "editor.useTabStops": false,
    "files.autoSave": "onFocusChange",
    "files.encoding": "utf8",
    "files.eol": "\n",
    "files.insertFinalNewline": true,
    "files.trimFinalNewlines": true,
    "files.trimTrailingWhitespace": true,
    "files.watcherExclude": {
        "**/.ammonite": true,
        "**/.bloop": true,
        "**/.bsp": true,
        "**/.dotty-ide": true,
        "**/.g8": true,
        "**/.history": true,
        "**/.idea": true,
        "**/.metals": true,
        "**/.scala-build": true,
        "**/.scala": true,
        "**/metals.sbt": true,
        "**/target": true
    },
    "git.autofetch": false,
    "githubPullRequests.pullBranch": "never",
    "metals.enableIndentOnPaste": true,
    "metals.enableSemanticHighlighting": true,
    "metals.superMethodLensesEnabled": false,
    "multiCommand.commands": [
        {
            "command": "multiCommand.toggleInfo",
            "sequence": [
                "metals.toggle-implicit-conversions-and-classes",
                "metals.toggle-implicit-parameters",
                "metals.toggle-show-inferred-type"
            ]
        }
    ],
    "problems.showCurrentInStatus": true,
    "search.exclude": {
        "**/.ammonite": true,
        "**/.bloop": true,
        "**/.bsp": true,
        "**/.dotty-ide": true,
        "**/.g8": true,
        "**/.history": true,
        "**/.idea": true,
        "**/.metals": true,
        "**/.scala-build": true,
        "**/.scala": true,
        "**/metals.sbt": true,
        "**/target": true,
    },
    "workbench.editor.highlightModifiedTabs": true,
}
```

# Snippets
Использование snippets: Cmd+Shift+P ввести snippet и выбрать new global snippets file, придумать имя файла и сделать snippet:
Ex snippet (но с $1 не работает автокомплит):
```
  "Print to console": {
    "scope": "javascript,typescript",
    "prefix": "cl",
    "body": ["console.log($1);"],
    "description": "Log output to console"
  }
```
