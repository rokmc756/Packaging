### Build Incus Binary
```
$ make deps

$ export CGO_CFLAGS="-I/root/Incus-Build/incus-6.10/vendor/raft/include/ -I/root/Incus-Build/incus-6.10/vendor/cowsql/include/"
$ export CGO_LDFLAGS="-L/root/Incus-Build/incus-6.10/vendor/raft/.libs -L/root/Incus-Build/incus-6.10/vendor/cowsql/.libs/"
$ export LD_LIBRARY_PATH="/root/Incus-Build/incus-6.10/vendor/raft/.libs/:/root/Incus-Build/incus-6.10/vendor/cowsql/.libs/"
$ export CGO_LDFLAGS_ALLOW="(-Wl,-wrap,pthread_create)|(-Wl,-z,now)"

$ make
```

### Install devscript package to create changelog by dch command
```bash
$ apt install -y devscript
```

### Create changelog file
```
$ cd incus

$ dch -i --create
dch warning: ignoring -a/-i/-e/-r/-b/--allow-lower-version/-n/--bin-nmu/-q/--qa/-R/-s/--lts/--team/--bpo/--stable,-l options with --create
dch warning: neither DEBEMAIL nor EMAIL environment variable is set
dch warning: building email address from username and FQDN

dch: Did you see those 3 warnings?  Press RETURN to continue...

Select an editor.  To change later, run 'select-editor'.
  1. /usr/bin/vim.motif
  2. /usr/bin/vim.gtk3
  3. /usr/bin/vim.nox
  4. /bin/nano        <---- easiest
  5. /usr/bin/vim.basic
  6. /usr/bin/vim.tiny
  7. /usr/bin/emacs
  8. /bin/ed

Choose 1-8 [4]: 5

incus (6.10.0) UNRELEASED; urgency=medium

  * Initial release. (Closes: #XXXXXX)

 -- Jack, Moon <rokmc756@gmail.com>  Mon, 10 Mar 2025 01:10:23 +0900
```

```
$ apt install debhelper-compat

$ dpkg-source --before-build .

$ dpkg-buildpackage
```


### References
- https://john-tucker.medium.com/debian-packaging-by-example-118c18f5dbfe
- https://medium.com/@flavienb/packaging-software-for-debian-systems-19562b077050

