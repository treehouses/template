How to create vagrant boxes

```sh
vagrant login
vagrant destroy -f

vagrant up |& tee template.log
vagrant halt
vagrant package --output ole-0.4.2.box
```
