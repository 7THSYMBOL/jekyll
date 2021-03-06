---
layout: docs
title: Deployment methods
permalink: /docs/deployment-methods/
---

Sites built using Jekyll can be deployed in a large number of ways due to the static nature of the generated output. A few of the most common deployment techniques are described below.

## Web hosting providers (FTP)

Just about any traditional web hosting provider will let you upload files to their servers over FTP. To upload a Jekyll site to a web host using FTP, simply run the `jekyll build` command and copy the generated `_site` folder to the root folder of your hosting account. This is most likely to be the `httpdocs` or `public_html` folder on most hosting providers.

### FTP using Glynn

There is a project called [Glynn](https://github.com/dmathieu/glynn), which lets you easily generate your Jekyll powered website’s static files and
send them to your host through FTP.

## Self-managed web server

If you have direct access to the deployment web server, the process is essentially the same, except you might have other methods available to you (such as `scp`, or even direct filesystem access) for transferring the files. Just remember to make sure the contents of the generated `_site` folder get placed in the appropriate web root directory for your web server.

## Automated methods

There are also a number of ways to easily automate the deployment of a Jekyll site. If you’ve got another method that isn’t listed below, we’d love it if you [contributed](../contributing/) so that everyone else can benefit too.

### Git post-update hook

If you store your Jekyll site in [Git](https://git-scm.com/) (you are using
version control, right?), it’s pretty easy to automate the
deployment process by setting up a post-update hook in your Git
repository, [like
this](http://web.archive.org/web/20091223025644/http://www.taknado.com/en/2009/03/26/deploying-a-jekyll-generated-site/).

### Git post-receive hook

To have a remote server handle the deploy for you every time you push changes using Git, you can create a user account which has all the public keys that are authorized to deploy in its `authorized_keys` file. With that in place, setting up the post-receive hook is done as follows:

{% highlight shell %}
laptop$ ssh deployer@example.com
server$ mkdir myrepo.git
server$ cd myrepo.git
server$ git --bare init
server$ cp hooks/post-receive.sample hooks/post-receive
server$ mkdir /var/www/myrepo
{% endhighlight %}

Next, add the following lines to hooks/post-receive and be sure Jekyll is
installed on the server:

{% highlight shell %}
GIT_REPO=$HOME/myrepo.git
TMP_GIT_CLONE=$HOME/tmp/myrepo
PUBLIC_WWW=/var/www/myrepo

git clone $GIT_REPO $TMP_GIT_CLONE
jekyll build -s $TMP_GIT_CLONE -d $PUBLIC_WWW
rm -Rf $TMP_GIT_CLONE
exit
{% endhighlight %}

Finally, run the following command on any users laptop that needs to be able to
deploy using this hook:

{% highlight shell %}
laptops$ git remote add deploy deployer@example.com:~/myrepo.git
{% endhighlight %}

Deploying is now as easy as telling nginx or Apache to look at
`/var/www/myrepo` and running the following:

{% highlight shell %}
laptops$ git push deploy master
{% endhighlight %}

### Jekyll-hook

You can also use jekyll-hook, a server that listens for webhook posts from
GitHub, generates a website with Jekyll, and moves it somewhere to be
published. Use this to run your own GitHub Pages-style web server.

This method is useful if you need to serve your websites behind a firewall,
need extra server-level features like HTTP basic authentication or want to
host your site directly on a CDN or file host like S3.

Setup steps are fully documented
[in the `jekyll-hook` repo](https://github.com/developmentseed/jekyll-hook).

### Static Publisher

[Static Publisher](https://github.com/static-publisher/static-publisher) is another automated deployment option with a server listening for webhook posts, though it's not tied to GitHub specifically. It has a one-click deploy to Heroku, it can watch multiple projects from one server, it has an easy to user admin interface and can publish to either S3 or to a git repository (e.g. gh-pages).

### Rake

Another way to deploy your Jekyll site is to use [Rake](https://github.com/ruby/rake), [HighLine](https://github.com/JEG2/highline), and
[Net::SSH](https://github.com/net-ssh/net-ssh). A more complex example of deploying Jekyll with Rake that deals with multiple branches can be found in [Git Ready](https://github.com/gitready/gitready/blob/cdfbc4ec5321ff8d18c3ce936e9c749dbbc4f190/Rakefile).


### scp

Once you’ve generated the `_site` directory, you can easily scp it using a
`tasks/deploy` shell script similar to [this deploy script][]. You’d obviously
need to change the values to reflect your site’s details. There is even [a
matching TextMate command][] that will help you run this script.

[this deploy script here]: https://github.com/henrik/henrik.nyh.se/blob/master/script/deploy

[a matching TextMate command]: https://gist.github.com/henrik/214959

### rsync

Once you’ve generated the `_site` directory, you can easily rsync it using a `tasks/deploy` shell script similar to [this deploy script here](https://github.com/vitalyrepin/vrepinblog/blob/master/transfer.sh). You’d obviously need to change the values to reflect your site’s details.

Certificate-based authorization is another way to simplify the publishing
process. It makes sense to restrict rsync access only to the directory which it is supposed to sync. This can be done using rrsync.

#### Step 1: Install rrsync to your home folder (server-side)

If it is not already installed by your host, you can do it yourself:

- [Download rrsync](https://ftp.samba.org/pub/unpacked/rsync/support/rrsync)
- Place it in the `bin` subdirectory of your home folder  (`~/bin`)
- Make it executable (`chmod +x`)

#### Step 2: Set up certificate-based SSH access (server side)

This [process](https://wiki.gentoo.org/wiki/SSH#Passwordless_Authentication) is
described in several places online. What is different from the typical approach
is to put the restriction to certificate-based authorization in
`~/.ssh/authorized_keys`. Then, launch `rrsync` and supply
it with the folder it shall have read-write access to:

{% highlight shell %}
command="$HOME/bin/rrsync <folder>",no-agent-forwarding,no-port-forwarding,no-pty,no-user-rc,no-X11-forwarding ssh-rsa <cert>
{% endhighlight %}

`<folder>` is the path to your site. E.g., `~/public_html/you.org/blog-html/`.

#### Step 3: Rsync (client-side)

Add the `deploy` script to the site source folder:

{% highlight shell %}
#!/bin/sh

rsync -crvz --rsh='ssh -p2222' --delete-after --delete-excluded   <folder> <user>@<site>:
{% endhighlight %}

Command line parameters are:

- `--rsh=ssh -p2222` &mdash; The port for SSH access. It is required if
your host uses a different port than the default (e.g, HostGator)
- `<folder>` &mdash; The name of the local output folder (defaults to `_site`)
- `<user>` &mdash; The username for your hosting account
- `<site>` &mdash; Your hosting server

Using this setup, you might run the following command:

{% highlight shell %}
rsync -crvz --rsh='ssh -p2222' --delete-after --delete-excluded _site/ hostuser@example.org:
{% endhighlight %}

Don't forget the column `:` after server name!

#### Step 4 (Optional): Exclude the transfer script from being copied to the output folder.

This step is recommended if you use these instructions to deploy your site. If
you put the `deploy` script in the root folder of your project, Jekyll will
copy it to the output folder. This behavior can be changed in `_config.yml`.

Just add the following line:

{% highlight yaml %}
# Do not copy these files to the output directory
exclude: ["deploy"]
{% endhighlight %}

Alternatively, you can use an `rsync-exclude.txt` file to control which files will be transferred to your server.

#### Done!

Now it's possible to publish your website simply by running the `deploy`
script. If your SSH certificate  is [passphrase-protected](https://martin.kleppmann.com/2013/05/24/improving-security-of-ssh-private-keys.html), you will be asked to enter it when the
script executes.

## Rack-Jekyll

[Rack-Jekyll](https://github.com/adaoraul/rack-jekyll/) is an easy way to deploy your site on any Rack server such as Amazon EC2, Slicehost, Heroku, and so forth. It also can run with [shotgun](https://github.com/rtomayko/shotgun/), [rackup](https://github.com/rack/rack), [mongrel](https://github.com/mongrel/mongrel), [unicorn](https://github.com/defunkt/unicorn/), and [others](https://github.com/adaoraul/rack-jekyll#readme).

Read [this post](http://andycroll.com/ruby/serving-a-jekyll-blog-using-heroku) on how to deploy to Heroku using Rack-Jekyll.

## Jekyll-Admin for Rails

If you want to maintain Jekyll inside your existing Rails app, [Jekyll-Admin](https://github.com/zkarpinski/Jekyll-Admin) contains drop in code to make this possible. See Jekyll-Admin’s [README](https://github.com/zkarpinski/Jekyll-Admin/blob/master/README) for more details.

## Amazon S3

If you want to host your site in Amazon S3, you can do so by
using the [s3_website](https://github.com/laurilehmijoki/s3_website)
application. It will push your site to Amazon S3 where it can be served like
any web server,
dynamically scaling to almost unlimited traffic. This approach has the
benefit of being about the cheapest hosting option available for
low-volume blogs as you only pay for what you use.

## OpenShift

If you'd like to deploy your site to an OpenShift gear, there's [a cartridge
for that](https://github.com/openshift-cartridges/openshift-jekyll-cartridge).

<div class="note">
  <h5>ProTip™: Use GitHub Pages for zero-hassle Jekyll hosting</h5>
  <p>GitHub Pages are powered by Jekyll behind the scenes, so if you’re looking for a zero-hassle, zero-cost solution, GitHub Pages are a great way to <a href="../github-pages/">host your Jekyll-powered website for free</a>.</p>
</div>

## Kickster

Use [Kickster](http://kickster.nielsenramon.com/) for easy (automated) deploys to GitHub Pages when using unsupported plugins on GitHub Pages.

Kickster provides a basic Jekyll project setup packed with web best practises and useful optimization tools increasing your overall project quality. Kickster ships with automated and worry-free deployment scripts for GitHub Pages.

Setting up Kickster is very easy, just install the gem and you are good to go. More documentation can here found [here](https://github.com/nielsenramon/kickster#kickster). If you do not want to use the gem or start a new project you can just copy paste the deployment scripts for [Travis CI](https://github.com/nielsenramon/kickster/tree/master/snippets/travis) or [Circle CI](https://github.com/nielsenramon/kickster#automated-deployment-with-circle-ci).

## Aerobatic

[Aerobatic](https://www.aerobatic.com) is an add-on for Bitbucket that brings GitHub Pages style functionality to Bitbucket users. It includes continuous deployment, custom domains with a wildcard SSL cert, CDN, basic auth, and staging branches all in the box.

Automating the build and deployment of a Jekyll site is just as simple as GitHub Pages - push your changes to your repo (excluding the `_site` directory) and within seconds a build will be triggered and your built site deployed to our highly- available, globally distributed hosting service. The build process will even install and execute custom Ruby plugins. See our [Jekyll docs](https://www.aerobatic.com/docs/static-generators#jekyll) for more details.

## PubStorm

[PubStorm](https://www.pubstorm.com) is a free front-end and static-site publishing platform built by [Nitrous](https://www.nitrous.io). PubStorm is distributed as a node package and can be installed by running `npm install -g pubstorm`. You can create a free account by running `storm signup`.

To publish your site, run `storm init` from the root of your project and enter `_site` as the project path when prompted. You can the run `jekyll build` to build your site and then run `storm deploy` to publish your site in seconds.

PubStorm offers a pre-configured CDN, free custom domains, SSL certs, rollbacks, collaboration and more. To configure additional features, [follow the instructions on the PubStorm help site](http://help.pubstorm.com).

You can also use the [Nitrous Jekyll Template](https://www.nitrous.io/quickstarts) to develop your Jekyll project and deploy to PubStorm directly from Nitrous. This is a great option for developing Jekyll projects on Windows.
