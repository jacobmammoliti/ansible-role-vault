Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox"
  config.vm.box = "bento/ubuntu-20.04"

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "tests/test.yml"
  end
end