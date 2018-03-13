How to create vagrant boxes

```sh
vagrant login

vagrant up |& tee template.log
vagrant halt
vagrant package --output ole-0.3.2.box
```
