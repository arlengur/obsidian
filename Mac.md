## nc / netcat
```
nc localhost 8888
echo -n "GET /hello/A HTTP/1.0\r\n\r\n" | nc localhost 8080
```

## Block sleep mode
```
caffeinate -u -t 3600 // time in sec / an hour
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

## Check port usage
```
lsof -w -n -i tcp:8080
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