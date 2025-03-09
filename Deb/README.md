## Packaging software for Debian systems
Whether it’s Debian, Ubuntu, or CentOS, packages are everywhere in Linux. Most of the time, we only “use” them. But sometimes, whether it’s for easily distributing software or for compliance reasons, you have to create one.


A floating package in the clouds — DALL-E
In this article, we are going to discover how to build these packages we so often use. It is also an opportunity to find out more about the ecosystem of packages and how they are managed.

### Packages in the age of containers
The following chapter is my opinion following my experience. Please share your thoughts and experience in the comments!
Although undoubtedly useful for OS core components and libraries, it is normal to wonder why to bother to maintain deb packages (including documentation and dev tooling) for software distribution instead of only dealing with containers.
Debian packages and the Advanced Package Tool (the famous “apt” command) were born at a time when containers didn’t exist. Respectively in 1994 and 1998. LXC then took life around 2008, and Docker popularized this technology back in 2013.
But still, we use apt to download packages. Why? First, because it’s handy. Let’s say I want to install Chromium and don’t want to bother going to the official website to retrieve the installation procedure and manually install the binary. I just have to use the apt-get install chromium-browser command. Also, most packages on official repositories, such as those on packages.ubuntu.com are checked to meet high-quality standards, such as security and maintainability criterias.
Then, because it’s lightweight. A deb package starts from a few kilobytes and averages to dozens of megabytes, while a container image can easily reach hundreds of megabytes. It is really useful for low-bandwidth use cases or frugal hardware, without the need to install additional software such as Docker.
Deb packages have a defined lifecycle for installing, starting, updating, and removing the package. It is quite easy to manipulate as it allows monolith software or tools to be manipulated as any other package. Also, the app starts as programmed by the deb developer, without the need for the deployer to do anything. It is important when your engineers are not skilled to deploy on containerized environments (i.e., Docker Swarm, Kubernetes).
In restricted systems, it is hard to get one package installed without falling into dependency hell or downloading an entire mirror. Alternatives such as snap, flatpak, or AppImage try to solve this issue by self-containing all dependencies into one archive.
Some will even argue that containers are of no use when “properly packaging”. Truth is quite different, as the interest in Docker or Kubernetes goes well beyond installing and deploying software. It is about managing the entire software lifecycle, including hundreds to thousands of micro-services interacting altogether with a set of standardized services (i.e., storage, networking, health checks, services mesh, logs management, security audit…), often across different computers.
And sometimes, the infrastructure you are deploying to just supports deb packages.


### Architecture of a package
All files of this tutorial can be found on flavienbwk/deb-package-tutorial’s repo. There are no pre-requisite but a Debian-based OS such as Ubuntu to create your deb package.

You first need to create a folder with the following nomenclature :

package-name_version_architecture.

Package name : mypackage
Version : 1.0
Architecture: could be all or a subset of supported architectures
Our package will be namedmypackage_1.0_all. It will open Ubuntu’s official website URL in user’s default browser.

Now let’s create some files to have the proper architecture :


```
.
└── mypackage_1.0_all                   # Package main folder
    ├── DEBIAN
    │   ├── control                     # File with package's main info
    │   ├── postinst                    # Script executing after the install
    │   └── preinst                     # Script executing before the install
    ├── opt
    │   └── mypackage                   # Folder including our software
    │       └── open_link.sh            # Script opening browser to ubuntu.com
    └── usr
        └── share
            ├── applications
            │   └── mypackage.desktop   # File with app info in launcher
            └── icons
                └── mypackage.xpm       # Launcher app icon
```

At the root of the package folder — except for the DEBIAN folder — the directory’s architecture mirrors the desired location of the package’s files on the target host. For example, the ./opt/mypackage/open_link.sh file will get copied to /opt/mypackage/open_link.sh when installed.
Our open_link.sh script is simple: it opens the browser of the user to Ubuntu’s official website if it is up and running (curl command) or displays a graphical window on error.


```
#!/bin/bash
url="https://ubuntu.com"
response=$(curl --head --silent --output /dev/null --write-out "%{http_code}" "$url")
if [[ $response -eq 200 ]]; then
    zenity --info --text="You will get redirected to $url"
    exit_code=$?
    if [[ $exit_code -eq 1 ]]; then
        echo "User closed the window. Exiting script."
        exit 1
    fi
    open "$url"
else
    zenity --warning --text="Ubuntu's official website is not available at this moment."
fi
```

The mypackage.desktop file allows to display our app in the launcher so we can graphically click on it.
One important file is DEBIAN/control, as it includes the package's main information:


```
Package: mypackage
Version: 1.0            # package version
Architecture: all       # our package sums up to a bash script and this is POSIX
Essential: no           # essential to the system ?
Priority: optional      # install order in package management system
Depends: curl,zenity    # comma-separated dependency packages (,)
Maintainer: flavienbwk
Description: A sample package...
```

Although there are ways to install deb archives without sudo, most deb packages are designed to be installed system-wide. This means that preinst and postinst scripts or any other binary included in the archive can run without any restriction on one’s system (see Snap packages for an alternative). Triple-check your scripts and be careful when sharing so you don’t break someone’s computer.


### Build and install
It was easy right ? Now let’s build our package:
```
dpkg-deb --build ./mypackage_1.0_all
```


It should have produced a mypackage_1.0_all.deb file. Let’s install it:

```
sudo dpkg -i ./mypackage_1.0_all.deb
```

You should see Mypackage in your launcher :

Install from the APT CLI
The first option is the easiest : we can install packages locally.

1. Create a folder where our repository will be located and move our .deb package inside
```
mkdir -p ./mirror/pool
cp ./mypackage_1.0_all.deb ./mirror/pool/
```


2. Create the Packages index file
```
cd ./mirror
dpkg-scanpackages -m ./pool > Packages

```


3. Add the directory to your system’s sources
```
echo "deb [trusted=yes] file:/path/to/repository/mirror /" | sudo tee /etc/apt/sources.list.d/mypackage.list
```


4. Update your packages definition and install
```
sudo apt update
sudo apt install mypackage
```

Locally-installed repositories can then be served from a simple Apache server on your own machine.
You may choose to create your Personal Package Archive (PPA), hosted on , then accessible from everyone with a simple add-apt-repository ppa:<repository_name> command.
If you want your package to get published into Ubuntu’s universe/multiverse repositories, it may get tricky as you should get the approval of a MOTU. Want to publish it to main? That’s a lot of conditions to meet, including security and commitment to maintenance criteria.


### A word about META packages
META packages are packages that install nothing but a list of dependencies.

That’s how you can install a whole desktop through one package.

### A word about snap packages
APT is the traditional package management system used by Debian and its derivatives (incuding Ubuntu). It debuted in 1998 and uses .deb packages.

Snap, introduced by Canonical in 2014, is a newer package manager designed to provide easier package distribution across different Linux distributions. It bundles dependencies within each .snap package, leading to larger package sizes but mitigating "dependency hell". This comes useful, especially in offline systems.

The key differences is that snap packages focus on cross-distribution compatibility and self-containment, potentially better security through package sandboxing, and automatic updates. APT, on the other hand, relies on system-wide libraries, which makes packages smaller but can cause dependency issues.

### Final word
You have learned how to create a Debian package. Congrats ✨
Show your support by clapping this article and staring the repo ⭐if they were useful to you. Share your thoughts in the comments, I read them.

