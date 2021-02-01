user = `whoami`
user = user.strip + '-autofs'

servers = [
  { svrName: "#{user}-nfs", playbook: 'test-nfs.yml', template: 'centos/7', ip: '192.168.60.5', memory: '2048', vcpus: '1', },
  { svrName: "#{user}-client1", playbook: 'test-autofs.yml', template: 'centos/7', ip: '192.168.60.6', memory: '1024', vcpus: '1'}
]

inventory = {
      "nfs" => ["#{user}-nfs"],
      "clients" => ["#{user}-client1","#{user}-client2","#{user}-client3"]
#      "repository-control:vars" => {"var1" => "value1", "var2" => "value2"}
}

host_vars = {
      "#{user}-nfs" => { "nfs_exports" => ["/home/public *(rw,sync,no_root_squash)"], "nfs_rpcbind_state" => "started", "nfs_rpcbind_enabled" => "true" },
      "#{user}-client1" => { "nfs_host" => "192.168.60.5", "nfs_path" => "/home/public", "nfs_options" => "rw,sync" },
      "#{user}-client2" => { "nfs_host" => "192.168.60.5", "nfs_path" => "/home/public", "nfs_options" => "rw,sync" },
      "#{user}-client3" => { "nfs_host" => "192.168.60.5", "nfs_path" => "/home/public", "nfs_options" => "rw,sync" }
}

Vagrant.configure('2') do |config|

  servers.each do |server|

    host_name = server[:svrName]

    config.vm.define host_name do |node|

        node.vm.hostname = host_name
        node.vm.box = server[:template]
        node.vm.network 'private_network', ip: server[:ip]
	    #node.vm.network "public_network", :dev => "br0", :mode => 'bridge', :type => "bridge"

        node.vm.provider :libvirt do |v|
            v.memory = server[:memory]
            v.cpus = server[:vcpus]

            if server.key?(:disks)
                server[:disks].each do |disk|
                    v.storage :file, :size => disk
                end
            end

        end

        node.vm.provision 'ansible' do |ansible|
            ansible.playbook = server[:playbook]
            ansible.galaxy_role_file = 'roles/requirements.yml'
            ansible.host_vars = host_vars
            ansible.groups = inventory
        end
    end

  end
end

