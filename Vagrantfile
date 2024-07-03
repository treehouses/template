# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "debian/bookworm64"
  config.vm.box_version = "12.20240503.1"
  # config.disksize.size = '99GB'

  config.vm.hostname = "template"

  config.vm.define "template" do |template|
  end

  config.vm.provider "virtualbox" do |vb|
    vb.name = "template"
    vb.memory = "3333"
    vb.cpus = "3"
  end

  config.vm.post_up_message = "ole-vi debian bookworm box - see https://github.com/ole-vi/template/issues for help and bug reports"

  # Prevent TTY Errors (copied from laravel/homestead: "homestead.rb" file)... By default this is "bash -l".
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  config.vm.provision "shell", env: {"GITHUB_KEY" => ENV['GITHUB_KEY']}, inline: <<-SHELL
    log() {
      local message="$1"
      local type="$2"
      local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
      local color
      local endcolor="\033[0m"
    
      case "$type" in
        "info") color="\033[38;5;79m" ;;
        "success") color="\033[1;32m" ;;
        "error") color="\033[1;31m" ;;
        *) color="\033[1;34m" ;;
      esac
    
      echo -e "${color}${timestamp} - ${message}${endcolor}"
    }
    # install important packages
    apt update --allow-releaseinfo-change
    apt install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common parted
    # remove swap partition
    # add swap by file
    # resize root
    # install docker https://docs.docker.com/engine/install/debian/#install-using-the-repository
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    log "docker version: $(docker version)" "info"
    log "docker compose version: $(docker compose version)" "info"
    # install node https://deb.nodesource.com/
    curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
    sudo apt install -y nodejs
    log "nodejs version: $(nodejs -v)" "info"
    # adding official chrome apt servers
    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
    echo "deb http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
    # next round of packages
    apt update
    apt upgrade -y
    apt install -y vim screen htop git autossh google-chrome-stable wget unzip jq aptitude tor netcat-openbsd net-tools openvpn speedtest-cli nmap bc iotop ffmpeg dos2unix bats go-1.22 qemu-user-static kpartx aria2 python-requests links tree imagemagick
    log "unzip version: $(unzip -v)" "info"
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
    log "docker images: $(docker images)" "info"
    # install CLI's
    npm install -g @angular/cli @treehouses/cli
    sync; sync; sync
    ln -sr "/usr/lib/node_modules/@treehouses/cli/_treehouses" "/etc/bash_completion.d/_treehouses"
    ls -al "/etc/bash_completion.d/_treehouses"
    log "treehouses version: $(treehouses version)" "info"
  SHELL
end
