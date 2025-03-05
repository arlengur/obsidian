### K8s: https://dashboard.testing.qiwi.com/

1. Install and Set Up kubectl on macOS: https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/ 
- Это клиент для k8s
- check: `kubectl version --client`
2. Install qubectl https://wiki.qiwi.com/pages/viewpage.action?pageId=162425225
- Это обертка qiwi, чтобы удобнее переключаться и авторизоваться
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
	- https://github.com/jqlang/jq/releases
- Команды
	- qubectl login - получить токены по своей доменной учетке
	- cubectl sw - переключение кластеров
- Копирование: 
	- `qubectl -n ao-ads cp --retries=-1 ao-ads-marmot-day-7965478b66-wx4ck:/opt/user-files/resources/marmot_day.tsv.gz ~/Downloads/marmot_day.tsv.gz`
