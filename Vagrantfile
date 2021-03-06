# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "commana/arsnova-debian-wheezy-puppet3-amd64"

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  # config.vm.box_url = "http://domain.com/path/to/above.box"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  synced_folder_type = synced_folder config, ".", "/vagrant"
  synced_folder_type = synced_folder config, "./puppet/files", "/etc/puppet/files"

  # Map UID used inside the VM to the UID of the user who started Vagrant
  config.nfs.map_uid = Process.uid
  config.nfs.map_gid = Process.uid

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
   config.vm.provider "virtualbox" do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
     # Use VBoxManage to customize the VM. For example to change memory:
     vb.memory = "2048"
     vb.cpus = "2"
   end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  # Enable provisioning with Puppet stand alone.  Puppet manifests
  # are contained in a directory path relative to this Vagrantfile.
  # You will need to create the manifests directory and a manifest in
  # the file fadenb/debian-wheezy-puppet3.pp in the manifests_path directory.
  #
  # An example Puppet manifest to provision the message of the day:
  #
  # # group { "puppet":
  # #   ensure => "present",
  # # }
  # #
  # # File { owner => 0, group => 0, mode => 0644 }
  # #
  # # file { '/etc/motd':
  # #   content => "Welcome to your Vagrant-built virtual machine!
  # #               Managed by Puppet.\n"
  # # }
  #
  pp_manifest_path = "puppet/manifests"
  pp_module_path = "puppet/modules"
  pp_manifest_file = "debian-wheezy.pp"

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # config.vm.provision "chef_solo" do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { :mysql_password => "foo" }
  # end

  # Enable provisioning with chef server, specifying the chef server URL,
  # and the path to the validation key (relative to this Vagrantfile).
  #
  # The Opscode Platform uses HTTPS. Substitute your organization for
  # ORGNAME in the URL and validation key.
  #
  # If you have your own Chef Server, use the appropriate URL, which may be
  # HTTP instead of HTTPS depending on your configuration. Also change the
  # validation key to validation.pem.
  #
  # config.vm.provision "chef_client" do |chef|
  #   chef.chef_server_url = "https://api.opscode.com/organizations/ORGNAME"
  #   chef.validation_key_path = "ORGNAME-validator.pem"
  # end
  #
  # If you're using the Opscode platform, your validator client is
  # ORGNAME-validator, replacing ORGNAME with your organization name.
  #
  # If you have your own Chef Server, the default validation client name is
  # chef-validator, unless you changed the configuration.
  #
  #   chef.validation_client_name = "ORGNAME-validator"

  config.vm.provision :hosts do |provisioner|
    provisioner.add_host '192.168.33.2', ['arsnova-dev.internal']
    provisioner.add_host '192.168.33.3', ['arsnova-production.internal']
  end

  config.vm.define "dev", primary: true do |dev|
    dev.vm.hostname = "arsnova-dev"
    dev.vm.provision "puppet" do |puppet|
      puppet.manifests_path = pp_manifest_path
      puppet.manifest_file = pp_manifest_file
      puppet.module_path = pp_module_path
      puppet.options = ["--environment=development"]
      puppet.facter = {
        "vagrant_owner" => "vagrant",
        "vagrant_group" => "vagrant",
        "synced_folder_type" => synced_folder_type
      }
    end
    dev.vm.network :private_network, :ip => '192.168.33.2'
    dev.vm.network "forwarded_port", guest: 8080, host: 8080
    # socket.io port
    dev.vm.network "forwarded_port", guest: 10443, host: 10443
    # CouchDB
    dev.vm.network "forwarded_port", guest: 5984, host: 5984
    # SonarQube
    dev.vm.network "forwarded_port", guest: 9000, host: 9000
    # Jenkins
    dev.vm.network "forwarded_port", guest: 9090, host: 9090
  end
  config.vm.define "production", autostart: false do |production|
    production.vm.hostname = "arsnova-production"
    production.vm.provision "puppet" do |puppet|
      puppet.manifests_path = pp_manifest_path
      puppet.manifest_file = pp_manifest_file
      puppet.module_path = pp_module_path
      puppet.options = ["--environment=production"]
      puppet.facter = {
        "vagrant_owner" => "vagrant",
        "vagrant_group" => "vagrant",
        "synced_folder_type" => synced_folder_type
      }
    end
    production.vm.network :private_network, :ip => '192.168.33.3'
    production.vm.network "forwarded_port", guest: 80, host: 8081
    # socket.io port
    production.vm.network "forwarded_port", guest: 10444, host: 10444
    # CouchDB
    production.vm.network "forwarded_port", guest: 5984, host: 5985
  end
end

# Defines a synced folder with optimized options for the OS and returns the
# type.
def synced_folder config, host_path, guest_path
  options = Hash.new
  case RUBY_PLATFORM
    when /darwin|linux/ then
      options[:type] = "nfs"
    when /mswin|mingw|cygwin/ then
      options[:type] = "smb"
    else
      options[:type] = "vboxsf"
  end
  config.vm.synced_folder host_path, guest_path, options

  options[:type]
end
