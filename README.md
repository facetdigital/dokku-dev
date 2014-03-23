# Dokku-Dev

This project contains the basics and some helper scripts to get you started using [Dokku](https://github.com/progrium/dokku) on your local development machine using a VirtualBox VM controlled by Vagrant, to get your webapp project being close to ready for deployment to Heroku.

We assume you have already installed [VirutalBox](https://www.virtualbox.org/) and [Vagrant](http://www.vagrantup.com/) on your Mac or Unix machine. If you are using Windows, sorry, you're on your own.

## Setting up the Dokku Dev VM

* Checkout this repository. For this example, we'll assume a `~/src` directory:

  ```bash
  $ cd ~/src
  $ git clone git@github.com:facetdigital/dokku-dev.git
  ```

* Move into the project directory:

  ```bash
  $ cd ~/src/dokku-dev
  $ ./dokku-create
  ```

* When this finishes, you have a Ubuntu VirutalBox VM running Dokku, managed by Vagrant. You can now manage this with commands such as:

  ```bash
  $ vagrant ssh       # log into the VM
  $ vagrant halt      # stop the VM
  $ vagrant up        # start it back up
  $ vagrant destroy   # completely destroy the VM
  $ ./dokku-create    # re-build VM after destroying it
  ```

* This VM is set to have the name `dokku.me`, which should always resolve to `127.0.0.1`, as should all `*.dokku.com` URLs. The VM will be port-mapped to listen for HTTP connections on port `8080` and hosts all your apps in Dokku containers, routing requests to them via virtual hosting, using domain names like: `myapp.dokku.me`.

* Your development host will also be setup to ssh into the VM as the `dokku` user to issue Dokku commands, by adding something like the following to your `~/.ssh/config` file:

  ```
  ########## Dokku ##########
  Host dokku.me
    HostName 127.0.0.1
    User dokku
    Port 2222
    UserKnownHostsFile /dev/null
    StrictHostKeyChecking no
    PasswordAuthentication no
    IdentitiesOnly yes
    LogLevel FATAL
    RequestTTY yes
  ```

* This means you can `ssh dokku.me help` from anywhere on your system (not just this directory) to issue a Dokku command such as `help` to the VM.

* It also means you can now deploy to Dokku using the appropriate `git push` and remote repository reference to `dokku.me` (more on this later for the unfamiliar).

## Individual App Project Setup

If your project is not already setup for a system like Heroku or Dokku, there are a few things you may need to do.

### Procfile
Most notably, you will need a `Procfile`. For example, for a Ruby rack-based app, you will want to make sure you have a `Procfile` that looks something like this:

```yaml
---
web: bundle exec rackup config.ru -p $PORT
```

Google Heroku's docs for more information on Procfiles.

### Buildpacks
Dokku can auto-detect most of the buildpacks that Heroku does, so for a vanilla app, you probably don't need to worry about this. If you are used to setting your own buildpack on Heroku with the `--buildpacks` flag, you'll want to do that in your project by creating a `.env` file that looks something like this:

```bash
export BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
```

That's an example of using the `heroku-buildpack-multi` which lets you explicitly specify one or more buildpacks to use.

### Multiple Buildpacks
If you need multiple buildpacks, you can use the `.env` shown above, and also include a `.buildpacks` file in your repository that lists out the buildpacks to use, using a format like this:

```
https://github.com/facetdigital/heroku-buildpack-libsodium.git
https://github.com/heroku/heroku-buildpack-ruby.git#v102
```

Google the `heroku-buildpack-multi` project for more instructions on this file format.

### Setup for Git-based Deployment
Like Heroku, Dokku uses a git-based deloyment mechanism. You can use our helper script for setting the remote upstream in your repository to do this, by doing:

```bash
$ cd ~src/myapp
$ ~/src/dokku-dev/dokku-setup-repo myapp
```

This will add a remote named `dokku` to your `.git/config` that you can push to in order to deploy your app.

### Convenience Helpers
We have provided a script to enable a few convenient helper alias into a running terminal session. To load these, you can:

```bash
$ source ~/src/dokku-dev/dokku-env
```

This will give you the following commands _in your current terminal session_:

* `dokku` - An alias to remotely execute `dokku` on the VM via ssh, such that you can enter commands such as `dokku logs myapp` or `dokku help` when your VM is running.

* `dokku-deploy` - An alias for `git push dokku HEAD:master`, which deploys the `HEAD` of _the current working branch_. The first time you use this, it will create your app on the Dokku Dev VM. After that, it will just deploy. Your app will be accessible from your host machine at http://myapp.dokku.me:8080 (with a subdomain named for your app).

### Set Custom App Environment Variables
If you need to set custom environment variables for your app, you can do so with the command:

```bash
$ dokku config:set myapp NAME=VALUE
```

For example, you might want to set variables like:

```bash
$ dokku config:set myapp RAILS_ENV=development DATABASE=10.0.2.2
```

_NOTE: If you want to do this in a script, you'll need to replace `dokku` with `ssh dokku.me`, because your shell aliases won't apply to the script._
