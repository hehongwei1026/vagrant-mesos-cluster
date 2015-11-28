#
# A Vagrant-based Mesos cluster
#

# List of VM and their roles
HOSTS = {
  "masters" => {
    "master1" => "192.168.99.11",
    "master2" => "192.168.99.12",
    "master3" => "192.168.99.13"
  },
  "nodes" => {
    "node1"   => "192.168.99.21",
    "node2"   => "192.168.99.22",
    "node3"   => "192.168.99.23"
  }
}

Vagrant.configure(2) do |config|
  config.vm.box = "bento/centos-7.1"

  HOSTS['masters'].each_with_index do |host, index|
    config.vm.define host[0] do |machine|
      machine.vm.hostname = host[0]

      # be aware about dhcp issues https://github.com/mitchellh/vagrant/issues/2968
      machine.vm.network "private_network", :ip => host[1]

      if host == "master1"
        machine.vm.network :forwarded_port, guest: 80, host: 80
        machine.vm.network :forwarded_port, guest: 81, host: 81      # HaProxy
        machine.vm.network :forwarded_port, guest: 8080, host: 8080  # Mesos
        machine.vm.network :forwarded_port, guest: 5050, host: 5050  # Marathon
        machine.vm.network :forwarded_port, guest: 8000, host: 8000  # Bamboo
      end

      machine.vm.provision "ansible" do |ansible|
          ansible.playbook = "provisioning/master.yml"
          ansible.groups = { "masters" => HOSTS['masters'].keys }
          ansible.extra_vars = {
            zk_id: "#{index + 1}",
            zk_host1: "master1",
            zk_host2: "master2",
            zk_host3: "master3",
            mesos_quorum: 2,
            hosts: HOSTS['masters'].merge(HOSTS['nodes'])
          }
      end

    end
  end

  HOSTS['nodes'].each_with_index do |host, index|
    config.vm.define host[0] do |machine|
      machine.vm.hostname = host[0]
      machine.vm.network "private_network", :ip => host[1]

      machine.vm.provision "ansible" do |ansible|
          ansible.playbook = "provisioning/node.yml"
          ansible.groups = { "nodes" => HOSTS['nodes'].keys }
          ansible.extra_vars = {
            zk_host1: "master1",
            zk_host2: "master2",
            zk_host3: "master3",
            hosts: HOSTS['masters'].merge(HOSTS['nodes'])
          }
      end
    end
  end

end
