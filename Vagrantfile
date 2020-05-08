# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "debian/contrib-buster64"
  config.vm.box_version = "10.1.0"
  config.disksize.size = '99GB'

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

  config.vm.post_up_message = "treehouses debian box - see https://github.com/treehouses/template/issues for help and bug reports"

  # Prevent TTY Errors (copied from laravel/homestead: "homestead.rb" file)... By default this is "bash -l".
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  config.vm.provision "shell", env: {"GITHUB_KEY" => ENV['GITHUB_KEY']}, inline: <<-SHELL
    # adding backports
    echo "deb http://ftp.de.debian.org/debian buster-backports main non-free" | sudo tee -a /etc/apt/sources.list
    # blacklist grub-pc and linux-image-amd64 from upgrades
    echo grub-pc hold | dpkg --set-selections
    echo linux-image-amd64 hold | dpkg --set-selections
    # temporary fix also for unwanted interactive config upgrading openssh-server
    echo openssh-server hold | dpkg --set-selections
    # install important packages
    apt update --allow-releaseinfo-change
    apt install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common parted
    # remove swap partition
    swapoff -a
    sed -i -e '11,2d;12d' /etc/fstab
    printf "d\n5\nd\n2\nw\n" | fdisk /dev/sda
    # add swap by file
    fallocate -l 1g /swapfile
    chmod 600 /swapfile
    mkswap /swapfile
    swapon /swapfile
    echo "/swapfile    none    swap    sw    0   0" >> /etc/fstab
    # resize root
    printf "R\nWyes\nQ" | cfdisk /dev/sda
    partprobe
    resize2fs /dev/sda1
    # adding official docker/chrome/nodesource apt servers
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
    echo "deb http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource.gpg.key | sudo apt-key add -
    add-apt-repository "deb https://deb.nodesource.com/node_10.x buster main"
    # next round of packages
    apt update
    apt upgrade -y
    apt install -y vim screen htop git autossh docker-ce google-chrome-stable nodejs wget unzip jq aptitude tor netcat-openbsd net-tools openvpn speedtest-cli nmap bc iotop ffmpeg dos2unix bats
    usermod -aG docker $USER
    usermod -aG docker vagrant
    # install docker-compose
    curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
    # blacklist virtualbox-guest-* from upgrades
    echo virtualbox-guest-dkms hold | dpkg --set-selections
    echo virtualbox-guest-utils hold | dpkg --set-selections
    # install docker couchdb 2.3.1
    docker pull treehouses/couchdb:2.3.1
    # install docker planet latest
    docker pull treehouses/planet:latest
    docker pull treehouses/planet:db-init
    docker tag treehouses/planet:latest treehouses/planet:local
    docker tag treehouses/planet:db-init treehouses/planet:db-init-local
    wget https://raw.githubusercontent.com/open-learning-exchange/planet/master/docker/planet.yml
    wget https://raw.githubusercontent.com/open-learning-exchange/planet/master/docker/volumes.yml
    wget https://raw.githubusercontent.com/open-learning-exchange/planet/master/docker/install.yml
    # install terraform
    wget https://releases.hashicorp.com/terraform/0.12.10/terraform_0.12.10_linux_amd64.zip
    unzip terraform_0.12.10_linux_amd64.zip
    mkdir -p /usr/local/bin/
    mv terraform /usr/local/bin/.
    # install balena
    url="https://github.com/balena-os/balena-engine/releases/download/v17.12.0/balena-engine-v17.12.0-x86_64.tar.gz"
    curl -sL "$url" | sudo tar xzv -C /usr/local/bin --strip-components=1
    ln -sr /usr/local/bin/balena-engine /usr/local/bin/balena
    groupadd balena-engine
    usermod -aG balena-engine vagrant
    usermod -aG balena-engine root
    service_file=/etc/systemd/system/balena.service
    socket_file=/etc/systemd/system/balena.socket
    {
      echo "[Unit]"
      echo "Description=Docker Application Container Engine"
      echo "Documentation=https://docs.docker.com"
      echo "After=network-online.target docker.socket firewalld.service"
      echo "Wants=network-online.target"
      echo "Requires=balena.socket"
      echo ""
      echo "[Service]"
      echo "Type=notify"
      echo "# the default is not to use systemd for cgroups because the delegate issues still"
      echo "# exists and systemd currently does not support the cgroup feature set required"
      echo "# for containers run by docker"
      echo "ExecStart=/usr/local/bin/balena-engine-daemon -H unix:///var/run/balena-engine.sock"
      echo "ExecReload=/bin/kill -s HUP $MAINPID"
      echo "LimitNOFILE=1048576"
      echo "# Having non-zero Limit*s causes performance problems due to accounting overhead"
      echo "# in the kernel. We recommend using cgroups to do container-local accounting."
      echo "LimitNPROC=infinity"
      echo "LimitCORE=infinity"
      echo "# Uncomment TasksMax if your systemd version supports it."
      echo "# Only systemd 226 and above support this version."
      echo "#TasksMax=infinity"
      echo "TimeoutStartSec=0"
      echo "# set delegate yes so that systemd does not reset the cgroups of docker containers"
      echo "Delegate=yes"
      echo "# kill only the docker process, not all processes in the cgroup"
      echo "KillMode=process"
      echo "# restart the docker process if it exits prematurely"
      echo "Restart=on-failure"
      echo "StartLimitBurst=3"
      echo "StartLimitInterval=60s"
      echo ""
      echo "[Install]"
      echo "WantedBy=multi-user.target"
    } > "$service_file"
    {
      echo "[Unit]"
      echo "Description=Docker Socket for the API"
      echo "PartOf=balena.service"
      echo ""
      echo "[Socket]"
      echo "ListenStream=/var/run/balena-engine.sock"
      echo "SocketMode=0660"
      echo "SocketUser=root"
      echo "SocketGroup=balena-engine"
      echo ""
      echo "[Install]"
      echo "WantedBy=sockets.target"
    } > "$socket_file"
    rm -rf /var/lib/balena-engine
    ln -sr /var/lib/docker /var/lib/balena-engine
    # install CLI's
    npm install -g @angular/cli @treehouses/cli
    sync; sync; sync
    echo "$PWD"
    ln -sr "/usr/lib/node_modules/@treehouses/cli/_treehouses" "/etc/bash_completion.d/_treehouses"
    ls -al "/etc/bash_completion.d/_treehouses"
    # Add CORS to CouchDB so app has access to databases
    git clone https://github.com/pouchdb/add-cors-to-couchdb.git
    cd add-cors-to-couchdb
    npm install
    cd ..
    # bats additional stuff
    npm install --unsafe-perm -g bats-support@0.3.0
    npm install --unsafe-perm -g bats-assert@2.0.0
    # github cli
    wget https://github.com/cli/cli/releases/download/v0.7.0/gh_0.7.0_linux_amd64.deb
    dpkg -i gh_0.7.0_linux_amd64.deb
    rm gh_0.7.0_linux_amd64.deb
    # youtube-dl
    curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
    chmod a+rx /usr/local/bin/youtube-dl
    # planet
    wget https://raw.githubusercontent.com/open-learning-exchange/planet/master/package.json
    npm i --unsafe-perm
    npm run webdriver-set-version
    mv node_modules /vagrant_node_modules
    chown vagrant:vagrant /vagrant_node_modules
    rm package.json
    # sshkeys
    mkdir -p /root/.ssh
    chmod 700 .ssh
    echo $GITHUB_KEY
    treehouses sshkey github addteam treehouses support $GITHUB_KEY
    sync; sync; sync
    cat /root/.ssh/authorized_keys >> /home/vagrant/.ssh/authorized_keys
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
