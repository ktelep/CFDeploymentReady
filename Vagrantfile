# -*- mode: ruby -*-
# vi: set ft=ruby :

$manager_script = <<SCRIPT
#!/usr/bin/env bash

# Grab Ubuntu build packages, etc
sudo -E add-apt-repository -y multiverse
sudo apt-get update && sudo apt-get install -y build-essential ruby ruby-dev git zip linux-headers-`uname -r` zlib1g-dev libsqlite3-dev
sudo gem install bosh_cli --no-ri --no-rdoc

# Install Go
echo "Installing Go"
curl -s -OL http://storage.googleapis.com/golang/go1.4.2.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.4.2.linux-amd64.tar.gz

# Setup Environment
echo "export GOPATH=~/go" >> ~/.profile
echo "export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin" >> ~/.profile
export GOPATH=~/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin

# Grab our git repos
echo "Collecting code from GitHub"
git clone https://github.com/cloudfoundry/bosh-lite.git
git clone https://github.com/cloudfoundry/cf-release

# Patch cf-release
echo "Patching cf for our environment (BOSH target is at 192.168.50.4, not this VM)"
cd /home/vagrant/bosh-lite/bin
patch < /vagrant/provision_cf.patch
cd -

# Build Spiff
echo "Building Spiff"
git clone https://github.com/cloudfoundry-incubator/spiff.git /tmp/go/src/github.com/cloudfoundry-incubator/spiff
GOPATH=/tmp/go /tmp/go/src/github.com/cloudfoundry-incubator/spiff/scripts/build
sudo cp /tmp/go/src/github.com/cloudfoundry-incubator/spiff/spiff /usr/local/bin
sudo chmod a+x /usr/local/bin/spiff
rm -rf /tmp/go


SCRIPT

Vagrant.configure(2) do |config|
  config.vm.define "cloudFoundry" do |cloudfoundry|
	  cloudfoundry.vm.box = "cloudfoundry/bosh-lite"
	  cloudfoundry.vm.provider :virtualbox do |v, override|
	      override.vm.network :private_network, ip: '192.168.50.4', id: :local
              override.vm.network :public_network
	      v.memory = 3192
	      v.cpus = 2
  	  end
  end

  config.vm.define "manager" do |manager|
	  manager.vm.box = "ubuntu/trusty64"
	  manager.vm.provider :virtualbox do |v, override|
		  override.vm.network :private_network, ip: '192.168.50.14', id: :local
		  override.vm.network :public_network
          	  override.vm.provision "shell", inline: $manager_script, privileged: false
		  v.memory = 2048
		  v.cpus = 2
	  end
  end
end
