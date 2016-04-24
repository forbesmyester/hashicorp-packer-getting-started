# Custom Vagrant Config
#
# NOTE: Specify 'SSH_KEY_TYPE', 'SSH_PRIVATE_KEY', 'SSH_PUBLIC_KEY' and 'ROLES' during provisioning\n\n"

VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "./packer_virtualbox-ovf_virtualbox.box"
  # config.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/vivid/current/vivid-server-cloudimg-amd64-vagrant-disk1.box"
  # config.vm.network "forwarded_port", guest: 80, host: 8080
  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"
  config.ssh.forward_agent = true # See https://coderwall.com/p/p3bj2a/cloning-from-github-in-vagrant-using-ssh-agent-forwarding
  config.vm.provider :virtualbox do |vb|
    # vb.gui = true
    vb.customize ["modifyvm", :id, "--memory", "2048"]
  end
end
