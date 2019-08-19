How to create vagrant boxes

```sh
vagrant cloud auth login
`cat key` # echo "export GITHUB_KEY=..." > key

vagrant destroy -f
rm template.log

vagrant up |& tee template.log
vagrant halt
vagrant package --output treehouses-0.9.6.box
```

upload the new box to https://app.vagrantup.com/treehouses/boxes/buster64
