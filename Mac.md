# Check IP
- curl ipinfo.io/ip 
- curl icanhazip.com

На запись в файл влияют параметры:
checkRebalanceTime если истекло, то пишем в файл
TimeOut если истекло, то пишем в файл
batchCount если прывысиил, то пишем в файл

```

# Shortcuts
– `opt+-`
— `opt+shift+-`
Блокировка - `cmd+ctrl+q`

Сброс параметров контроллера управления системой (SMC)
- Одновременно нажмите клавиши Shift, Control и Option и, не отпуская их, нажмите кнопку питания.
- Отпустите клавиши и еще раз нажмите кнопку питания, чтобы включить ноутбук Mac.

# Curl
`curl -O <url>` Работает также как `wget <url>` поэтому можно в .zprofile (или .bashrc) добавить

```bash
alias wget='curl -O'
```


cd (change directory) переход в директорию
```
cd folder
```
# Shortcuts
```
Ctrl+A - jump to the beginning of a line inside the OS X terminal
Ctrl+E - jump to the end of a line inside the OS X terminal
```

## Update shortcuts
System Settings - Keyboard - Spotlight
# Мониторинг системы
```
top -o cpu
```
## nc / netcat
```
nc localhost 8888
echo -n "GET /p/1 HTTP/1.0\r\n\r\n" | nc localhost 9000
```
## Check port usage
```
lsof -w -n -i tcp:8080
```

## Caffeinate
Command to prevent sleep.
Time in sec (an hour).
```
caffeinate -u -t 3600
```

# Base64
```
base64 <<< string
base64 -i filename

base64 -d <<< string
```

# find location
```
where scala
```
# Path 
.zshrc

## Zsh promt

source ~/.zshrc - load from settings

Параметры
- %n - username
- %m - hostname
- %1~ - current directory
	- %~ - home directory
	- %/ - root directory
- %# - `#` with root privileges, `%` otherwise
- %(x.true-text.false-text) - ternary operator
	- %(?.✓.x)
	- %(!.#.&gt;) - shows the `#` if we're root, and `>` otherwise
- %? - exit code of previous command
	- %F{red}?%?)%f
- %F{colour} and `%f` - The `%F{colour}` will set the foreground colour, and `%f` will reset it back to the default. Ex. %(?.%F{blue}✓.%F{red}x)%f
	- there is 256 color pallet with `%F{0}` through `%F{255}` [colors](https://www.calmar.ws/vim/256-xterm-24bit-rgb-color-chart.html)
- `%B` `%b` - start/stop bold
	- %B%F{240}~%f%b

```
PROMPT='%(?.%F{14}✓.%F{9}x %?)%f %~ %(!.#.>)'
```

Example:
```
export JAVA_8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home
export JAVA_11_HOME=/Library/Java/JavaVirtualMachines/temurin-11.jdk/Contents/Home
export M2_HOME=/opt/apache-maven-3.8.4
export CATALINA_HOME=/opt/apache-tomcat-9.0.58
export PIP_HOME=/Users/agalin/Library/Python/2.7/bin

export PATH=$PATH:$M2_HOME/bin:$CATALINA_HOME/bin:$PIP_HOME

alias java8='export JAVA_HOME=$JAVA_8_HOME'
alias java11='export JAVA_HOME=$JAVA_11_HOME'
```

# Show the full Path in the Finder Title Bar
- Launch Terminal
- Enter the following command:
```
defaults write com.apple.finder _FXShowPosixPathInTitle -bool true; killall Finder
```
- Reverting the changes:
```
defaults write com.apple.finder _FXShowPosixPathInTitle -bool NO;
```

# Run program from everywhere
```
/usr/local/bin
```

# Store program for all
```
/opt or /usr/local
```

# Создать символическую ссылку
```
ln -s <откуда> <куда> (sudo ln -s /usr/local/sbt/bin/sbt /usr/local/bin/sbt)
```

# Переместить папку
```
mv <откуда> <куда> (sudo mv ~/Downloads/test /opt)
```

# Filename extensions
```
Finder > Settings, then click Advanced. Select or deselect “Show all filename extensions.”
```

# Заметки
```
Mobile File Transfer
https://www.android.com/filetransfer/
https://android.p2hp.com/filetransfer/index.html
```

Расположение приложения
```bash
which ls
```

# Tgz
```
tar -czf LotsOfFiles.tgz LotsOfFiles // create tgz

tar -zxf LotsOfFiles.tgz // unpack tgz

tar -xvzf community_images.tar.gz
```

Опции:
- f - последний в списке команд и tar файл должен идти за ним. Указывает имя и путь до файла
- z - распакавать архив используя g**z**ip
- x - говорит что файлы надо извлечь
- v - выводит подробный лог сообщений

# SSH
```
ssh-keygen # generate ssh key

cat ~/.ssh/id_rsa.pub | pbcopy # copy key to clipboard
```
## SSH to Connect to a Remote Server
`ssh user@machine`

# Java install
```zsh
curl https://download.java.net/java/ga/jdk11/openjdk-11_osx-x64_bin.tar.gz \
 | tar -xz \
 && sudo mv jdk-11.jdk /Library/Java/JavaVirtualMachines \
 && java -version

curl https://download.oracle.com/java/17/archive/jdk-17.0.12_macos-aarch64_bin.tar.gz \
| tar -xz \
&& sudo mv jdk-17.0.12.jdk /Library/Java/JavaVirtualMachines \
&& java -version
```

execute:
```zsh
echo "\nalias java11='export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home'" >>~/.zshrc
echo "\nalias java17='export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-17.0.12.jdk/Contents/Home'" >>~/.zshrc
```

Or add these lines to ~/.zshrc:
```zsh
alias java11='export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home'
alias java17='export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-17.0.12.jdk/Contents/Home'
```

Найти JAVA_HOME
```
java -XshowSettings:properties -version 2>&1 > /dev/null | grep 'java.home'
```

# Maven install
```zsh
curl https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz \
| tar -xz \
&& sudo mv apache-maven-3.9.9 /opt \
&& /opt/apache-maven-3.9.9/bin/mvn -version
```

Add these lines to ~/.zshrc:
```zsh
export PATH=$PATH:/opt/apache-maven-3.9.9/bin
```

# Scala install

Download scala https://www.scala-lang.org/download/
Unpack
Move to /opt
Add to PATH `export PATH=$PATH:/opt/scala-2.13.15/bin`
Check `scala -version`

## Uninstall
Remove folder `/Users/agalin/Library/Application Support/Coursier`
Clear coursier install directory from PATH

# Telnet
```
www.gnu.org/software/inetutils
Download "inetutils-1.9.4.tar.gz"
cd inetutils-1.9.4
./configure
make
sudo make install

telnet rainmaker.wunderground.com
telnet ao-ads-fraud-test.db.osmp.ru 5432
```

Username
```
id -F
id -un | whoami | echo $USER
```

Плеер https://iina.io/