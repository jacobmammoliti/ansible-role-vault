Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox"
  config.vm.box = "bento/ubuntu-18.04"
  config.vm.define "vault01"

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yaml"
    ansible.groups = {
      "vault" => ["vault01"],
    }
  end
end