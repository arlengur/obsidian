Сброс параметров контроллера управления системой (SMC)
- Одновременно нажмите клавиши Shift, Control и Option и, не отпуская их, нажмите кнопку питания.
- Отпустите клавиши и еще раз нажмите кнопку питания, чтобы включить ноутбук Mac.

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

## Block sleep mode
```
caffeinate -u -t 3600 // time in sec / an hour
```

# Base64
```
base64 <<< string
base64 -i filename

base64 -d <<< string
```
## Path 
.zshrc

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

## Show the full Path in the Finder Title Bar
- Launch Terminal
- Enter the following command:
```
defaults write com.apple.finder _FXShowPosixPathInTitle -bool true; killall Finder
```
- Reverting the changes:
```
defaults write com.apple.finder _FXShowPosixPathInTitle -bool NO;
```

## Run program from everywhere
```
/usr/local/bin
```

## Store program for all
```
/opt or /usr/local
```

## Создать символическую ссылку
```
ln -s <откуда> <куда> (sudo ln -s /usr/local/sbt/bin/sbt /usr/local/bin/sbt)
```

## Переместить папку
```
mv <откуда> <куда> (sudo mv ~/Downloads/test /opt)
```

## Filename extensions
```
Finder > Settings, then click Advanced. Select or deselect “Show all filename extensions.”
```

## Заметки
```
– opt+-
— opt+shift+-

Mobile File Transfer
https://www.android.com/filetransfer/
```

Расположение приложения
```bash
which ls
```

# Tgz
```
tar -czf LotsOfFiles.tgz LotsOfFiles // create tgz

tar -zxf LotsOfFiles.tgz // unpack tgz
```

Найти JAVA_HOME
```
java -XshowSettings:properties -version 2>&1 > /dev/null | grep 'java.home'
```