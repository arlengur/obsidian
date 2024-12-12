1. Install and Set Up kubectl on macOS: https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/
2. Install qubectl https://wiki.qiwi.com/pages/viewpage.action?pageId=162425225
- Переменные среды лучше установить в 
	- `echo` `'PATH="$HOME/.kube:$PATH"'` `>>` `"$HOME/.zprofile"`
- Для этого потребуются программы
	- timeout
		- скачать исходники https://www.gnu.org/software/coreutils/#source
		- установить
			- test -f configure
			- ./bootstrap ./configure
			- make
			- make install prefix=/Users/a.galin/repo/coreu
	- jq
	- 
