
Fetch a bunch called devel
````
git fetch
git checkout devel
git pull
````

# checkout branches as directories  (used for puppet checkout branch as envrionment), but useful for other work if we want checkout branchs

```
git_dir=$1
env_dir=$2
# Move to the relevant git dir, and perform a refresh
cd $git_dir
su gituser -c "cd ${git_dir}; git fetch -q --all -p"

# Get the array of branches, ignoring meta ones
branches=$(git for-each-ref refs/heads/ --format='%(refname:short)')
# Loop the branches
for branch in $branches; do
echo "Checking branch $branch"
# if environments/branchname doesn't exist, clone into it
if [ ! -d $env_dir/$branch ]; then
mkdir $env_dir/$branch
git clone $git_dir $env_dir/$branch
fi

# move to the clone
cd $env_dir/$branch

# ensure we're in the right branch, if this is the first time the script has
# run then we'll be in the default (master) branch
git checkout $branch

# we now need to get the latest content, note that we don't just pull because
# a pull will try and merge which will fail miserably if someone has deleted
# their branch and re-created it

# ensure we have the latest content
git fetch

# ensure our local content matches the origin exactly
git reset --hard origin/$branch

done
```

git squash commits into one commit, 2 commits squash example
```
git log --oneline
git rebase -i HEAD~2
git push origin --delete $BRANCHNAME
git push origin $BRANCHNAME

```
