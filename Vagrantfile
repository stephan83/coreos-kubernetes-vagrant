require 'fileutils'
require 'open-uri'
require 'yaml'

Vagrant.require_version ">= 1.6.0"

SHARED_SRC_PATH = File.join(File.dirname(__FILE__), "shared.yml")
MASTER_SRC_PATH = File.join(File.dirname(__FILE__), "master.yml")
NODE_SRC_PATH = File.join(File.dirname(__FILE__), "node.yml")
MASTER_DST_PATH = File.join(File.dirname(__FILE__), "master.tmp.yml")
NODE_DST_PATH = File.join(File.dirname(__FILE__), "node.tmp.yml")

$num_instances = 2
$update_channel = "alpha"
#$expose_docker_tcp = 2375
#$expose_etcd_tcp = 4001
#$expose_http = 3000
$enable_serial_logging = false
$vb_gui = false
$vb_memory = 1024
$vb_cpus = 2

token = open('https://discovery.etcd.io/new').read

class ::Hash
    def deep_merge(second)
        merger = proc { |key, v1, v2|
          if Hash === v1 && Hash === v2
            v1.merge(v2, &merger)
          elsif Array === v1 && Array === v2
            v1 + v2
          else
            v2
          end
        }
        self.merge(second, &merger)
    end
end

shared = YAML.load(IO.readlines(SHARED_SRC_PATH)[1..-1].join)
master = YAML.load(IO.readlines(MASTER_SRC_PATH)[1..-1].join)
node = YAML.load(IO.readlines(NODE_SRC_PATH)[1..-1].join)

shared['coreos']['etcd']['discovery'] = token

master = shared.deep_merge(master)
node = shared.deep_merge(node)
File.open(MASTER_DST_PATH, 'w') { |file| file.write("#cloud-config\n\n#{YAML.dump(master)}") }
File.open(NODE_DST_PATH, 'w') { |file| file.write("#cloud-config\n\n#{YAML.dump(node)}") }

Vagrant.configure("2") do |config|
  config.vm.box = "coreos-%s" % $update_channel
  config.vm.box_version = ">= 308.0.1"
  config.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json" % $update_channel

  config.vm.provider :vmware_fusion do |vb, override|
    override.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant_vmware_fusion.json" % $update_channel
  end

  config.vm.provider :virtualbox do |vb|
    vb.check_guest_additions = false
    vb.functional_vboxsf     = false
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  (1..$num_instances).each do |i|
    config.vm.define vm_name = "core-%02d" % i do |config|
      config.vm.hostname = vm_name

      if $enable_serial_logging
        logdir = File.join(File.dirname(__FILE__), "log")
        FileUtils.mkdir_p(logdir)

        serialFile = File.join(logdir, "%s-serial.txt" % vm_name)
        FileUtils.touch(serialFile)

        config.vm.provider :vmware_fusion do |v, override|
          v.vmx["serial0.present"] = "TRUE"
          v.vmx["serial0.fileType"] = "file"
          v.vmx["serial0.fileName"] = serialFile
          v.vmx["serial0.tryNoRxLoss"] = "FALSE"
        end

        config.vm.provider :virtualbox do |vb, override|
          vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          vb.customize ["modifyvm", :id, "--uartmode1", serialFile]
        end
      end

      if $expose_docker_tcp
        config.vm.network "forwarded_port", guest: 2375, host: ($expose_docker_tcp + i - 1), auto_correct: true
      end

      if $expose_etcd_tcp
        config.vm.network "forwarded_port", guest: 4001, host: ($expose_etcd_tcp + i - 1), auto_correct: true
      end

      if $expose_http
        config.vm.network "forwarded_port", guest: 80, host: ($expose_http + i - 1), auto_correct: true
      end

      config.vm.provider :vmware_fusion do |vb|
        vb.gui = $vb_gui
      end

      config.vm.provider :virtualbox do |vb|
        vb.gui = $vb_gui
        vb.memory = $vb_memory
        vb.cpus = $vb_cpus
      end

      ip = "172.17.8.#{i+100}"
      config.vm.network :private_network, ip: ip

      config.vm.synced_folder "./monitoring", "/home/core/monitoring", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']
      config.vm.synced_folder "./opt/bin", "/opt/bin", id: "opt_bin", :nfs => true, :mount_options => ['nolock,vers=3,udp']

      user_data = i == 1 ? MASTER_DST_PATH : NODE_DST_PATH

      config.vm.provision :file, :source => user_data, :destination => "/tmp/vagrantfile-user-data"
      config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true

    end
  end
end
