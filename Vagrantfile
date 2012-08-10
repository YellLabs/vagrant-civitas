# Available options:
#
#   :box        Set the base box.
#   :ip         Set the static IP of an additional interface.
#   :network    Set the additional interface type - :hostonly, :bridged
#   :puppet     Set to false to disable Puppet provisioning.
#   :memory     Override a basebox's default RAM allocation - numeric in MB.
#                   Note: the default value for an existing box will not be
#                   restored until the box is destroyed and recreated.

box     = "centos58-puppet26"
domain  = "vagrant.civitas.yb.int"
nodes   = {
    # Load balancing and caching.
    :wc01     => { :ip => '172.16.201.11' },
    :wc02     => { :ip => '172.16.201.12' },

    # Frontends.
    :ja01     => { :ip => '172.16.201.50' },
    :ja02     => { :ip => '172.16.201.51' },

    :pla01    => { :ip => '172.16.201.60' },
    :pla02    => { :ip => '172.16.201.61' },

    # Datastores.
    :db01     => { :ip => '172.16.201.100' },

    :mq01     => { :ip => '172.16.201.110' },

    :es01     => { :ip => '172.16.201.120' },
    :es02     => { :ip => '172.16.201.121' },

    :mgd01    => { :ip => '172.16.201.130' },
    :mgd02    => { :ip => '172.16.201.131' },

    :rk01     => { :ip => '172.16.201.140' },
    :rk02     => { :ip => '172.16.201.141' },

    :ml01     => { :ip => '172.16.201.150' },
    :ml02     => { :ip => '172.16.201.151' },

    # Administration.
    :pup51    => { :ip => '172.16.201.200', :box => 'centos62-puppet26', :memory => 1024 },

    :log51    => { :ip => '172.16.201.210', :box => 'centos62-puppet26' },
    :log52    => { :ip => '172.16.201.211', :box => 'centos62-puppet26' },
    :log53    => { :ip => '172.16.201.212', :box => 'centos62-puppet26' },
    :log54    => { :ip => '172.16.201.213', :box => 'centos62-puppet26' },
    :log55    => { :ip => '172.16.201.214', :box => 'centos62-puppet26' },

    # Build and test.
    :aft01    => { :ip => '172.16.201.220', :box => 'centos62-puppet26' },
    :bld01    => { :ip => '172.16.201.221', :box => 'centos62-puppet26' },
    :tm01     => { :ip => '172.16.201.222', :box => 'centos62-puppet26' },

    # RPM building.
    :rpmbuild5 => {},
    :rpmbuild6 => { :box => 'centos62-puppet26' },
}

Vagrant::Config.run do |config|
    nodes.each do |node_name, node_opts|
        config.vm.define node_name do |c|
            c.vm.box = node_opts[:box] || box

            # Host only NICs on EL5 take a long time.
            c.ssh.max_tries = 100
            c.ssh.forward_agent = true

            fqdn = "#{node_name}.#{domain}"
            c.vm.host_name = fqdn

            modifyvm_args = ['modifyvm', :id]
            modifyvm_args << '--name' << fqdn

            # workaround for https://github.com/mitchellh/vagrant/issues/516
            modifyvm_args << '--nictype1' << 'Am79C973'

            # Isolate guests from host networking.
            modifyvm_args << '--natdnsproxy1' << 'on'
            modifyvm_args << '--natdnshostresolver1' << 'on'

            unless node_opts[:memory].nil?
                modifyvm_args << '--memory' << node_opts[:memory].to_s
            end

            c.vm.customize(modifyvm_args)

            if node_opts[:network] == :bridged
                c.vm.network(:bridged)
            elsif node_opts[:ip]
                c.vm.network(:hostonly, node_opts[:ip])
            end

            unless node_opts[:puppet] == false
                c.vm.share_folder("extdata", "/tmp/vagrant-puppet/extdata", "puppet/extdata")
                c.vm.provision :puppet do |puppet|
                    puppet.manifest_file = "site.pp"
                    puppet.manifests_path = "puppet/manifests"
                    puppet.module_path = "puppet/modules"
                end
            end
        end
    end
end
