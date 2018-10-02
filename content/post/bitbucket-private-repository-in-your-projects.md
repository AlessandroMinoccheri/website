---
title: "Using Bitbucket Private Repository in Your Projects"
date: 2018-10-02T08:58:31+02:00
draft: false
---

Many times you may like to use your personal private library in another project.
For example, you may have many private projects that share a common library that can't be open source for your business.
You can use Packagist and pay to have a private repository or you can use Bitbucket with its FREE private repositories.

To include your private Bitbucket repository via Composer you need to add this lines into your composer.json:


```
"repositories":[
     {
         "type": "vcs",
         "url" : "git@bitbucket.org:yourName/yourRepository.git"
     }
 ]
 ```

 VCS stands for version control system. This includes versioning systems like git, svn, fossil or hg. Composer has a repository type for installing packages from these systems.

 If you launch composer install now, you have to put a Consumer key into your CLI (command line interface).
 This isn't a problem but if you have continuous integration, or your deploy system launches composer install the command, you must have an automatic system able to insert the consumer key or some magic script to manage it.

 So, to avoid this problem you need to open your main repository, from Bitbucket for example, create an ssh key and copy the public key.

![Directory Structure Image](https://alessandrominoccheri.github.io/img/generate-ssh-key.jpg)

 After that you need to open your private library from Bitbucket, add an access ssh key and paste the public key created previously.

![Directory Structure Image](https://alessandrominoccheri.github.io/img/add-ssh-key.jpg)

 Now if you launch composer install any request comes up into your CLI and you can have a perfect automatic deployment.

 


