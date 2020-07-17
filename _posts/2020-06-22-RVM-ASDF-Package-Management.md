---
layout: default
title: RVM to ASDF Package Management
---

## RVM to ASDF Package Management

I got into `Ruby` development a few years back.  To manage `Ruby` packages, I had to use either `rbenv` or `rvm`.  For reasons I forget, I went with `rvm` and haven't looked back since.  Fast-forward to today, I'm using a lot more tools now, so my `.bash_profile` and `.bashrc` config files are starting to balloon.  It's still semi-neatly managed, but there has to be a better way.

I'm trying out this tool called `asdf-vm`.  See [asdf](https://asdf-vm.com/) for more information.  It's going to manage all my tools for me, and it's highly extensible so it'll be useful for more than what's currently available today.  It's also opensource, so anybody can add new stuff to it.

What I really like about this tool is you can customize the specific versions for tools per directory.  This is great if you're working on multiple projects and need a way to quickly switch between tooling contexts.


### Uninstall RVM
If you're coming from an existing installation of RVM, uninstalling it is very simple and straightforward.  Run the `rvm implode` command and it will delete itself, which basically just does an `rm -rf on the ~/.rvm`.  There might be some residual configurations laying around in `.bash_profile` and `.bashrc`, so you'll have to clean that up yourself.  

### Install ASDF
Installing this new tool was also very simple an straightforward.  Visit their home page for detailed instructions.  https://asdf-vm.com/#/core-manage-asdf-vm

Don't forget to source the `asdf` binary and the `asdf` shell completion files in your `.bashrc` file too.


### Configure Ruby

Once installed, you have to add the appropriate plugin into the tool with the following:

```bash
asdf plugin add ruby
```  

The plugins tell `asdf` where and how to install the different versions of tools.  You'll need to install a specific version of ruby now.

```bash
asdf install ruby 2.7.1
asdf global ruby 2.7.1
```
The global part isn't needed until you have multiple versions of tools installed.

### Where does everything go?

If you're like me, you want to know where things are getting installed to.  

Take a look at the `~/.asdf/installs` folder.  

```bash
du -h -d 1 ~/.asdf/installs/ruby
```
You should see all the different versions of packages you've installed.


### Test Environment
I've tested this on RHEL 7.5 and Fedora 32.  YMMV with other platforms.

Ruby 2.7.1 was installed already, but let's test multiple versions at once.

First, let's check what we have.
```bash
ruby --version
```
```text
ruby 2.7.1p83 (2020-03-31 revision a0c7c23c9c) [x86_64-linux]
```

Now create a folder with a local `.tool-versions` file that uses an older version of Ruby.  If you go too far back in version history, there will be other package dependencies needed.

```bash
mkdir test-older-ruby
echo "ruby 2.7.0" >> test-older-ruby/.tool-versions
# You could also run it via cli in the test-older-ruby folder
# asdf local ruby 2.7.0

cd test-older-ruby
ruby --version
```

```text
ruby 2.7.0p0 (2019-12-25 revision 647ee6f091) [x86_64-linux]
```

### Next
There's a bunch of great cli features that make interacting with `asdf` easier.  I'll be interested in testing out the different tooling options such as `istioctl` and `kubectl` in the future.
