
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

# find location
```
where scala
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

SSH
```
ssh-keygen # generate ssh key

cat ~/.ssh/id_rsa.pub | pbcopy # copy key to clipboard
```

# Java install
```
$ curl -C - https://download.java.net/java/ga/jdk11/openjdk-11_osx-x64_bin.tar.gz -O openjdk-11_osx-x64_bin.tar.gz
$ tar xf openjdk-11_osx-x64_bin.tar.gz
$ sudo mv jdk-11.jdk /Library/Java/JavaVirtualMachines/
$ java -version


curl https://download.java.net/java/ga/jdk11/openjdk-11_osx-x64_bin.tar.gz \
 | tar -xz \
 && sudo mv jdk-11.jdk /Library/Java/JavaVirtualMachines \
 && java -version

curl https://download.oracle.com/java/23/latest/jdk-23_macos-aarch64_bin.tar.gz | tar -xz \
 && sudo mv jdk-23.0.1.jdk /Library/Java/JavaVirtualMachines \
 && java -version

export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-17.0.12.jdk/Contents/Home
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-23.0.1.jdk/Contents/Home

alias java11='export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home'
alias java17='export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-17.0.12.jdk/Contents/Home'
alias java23='export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-23.0.1.jdk/Contents/Home'

Write aliases to profile:
echo alias java23=\"export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-23.0.1.jdk/Contents/Home\" >>~/.zprofile

Check installed java:
/usr/libexec/java_home -V

Check location sertain version:
/usr/libexec/java_home -v 17

Switch between several java: add function to ~/.zshrc:
jdk() {
    version=$1
    export JAVA_HOME=$(/usr/libexec/java_home -v"$version");
    java -version
}

and switch with: jdk 11
```
Unr9xtuh
```
# Scala install

https://www.scala-lang.org/download/

curl -fL https://github.com/VirtusLab/coursier-m1/releases/latest/download/cs-aarch64-apple-darwin.gz | gzip -d > cs && chmod +x cs && (xattr -d com.apple.quarantine cs || true) && ./cs setup

location scala & sbt:
/Users/agalin/Library/Application Support/Coursier/bin/scala
```

Telnet
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