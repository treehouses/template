How to create vagrant boxes

```sh
export VAGRANT_SERVER_URL=https://app.vagrantup.com
vagrant login

vagrant up |& tee template.log
vagrant halt
vagrant package --output ole-0.3.1.box
```
