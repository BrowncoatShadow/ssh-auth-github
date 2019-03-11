`ssh-auth-github` is a script designed to allow ssh-key authentication using
public keys managed in user's GitHub profile; instead of managing keys in
individual user's `authorized_keys` file. The two components that make this
work are the `AuthorizedKeysCommand` setting in `sshd_config` and GitHub's
public ssh-keys URL.

`AuthorizedKeysCommand` takes an argument for a command that will be run every
time a user attempts to make an ssh connection. It expects an output of 0 or
more authorized public keys, just like the `~/.ssh/authorized_keys` file.

GitHub lists all the authorized public keys for a user using the url
`https://github.com/<USERNAME>.keys`.

By tying these two things together, you can centralize your ssh-key
management. You add or remove a key from your GitHub profile, it adds and
removes access to every machine using ssh-auth-github.

# Installation
`sshd` will only exicute the script if the script and the entire parent
directory path are owned and exclusively writable by `root`. `/usr/bin`
is usually a safe place for this.
```
sudo wget https://raw.githubusercontent.com/BrowncoatShadow/ssh-auth-github/master/ssh-auth-github -O /usr/bin/ssh-auth-github
sudo chmod 755 /usr/bin/ssh-auth-github
```

Next, setup `sshd` to execute the script as the `AuthorizedKeysCommand` while
passing the home directory of the user attempting to log in via `ssh`.
This is done by adding the following lines to `/etc/ssh/sshd_config`:
```
AuthorizedKeysCommand /usr/bin/ssh-auth-github %h
AuthorizedKeysCommandUser nobody
```

Before being able to log in, you will need to configure the GitHub user
who's keys can ssh into your account. This is done by adding one or more
(one per line) GitHub account names to `~/.ssh/github_user`.
```
echo "BrowncoatShadow" > ~/.ssh/github_user
chmod 644 ~/.ssh/github_user
```

Finally, restart `sshd`.
```
sudo systemctl restart sshd.service
```
