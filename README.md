How to create vagrant boxes

```sh
export VAGRANT_SERVER_URL=https://app.vagrantup.com
vagrant login

vagrant up
vagrant halt
vagrant package --output ole-0.2.1.box
```
