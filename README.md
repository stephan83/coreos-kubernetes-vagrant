# CoreOS Kubernetes Vagrant

Vagrant script to experiment with CoreOS and Kubernetes.

- `shared.yml` = master and node cloud-config
- `master.yml` = master cloud-config, final config is shared.deep_merge(master)
- `node.yml` = node cloud-config, final config is shared.deep_merge(node)

By default, it creates one master (core-01) and three nodes (core-0[2..4]).

It installs `kubecfg` on the master.

Settings are in `Vagrantfile`.

Example:

		$ vagrant up

		...

		$ vagrant ssh core-01 -- -A

		$ kubecfg list minions
		Minion identifier
		----------
		172.17.8.102
		172.17.8.104
		172.17.8.103
