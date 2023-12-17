# to make sure the master node is created before the other nodes, we
# have to force a --no-parallel execution.
ENV['VAGRANT_NO_PARALLEL'] = 'yes'

require 'ipaddr'

number_of_nodes = 2
first_node_ip = '10.10.0.201'
node_ip_addr = IPAddr.new first_node_ip

Vagrant.configure(2) do |config|
  config.vm.box = "in-vagranti/ubuntu-20.04-amd64"

  config.vm.provider :libvirt do |lv, config|
    lv.memory = 1024
    lv.cpus = 2
    lv.cpu_mode = 'host-passthrough'
    # lv.nested = true
    lv.keymap = 'pt'
    config.vm.synced_folder '.', '/vagrant', type: 'nfs'
  end

  config.vm.provider 'virtualbox' do |vb|
    vb.linked_clone = true
    vb.memory = 1024
    vb.cpus = 2
  end

  (1..number_of_nodes).each do |n|
    name = "docker#{n}"
    fqdn = "#{name}.example.com"
    ip = node_ip_addr.to_s; node_ip_addr = node_ip_addr.succ

    config.vm.define name do |config|
      config.vm.hostname = fqdn
      config.vm.network :private_network, ip: ip, libvirt__forward_mode: 'route', libvirt__dhcp_enabled: false
      config.vm.provision 'shell', path: 'base.sh'
      config.vm.provision 'shell', path: 'certification-authority.sh'
      config.vm.provision 'shell', path: 'hosts.sh', args: [ip]
      config.vm.provision 'shell', path: 'install_docker.sh'
      config.vm.provision 'shell', path: 'docker_swarm.sh', args: [ip, first_node_ip]
      config.vm.provision 'shell', path: 'registry.sh' if ip == first_node_ip
      config.vm.provision 'shell', path: 'potainer.sh' if ip == first_node_ip
    #   config.vm.provision 'shell', path: 'provision-examples.sh' if ip == first_node_ip
    end
  end
end