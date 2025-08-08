# RHEL-10-Performance-Tuning
This project aims to improve the performance of RHEL 10 using image mode and bootable containers to perform updates.

Hardware used for this project includes Raspberry Pi 4 Model B (8GB) and the UP Squared 7100 Edge (8GB). Both of these allow me to have tested 64 bit ARM and Intel 64 bit CPUs.

## Building a Bootable Image (Same Architecture)
I provide Containerfiles here that you can use, but if you would like to use a different version of RHEL 10 bootc please obtain the correct tags and architecture from the [Red Hat Registry](https://catalog.redhat.com/software/containers/rhel10/rhel-bootc/6707d29f27f63a06f7873ee2?container-tabs=overview) and edit the Containerfile as needed. Please do keep in mind, you will also need to login into registry.redhat.io inorder to pull this images, but because I am installing and removing packages from the container, you will also need to login using the subscription manager with the command `sudo subscription-manager register`.

Now, you can build your container using the follow command:
`podman build -f <Containerfile-Name> -t <Name-of-Container>:<Desired-Tag> <Directory-to-Containerfile>`
> NOTE: If you're in the same directory as you're container file you can just include `.` as the directory.

If the build was successful, you should see the name and tag listed when you type the command `podman images`.
> NOTE: `sudo podman images` and `podman images` may display different images, so remain consistent if you build with or without sudo.

Now, create a new directory called output and follow the directions in this [Red Hat article](https://developers.redhat.com/learning/learn:rhel:build-and-run-bootable-container-image-image-mode-rhel-and-podman-desktop/resource/resources:access-red-hat-container-registry) but use the image you just made in place of what they're using.
> NOTE: There are many types of images you can make, but for this project we will be using .raw images. 

Now, for bare metal installations, you need to copy over the .raw image to an external storage device. First, locate the name of the storage using the `lsblk` command, and it will usually be called sdX, with the X being a letter. Once you know for sure what the name of your external storage is called, run the command `sudo dd if=disk.raw of=/dev/sdX bs=4M status=progress conv=fsync`. **This will overwrite anything that was previously on the storage device. Please ensure that everything you want from the drive is off the drive before running this command.**

Now, if you plug this storage device into a bare metal machine, it should boot into RHEL 10 without issues. For the Raspberry PI, there is are some extra hoops involved in getting RHEL 10 to run on it. You can find more information about what you will need on the [RPi4 Firmware Github Page](https://github.com/pftf/RPi4).


## Getting RHEL 10 bootc on RPi4
The simplest way to do this is by using two forms of storage. The first is a micro sd card, which you can put your UEFI firmware onto using the dd command once you've download it from the [RPi4 Firmware Github Page](https://github.com/pftf/RPi4). You can put that into the raspberry pi, then do a similar process with your OS but on a different external storage device that you'll plug in via USB. After that, you should be all set.
> NOTE: You may need custom mount points. You can find more details about that [here](https://qiot-project.github.io/blog/rhel9-on-arm8/)

## Building a Bootable Image (Different Architecture)
This is generally going to be the same process as above, however there is one key difference in that you need an extra argument when building the image in order to specify what architecture you're using, since by default podman will use whatever architecture is on your machine. The new command you should use is:

`podman build --arch <Desired-Architecture> -f <Containerfile-Name> -t <Name-of-Container>:<Desired-Tag> <Directory-to-Containerfile>`

Under desired architecture, the options you would have for RHEL 10 bootc images are amd64, arm64, ppc64Ie, and s390x. It's important to note that you may need to install extra dependencies on your machine before this build is successful, however that depends on what you already have installed.

## Updating via Image Mode (Online)
I recommend using a remote quay.io repo in order to streamline the process. If you've built the original container image with this quay.io repository, then you can push your updates of that container to the quay.io repo and then use `bootc upgrade`. If you would like to use a different repo for your upgrade, you can look at the `--transport` option for bootc switch. 

## Updating via Image Mode (Offline)
I've written a blog on this and will link to it once its been published.
> TBD August 13

### Configuring Ramalama using Image Mode Updates
The container file I provided will generally work as is, but if you're using it to build a container that is being updated to, then you'll need to move some files around. This is because Ramalama's default location saves files within the a directory contained within /var, but when preforming an Image Mode update, /var isn't overwritten. This can be worked around with just making a temporary location to store our models, and then move those models over to the correct location for Ramalama to find them while you're booted into the system.
- The default directory structure for Ramalama is found at /var/lib/ramalama
- The containerfile we've created stores the model within a copy of that directory structure, within /opt/lib/ramalama
- Once you've deployed the update and booted into the system, move both the model **and** blob from /opt/lib/ramalama to the same directory within /var/lib/ramalama. You should be able to find the names of the directories using ls, as the exact names will vary depending on the format of the model (Ollama, OCI, etc.).
