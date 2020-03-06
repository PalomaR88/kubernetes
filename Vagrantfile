# -- mode: ruby --
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :servidor1 do |servidor1|
    servidor1.vm.box = "debian/buster64"
    servidor1.vm.hostname = "Docker"
    servidor1.vm.network :public_network,:bridge=>"wlp1s0" 
    #servidor1.vm.network :public_network,:bridge=>"enp2s0"
  end
end

