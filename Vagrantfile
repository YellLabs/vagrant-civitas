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
    :wc01     => {},
    :wc02     => {},

    # Frontends.
    :ja01     => {},
    :ja02     => {},

    :pla01    => {},
    :pla02    => {},

    # Datastores.
    :db01     => {},

    :mq01     => {},

    :es01     => {},
    :es02     => {},

    :mgd01    => {},
    :mgd02    => {},

    :rk01     => {},
    :rk02     => {},

    # Administration.
    :pup51    => {},
    :log51    => {},
    :log52    => {},
    :log53    => {},
    :log54    => {},
    :log55    => {},

    # Build and test.
    :aft01    => {},
    :bld01    => {},
    :tm01     => {},
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
