##### Before you start

There are number developer around world, each have their own style of coding. As anyone can contribute to nishchay code its better to follow common coding standard which are followed by open source contributor community.

Below sections explains these standards and steps to start contributing to nishchay framework or website.
If you are contributing for first time, it would be better to first read `Coding standard` section.

##### Coding standards

Nishchay code must follow [PSR-1](https://www.php-fig.org/psr/psr-1/), [PSR-2](https://www.php-fig.org/psr/psr-1/) and [PSR-4](https://www.php-fig.org/psr/psr-1/) Apart from PSR coding standard, we have some more coding standard which must be followed.

##### Prerequisite

Make sure you have following in your machine

*   [Git](https://git-scm.com/downloads)
*   PHP
*   Caching servers and its php extensions: Memcache and RedisDB (If you are making changes which involves memcache or redisDB)
*   Database servers and its php extension: (MySQL, PostgreSQL, SQL Server, PDO, PDO extension for selected database)


##### Choose work

Visit [Nishchay Atlassian Jira](https://nishchay.atlassian.net/jira/software/c/projects/NF/issues?filter=allissues) and choose any task you want to work on.

If you have found an issue or thing you want to improve which does not exists in list, create new case and start working on it.

If you want Jira account, please send email to [support@nishchay.io](mailto:support@nishchay.io) from your registered email address(Nishchay website). 


##### Install development version

There's dedicated package for development of framework which comes with unit tests and other UI tools to test framework.

```console
composer create-project nishchay/develop nishchay
```

##### Setting up work environment

Once you install required prerequisite and framework

1. Create Github if you don't have
2. Fork Nishchay framework github repo. Go to [Nishchay Develop App](https://github.com/nishchay/framework) and click on `fork`.

3.  Point framework code to your forked repo

```console
cd vendors/nishchay/framework
git remote add origin git@github.com:USERNAME/framework.git
```

4.  Now you should add upstream repo as a remote

```console
git remote add upstream git://github.com/nishchay/framework.git
```
5.  Create new branch from master or branch of specific version.

Please check if work you to do if already exists in [Nishchay Atlassian Jira](https://nishchay.atlassian.net/jira/software/c/projects/NF/issues?filter=allissues), Put branch name same as issue number, example `NF-{number}`

```console
git checkout -b BRANCH_NAME master
```

OR

```console
git checkout -b BRANCH_NAME v1.0.1
```

##### Submit your work

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
git commit -m "{ISSUE_NUMBER} {MESSAGE}"
git push origin BRANCH_NAME
```  

##### Pull request

Create pull request with following points

1. Create [pull request](https://github.com/nishchay/translations/compare)
2.  Explain your work including if any pros or cons
3.  If you are adding feature, PR must contain how new feature can be used.
4.  If there's JIRA case for also add link to JIRA case
