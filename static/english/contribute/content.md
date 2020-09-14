##### Before you start

You can contribute to any pages by improving its content including this page. If you are not able to locate where to make change for such pages or sentences or whole topic. You can ask us and we will help you with that. You can ask on twitter, facebook or other social media handler of nishchay framework. You can directly send us message on any of social media platform.

##### Prerequisite

All of the translation such as sentences or word stored in `sentence` directory while other translations such whole page or learning center docs are stored in `static` directory.

Your must be familiar with markdown content because static contents are written markdown format. If you have never used markdown, don't worry its very easy. Please learn [here](https://guides.github.com/features/mastering-markdown/).

##### Setting up work environment
Once you install required prerequisite and framework

1. Create Github if you don't have
2. Fork Translations repo. Go to [Nishchay website translations](https://github.com/nishchay/translations) and click on `fork`.

3.  Point translations to your forked repo

```console
git remote add origin git@github.com:USERNAME/translations.git
```

4.  Now you should add upstream repo as a remote

```console
git remote add upstream https://github.com/nishchay/translations.git
```
5.  Create new branch from master

```console
git checkout -b BRANCH_NAME master
```
##### Submit work

Before you submit your work, you should update your branch

```console
git checkout master
git fetch upstream
git merge upstream/master
git checkout BRANCH_NAME
git rebase master
```

Push your changes to remote

```console
git add
git commit -m "{MESSAGE}"
git push origin BRANCH_NAME
```  

##### Pull request

You can create [pull request](https://github.com/nishchay/translations/compare) here.

