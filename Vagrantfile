# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "debian/contrib-buster64"
  config.vm.box_version = "10.0.0"

  config.vm.hostname = "template"

  config.vm.define "template" do |template|
  end

  config.vm.provider "virtualbox" do |vb|
    vb.name = "template"
  end

  config.vm.provider "virtualbox" do |vb|
     vb.memory = "3333"
     vb.cpus = "3"
  end

  config.push.define "atlas" do |push|
    push.app = "treehouses/buster64"
  end

  # Prevent TTY Errors (copied from laravel/homestead: "homestead.rb" file)... By default this is "bash -l".
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  config.vm.provision "shell", env: {"GITHUB_KEY" => ENV['GITHUB_KEY']}, inline: <<-SHELL
    # adding backports
    echo "deb http://ftp.de.debian.org/debian buster-backports main non-free" | sudo tee -a /etc/apt/sources.list
    # blacklist grub-pc and linux-image-amd64 from upgrades
    echo grub-pc hold | dpkg --set-selections
    echo linux-image-amd64 hold | dpkg --set-selections
    # install important packages
    sudo apt update --allow-releaseinfo-change
    sudo apt install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common
    # adding official docker/chrome/nodesource apt servers
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
    echo "deb http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource.gpg.key | sudo apt-key add -
    add-apt-repository "deb https://deb.nodesource.com/node_10.x buster main"
    # next round of packages
    sudo apt update
    sudo apt upgrade -y
    sudo apt install -y vim screen htop git autossh docker-ce google-chrome-stable nodejs wget unzip jq aptitude tor netcat-openbsd net-tools openvpn
    sudo usermod -aG docker $USER
    sudo usermod -aG docker vagrant
    # install docker-compose
    sudo curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    # blacklist virtualbox-guest-* from upgrades
    echo virtualbox-guest-dkms hold | dpkg --set-selections
    echo virtualbox-guest-utils hold | dpkg --set-selections
    # install docker couchdb 2.3.0
    sudo docker pull treehouses/couchdb:2.3.0
    # install docker planet latest
    sudo docker pull treehouses/planet:latest
    sudo docker pull treehouses/planet:db-init
    sudo docker tag treehouses/planet:latest treehouses/planet:local
    sudo docker tag treehouses/planet:db-init treehouses/planet:db-init-local
    wget https://raw.githubusercontent.com/open-learning-exchange/planet/master/docker/planet.yml
    wget https://raw.githubusercontent.com/open-learning-exchange/planet/master/docker/volumes.yml
    wget https://raw.githubusercontent.com/open-learning-exchange/planet/master/docker/install.yml
    # install CLI's
    sudo npm install -g @angular/cli @treehouses/cli
    # Add CORS to CouchDB so app has access to databases
    git clone https://github.com/pouchdb/add-cors-to-couchdb.git
    cd add-cors-to-couchdb
    npm install
    cd ..
    # plant
    wget https://raw.githubusercontent.com/open-learning-exchange/planet/master/package.json
    npm i --unsafe-perm
    sudo npm run webdriver-set-version
    mv node_modules /vagrant_node_modules
    chown vagrant:vagrant /vagrant_node_modules
    rm package.json
    # sshkeys
    echo $GITHUB_KEY
    treehouses sshkey addgithubgroup treehouses support $GITHUB_KEY
    # *.onion
    {
       echo "Host *.onion"
       echo "  ProxyCommand /bin/nc.openbsd -x localhost:9050 -X 5 %h %p"
    } > /root/.ssh/config
    v=$(cat /vagrant/README.md | grep -Po "(?<=treehouses-)(.*)\.box")
    echo "${v:0:-4}" > /boot/version.txt
    sync;sync;sync
    # change password for vagrant user
    echo vagrant:tnargav | chpasswd
    # prepare for packaging
    sudo apt-get clean
    sudo dd if=/dev/zero of=/EMPTY bs=1M
    sudo rm -f /EMPTY
    cat /dev/null > ~/.bash_history && history -c && exit
  SHELL
end
