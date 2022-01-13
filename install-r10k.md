# Install and Configure R10k

Install r10k via Ruby Gems.

```
/opt/puppetlabs/puppet/bin/gem install r10k
```

Configure r10k by creating the following directory structure and file `/etc/puppetlabs/r10k/r10k.yaml` and ensuring it has the following contents:

```
# The location to use for storing cached Git repos
:cachedir: '/var/cache/r10k'
# A list of git repositories to create
:sources:
  # This will clone the git repository and instantiate an environment per
  # branch in /etc/puppetlabs/code/environments
  :my-org:
    remote: 'git@github.com:$_Insert GitHub Organization Here_$/$_Insert GitHub Repository That Will Be Used For Your Puppet Code Here_$'
    basedir: '/etc/puppetlabs/code/environments'
```
# Configure Puppet Code Repository

Populate the repository by cloning it locally and performing each of the following actions within it:

Note that puppet defaults to the `production` environment. You may wish to change your default git
branch from `master` to `production` in order to match this. Alternatively, you can set your agents'
environment to `master`.

```
mkdir -p {modules,site/profile/manifests,hieradata}
touch hieradata/common.yaml
touch site/profile/manifests/base.pp
touch environment.conf
touch Puppetfile
touch site.pp
```

Edit the `environment.conf` file and ensure it has the following contents:

```
manifest = site.pp
modulepath = modules:site
```

Edit the `site.pp` file and ensure it has the following contents:

```
hiera_include('classes')
```

Edit the `hieradata/common.yaml file and ensure it has the following contents:
```
---
classes:
 - 'profile::base'

ntp::servers:
  - 0.us.pool.ntp.org
  - 1.us.pool.ntp.org
```
Edit the `Puppetfile` file and ensure it has the following contents:
```
forge 'forge.puppetlabs.com'

# Forge Modules
mod 'puppetlabs/ntp', '4.1.0'
mod 'puppetlabs/stdlib'
```
Edit the `site/profile/manifests/base.pp` file and ensure it has the following contents:
```
class profile::base {
  class { '::ntp': }
}
```
Ensure that the user r10k runs as (typically root) can access the git
repository. See the [git environment guide](git-environments.mkd)
for more detail.  You can test
the access by using su/sudo to perform `git clone yourrepoURL` as the correct
user.
# Summary
We now have the following functional pieces:
1. Puppet master
2. Hiera
3. r10k
4. Puppet code repository
5. Initial 'profile' named 'base' that will configure NTP on our servers.
This base will allow us to do all sorts of useful things. Most interesting (to me and for the purposes of this tutorial) is the ability to now utilize Git branches to help manage infrastructure as part of your software development lifecycle. Now, when you want to test a new profile, you can do the following:
1. Create a new branch of the Puppet code repository
2. Create your Puppet code in this new branch
3. Push the new branch up to the repository
4. Deploy it as a new environment using the `/opt/puppetlabs/puppet/bin/r10k deploy environment -p` command.
From any agent node (including the master), you may run the agent against the new environment by specifying it on the command line. For example, if you create the branch `test`, run puppet as:
```
puppet agent -t --environment test
```
You can also modify the `/etc/puppetlabs/puppet/puppet.conf` file on a node and add the environment setting to the agent section to make the change permanent:
```
...
[agent]
environment = test
```
Voila - you're testing code without impacting your production environment!
