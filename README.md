# RHEL-10-Performance-Tuning
This project aims to improve the performance of RHEL 10 using image mode an bootable containers

Hardware used for this project includes Raspberry Pi 4 Model B (8GB) and the UP Squared 7100 Edge (8GB). Both of these allow me to have tested 64 bit ARM and Intel 64 bit CPUs, and container files with directions on how to build and use them will be included at a later point in time.

NOTE: These directions are a work in progress and are subject change as the project develops further.

## Building a Bootable Image
I provide Containerfiles here that you can use, but if you would like to use a different version of RHEL 10 bootc please obtain the correct tags and architecture from https://catalog.redhat.com/software/containers/rhel10/rhel-bootc/6707d29f27f63a06f7873ee2?container-tabs=gti and edit the Containerfile as needed.
Please do keep in mind, you will also need to login into registry.redhat.io inorder to pull this images, but because I am installing and removing packages from the container, you will also need to login using the subscription manager with the command `sudo subscription-manager register`.

Now, ensure you're in the directory of your Containerfile and build the image using:
`podman build -t name-of-container:desired-tag .`

If the build was successful, you should see the name and tag listed when you type the command `podman images`.
NOTE: `sudo podman images` and `podman images` may display different images, so remain consistent if you build with or without sudo.

Now, create a new directory called output and follow the directions in this article but use the image you just made.
NOTE: Create a raw image, we will use it in the next step.

Now, for bare metal installtions, you need to copy over the .raw image to an external storage device. First, locate the name of the storage using the `lsblk` command, and it will usually be called sdX, with the X being some sort of letter. Once you know for sure what the name of your external storage is called, run the command `sudo dd if=disk.raw of=/dev/sdX bs=4M status=progress conv=fsync`. Note that this will overwrite anything that was previously on the storage device.

This should create a version of RHEL 10 you can boot into and can be updated with image mode. Updates can now be delivered via image mode.


## Updating via Image Mode
TODO: Add these steps in.
