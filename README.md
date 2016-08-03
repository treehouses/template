How to create vagrant boxes

```sh
vagrant up

vagrant ssh

#sudo apt upgrade
#grup-pc | first option | no install on drives

sudo apt-get clean
sudo dd if=/dev/zero of=/EMPTY bs=1M
sudo rm -f /EMPTY
cat /dev/null > ~/.bash_history && history -c && exit

vagrant package --output ole-0.1.0.box
```
