## Создать ключи
```
java -cp "generators.jar:/Users/agalin/repo/pki/test/*" com.wavesenterprise.generator.AccountsGeneratorApp accounts_pqc.conf
```
Пример accounts.conf:
```
accounts-generator {
	crypto.type = pqc
	chain-id = K
	amount = 3
	wallet = "keystore.dat"
	wallet-password = "sato"
	reload-node-wallet {
		enabled = false
		url = "http://localhost:6862/utils/reload-wallet"
	}
}
```
## Подписать генезис
```
java -cp "generators.jar:/Users/agalin/repo/pki/test/*" com.wavesenterprise.generator.GenesisBlockGenerator applications.conf
```
Пример node.conf:
```
node {
	crypto.type = PQC
	owner-address = "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv"

	ntp {
		fatal-timeout = "5 minutes"
	}
	directory = "/node"
	data-directory = "/node/data"
	wallet {
		file = "/node/keystore.dat"
		password = "sato"
	}
	blockchain {
		type = "CUSTOM"
		fees {
			enabled = false
		}
		consensus {
			type = "poa"
			round-duration = "17s"
			sync-duration = "3s"
			ban-duration-blocks = 100
			warnings-for-ban = 3
			max-bans-percentage = 40
		}
		custom {
			address-scheme-character = "K"
			functionality {
				feature-check-blocks-period = 1500
				blocks-for-feature-activation = 1000
				pre-activated-features {
					2 = 0
					3 = 0
					4 = 0
					5 = 0
					6 = 0
					7 = 0
					9 = 0
					10 = 0
					100 = 0
					101 = 0
				}
			}
		genesis {
			average-block-delay = "60s"
			initial-base-target = 153722867
			block-timestamp = 1690969500544
			initial-balance = 100000000000000
			genesis-public-key-base-58 = "3Q6p5qEAv8EhQsKJ3oacoNnY7m4bCJEUrTRbvtBGkNKb"
			signature = "3e4TAnhL4EjjAHq6nwi1G8ADENTvx7m1zxJqXk6T9J35t9H7YzbwB293jmhoknYr9BjjigLCFqQvzRcQT1uDtgNG"
			transactions = [
				{
					recipient = "3JMPZvdm6JHDCFHhuDfRVcWc7QoL8nSusVv"
					amount = 100000000000000
				}
			]
			network-participants = [
			{
			public-key = "3dnx34pAHDXKhavFRbafMuMo9oC7t7qjNT9UiuFTVBLf"
			roles = [
			"permissioner"
			"miner"
			"connection_manager"
			"contract_developer"
			"issuer"
			]
			}
			]
			}
		}
	}
	logging-level = "DEBUG"
	network {
		bind-address = "0.0.0.0"
		port = 6864
		known-peers = [
			"node-0:6864"
			"node-1:6864"
			"node-2:6864"
		]
		node-name = "node-0"
		peers-data-residence-time = "2h"
		declared-address = "0.0.0.0:6864"
		attempt-connection-delay = "5s"
	}
	miner {
		enable = "yes"
		quorum = 1
		interval-after-last-block-then-generation-is-allowed = "10d"
		micro-block-interval = "5s"
		min-micro-block-age = "3s"
		max-transactions-in-micro-block = 500
		minimal-block-generation-offset = "200ms"
	}
	rest-api {
		enable = "yes"
		bind-address = "0.0.0.0"
		port = 6862
		auth {
			type = "api-key"
			api-key-hash = "5M7C14rf3TAaWscd8fHvU6Kqo97iJFpvFwyQ3Q6vfztS"
			privacy-api-key-hash = "5M7C14rf3TAaWscd8fHvU6Kqo97iJFpvFwyQ3Q6vfztS"
		}
	}
	privacy {
		crawling-parallelism = 100
		storage {
			enabled = false
		}
	}
	docker-engine {
		enable = "yes"
		use-node-docker-host = "yes"
		default-registry-domain = "registry.wavesenterprise.com/waves-enterprise-public"
		docker-host = "unix:///var/run/docker.sock"
		execution-limits {
			timeout = "10s"
			memory = 512
			memory-swap = 0
		}
		reuse-containers = "yes"
		remove-container-after = "10m"
		remote-registries = []
		check-registry-auth-on-startup = "yes"
		contract-execution-messages-cache {
			expire-after = "60m"
			max-buffer-size = 10
			max-buffer-time = "100ms"
		}
	}
}
```