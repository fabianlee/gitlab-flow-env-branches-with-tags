

# Create original repo

tar xvfz gitlab-flow-env-branches-with-tags.tgz
cd gitlab-flow-env-branches-with-tags
git init
git add *
git commit -a -m "init"
git branch -M main
git remote add origin git@github.com:fabianlee/gitlab-flow-env-branches-with-tags.git
git push -u origin main

cd overlays
# 'main' branch always contains 'dev' overlay files

#
# 'preprod' branch will have both 'dev'+'preprod' overlay files
#
# create and switch to 'preprod' branch
branch_source=dev; branch=preprod
git branch $branch; git switch $branch
# populate branch
cp -r $branch_source $branch
sed -i "s/env: $branch_source/env: $branch/" $branch/ns-labels.yaml
# add and push to branch
git add $branch/*
git commit -a -m "content only for $branch branch"; git push -u origin $branch
git tag tag-$branch; git push --tags

#
# 'prod' branch will have 'dev'+'preprod'+'prod' overlay files
#
# create and switch to 'prod' branch
branch_source=preprod; branch=prod
git branch $branch; git switch $branch
# populate branch
cp -r $branch_source $branch
sed -i "s/env: $branch_source/env: $branch/" $branch/ns-labels.yaml
# add and push to branch
git add $branch/*
git commit -a -m "content only for $branch branch"; git push -u origin $branch
git tag tag-$branch; git push --tags

# create new branch and set upstream
git branch prod3 tag-prod; git push -u origin prod3
OR
git branch prod4 tag-prod; git push --set-upstream origin prod4


# move preprod tag
branch=preprod
git tag -d tag-$branch; git push origin :tag-$branch
# at this point, ACM says "fatal: Not a valid object name" ref=tag-preprod
git tag tag-$branch; git push --tags


# make change and then move preprod tag
timestr=$(date '+%Y-%m-%dT%H-%M-%S%z'); sed -i "s/mod2: .*/mod2: TIME$timestr/" ns-labels.yaml
git commit -a -m "first change at $timestr"
sha1=$(git rev-parse --short HEAD)
sleep 3
timestr=$(date '+%Y-%m-%dT%H-%M-%S%z'); sed -i "s/mod2: .*/mod2: TIME$timestr/" ns-labels.yaml
git commit -a -m "second change at $timestr"
sha2=$(git rev-parse --short HEAD)
git push


# move preprod tag to specific commit
branch=preprod
git tag -d tag-$branch; git push origin :tag-$branch
git tag tag-$branch $sha1 ; git push --tags

git tag -d tag-$branch; git push origin :tag-$branch
git tag tag-$branch $sha2 ; git push --tags

# more intuitive way of deleting remote tag
git push --delete origin tag-prod


# merge requests from other users, create personal fork with all branches preserved
git clone git@github.com:anotheruser/gitlab-flow-env-branches-with-tags.git
cd gitlab-flow-env-branches-with-tag
git branch -a
git checkout preprod
git remote add upstream git@github.com:fabianlee/gitlab-flow-env-branches-with-tags.git

# update branch
git switch main; git pull -r upstream main; git push
OR
git fetch upstream preprod; git merge upstream/preprod; git push

# update central repo
branch=preprod
git fetch origin $branch --tags --force; git merge origin/$branch
