# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

yml_config = {}
if File.exist?("vagrant_config.yml")
  yml_config = YAML::load_file("vagrant_config.yml")
end
# Minimum memory requirement is 2gb. Doubling to 4gb for safety.
user_config = {
  private_ip: "192.168.150.2",
  box: "hashicorp/precise64",
  box_url: nil,
  forward_port: 8001,
  memory: 4096,
  local: {
  },
  plugins: [
    "about",
    "meatspace",
    "liveupdate"
  ],
  testData: false,
  cpu: 4,
  dhcp: true,
  hostname: "reddit.local",
  nfs: true,
  smb: false
}.merge(yml_config)

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = user_config[:box]
  config.vm.box_url = user_config[:box_url]
  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  if user_config[:dhcp]
    config.vm.network "private_network", type: 'dhcp'
  else
    config.vm.network "private_network", ip: user_config[:private_ip]
  end
  config.vm.hostname = user_config[:hostname]
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  if user_config[:forward_port]
    config.vm.network "forwarded_port", guest: 80, host: user_config[:forward_port]
  end
  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  config.ssh.forward_agent = true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
    # vb.gui = false
    # # Use VBoxManage to customize the VM. For example to change memory:
    # vb.customize [
      # "modifyvm", :id,
      # "--memory", user_config[:memory],
      # "--cpus", user_config[:cpu]
    # ]
  # end

  # config.vm.provider "kvm" do |kvm|
    # kvm.core_number = user_config[:cpu]
    # kvm.memory_size = user_config[:memory].to_s+"m"
  # end

  # config.vm.provider "vmware_fusion" do |v|
    # v.vmx["numvcpus"] = user_config[:cpu]
    # v.vmx["memsize"] = user_config[:memory]
  # end

  config.vm.provider :aws do |aws, override|
    aws.access_key_id = "YAKIAJYPIHXVNZHVWLYKQ"
    aws.secret_access_key = "gyYfhgU7FGmahrZtq2lUIWAnkASLQ80jIj05yrHG"
   #aws.session_token = "SESSION TOKEN"
    aws.keypair_name = "eastKeyPair"
	aws.use_iam_profile = true
	aws.security_groups = %w[default]
    aws.ami = "ami-7747d01e"

    override.ssh.username = "ubuntu"
    override.ssh.private_key_path = "eastKeyPair.pem"
  end
  
  if Vagrant.has_plugin?("vagrant-cachier")
    # Configure cached packages to be shared between instances of the same base box.
    # More info on http://fgrehm.viewdocs.io/vagrant-cachier/usage
    config.cache.scope = :box

    # OPTIONAL: If you are using VirtualBox, you might want to use that to enable
    # NFS for shared folders. This is also very useful for vagrant-libvirt if you
    # want bi-directional sync
    config.cache.synced_folder_opts = {
      type: user_config[:nfs] && :nfs || user_config[:smb] && :smb || nil,
      # The nolock option can be useful for an NFSv3 client that wants to avoid the
      # NLM sideband protocol. Without this option, apt-get might hang if it tries
      # to lock files needed for /var/cache/* operations. All of this can be avoided
      # by using NFSv4 everywhere. Please note that the tcp option is not the default.
      mount_options: user_config[:nfs] && ['rw', 'vers=3', 'tcp', 'nolock'] || []
    }
    # For more information please check http://docs.vagrantup.com/v2/synced-folders/basic_usage.html
  end

  user_config[:local].each do |key, value|
    if value
      config.vm.synced_folder value, '/host-raw/src/'+key, type: user_config[:nfs] && 'nfs' || user_config[:smb] && 'smb' || nil, mount_options:user_config[:nfs] && ['rw', 'vers=3', 'tcp', 'nolock'] || nil
      config.bindfs.bind_folder '/host-raw/src/'+key, '/host/src/'+key
    end
  end

  config.vm.provision :shell, :path => 'bootstrap.sh', :args => [
    user_config[:plugins].join(" "),
    user_config[:testData] && '1' || '',
    user_config[:hostname]
  ]
end
