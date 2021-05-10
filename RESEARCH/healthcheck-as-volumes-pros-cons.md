# Healthcheck volumes pros/cons
This is a draft for examining the pros/cons of moving away from healthcheck scripts as files during container build and into volumes.

## PROS
* Updates of healthcheck do not require image rebuild: We can simply replace the healtcheck script on the docker servers and restart the target for the new volume to pickup.
* The healthcheck file can be `immutable` if mounted read-only: This can prevent users from replacing the healthcheck script contents. Although, this has not happened as of yet, it is still a concern.
* Can be as self-contained as we want: We can code the checks we want to perform and have all the needed data included into the binary. This can include checksums for files, environment variables etc.

## CONS
* Requires syncing files to the docker servers: This can be somewhat dangerous since we need to find a way to push healtcheck files to the docker servers.
* Breaks away from our concept of self-contained target images: All our images aim to be self-contained in every way and this can cause problems we haven't considered yet.
* It will require a separate repository to maintain the updated healtchecks
* It will require an additional step in the destruction/creation of the target in order to remove the volume. Volumes are persistent across container destruction/creation and so if we dont want to end up with a million volumes, we will have to ensure we destroy previous volumes belonging to the container.
