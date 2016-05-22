# How to make PR upstream to Kubernetes

## Sign individual / corporate CLA

## PR flow

- Fork the main repository

- Clone your fork to local machine
```
$ export KPATH=$GOPATH
$ mkdir -p $KPATH/src/k8s.io/
$ cd $KPATH/src/k8s.io/
$ git clone https://github.com/$YOUR_GITHUB_USERNAME/kubernetes.git
$ cd kubernetes
$ git remote add upstream 'https://github.com/kubernetes/kubernetes.git'
```

- Create a branch and make changes
You might be use `master` branch as well

- Keeping your development fork in sync
```
$ git fetch upstream
$ git rebase upstream/master
```

- Commit changes to your fork
```
$ git commit
$ git push -f origin <branch> // normally master
```

- Creating pull request

## Using Godeps

- Populate your new GOPATH
```
$ cd $KPATH/src/k8s.io/kubernetes
$ godep restore
```

- Next, you can either add a new dependency or update an existing one.
```
# To add a new dependency, do:
$ cd $KPATH/src/k8s.io/kubernetes
$ go get path/to/dependency
# Change code in Kubernetes to use the dependency.
$ godep save ./...

# To update an existing dependency, do:
$ cd $KPATH/src/k8s.io/kubernetes
$ go get -u path/to/dependency
# Change code in Kubernetes accordingly if necessary.
$ godep update path/to/dependency/...
```

**References**
[Development Guide] https://github.com/kubernetes/kubernetes/blob/release-1.2/docs/devel/development.md
