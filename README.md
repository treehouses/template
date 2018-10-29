How to create vagrant boxes

```sh
vagrant cloud auth login
vagrant destroy -f

vagrant up |& tee template.log
vagrant halt
vagrant package --output ole-0.4.4.box
```
