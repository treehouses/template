How to create vagrant boxes

```sh
# vagrant plugin install vagrant-disksize

vagrant cloud auth login
`cat key` # echo "export GITHUB_KEY=..." > key
export | grep GITHUB_KEY

vagrant destroy -f
rm template.log

vagrant up |& tee template.log
vagrant halt
vagrant package --output treehouses-0.10.4.box
```

upload the new box to https://app.vagrantup.com/treehouses/boxes/buster64
