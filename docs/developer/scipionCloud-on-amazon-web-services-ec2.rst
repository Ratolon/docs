.. figure:: /docs/images/scipion_logo.gif
   :width: 250
   :alt: scipion logo

.. _scipionCloud-on-amazon-web-services-ec2:

======================================================
ScipionCloud on Amazon Web Services EC2
======================================================

Creating an instance (virtual machine)
======================================

Depending on your computing needs, you can start ScipionCloud on a single node
("instance", in Amazon terminology).

Amazon EC2 offers a wide range of instance types, that can be check
on `AWS instance types <https://aws.amazon.com/ec2/instance-types/>`_.

* Connect to `AWS console <https://aws.amazon.com/>`_
* Open the EC2 dashboard
* Launch a ScipionCloud instance: Enter "Instances" panel, then click "Launch instance". Follow the wizard filling in the required information:

  * There is an ScipionCloud AMI but it is private and can only be used for non-commercial purposes. If you wish to use it please write an email to bcu-sysadmins@cnb.csic.es.
  * Choose the instance type, according to how much resources (CPU, RAM, GPU, disk) you want to use. This AMI has been tested on g4dn types, other GPUs instances types might not work straight away. Contact us if you need advise on this.
  * Regarding disk, you can use an integrated SSD disk, or attach an EBS Volume. Press "Next: Configure instance details".
  * Configure instance: leave everything as default unless you know what you are doing.
  * Add storage: specify the disk size of the system disk (that you can use also for data). Here you can also create a new volume of arbitrary size.
  * Tag instance: Add tag 'Name' and give a descriptive name.
  * Configure security group: To use the remote desktop (web based), you need to open ports 80 (HTTP) and 22 (SSH):
  * Click "Review and Launch", then "Launch"
* In the "Select an existing key pair or create a new key pair" panel, select "Create new key pair" (unless you have your own), type the key pair name and click "Download" key pair. Finally, click "Launch instances" and then "View instances"

Select your instance within the list of instances, and copy its public IP address. You will need this address to connect to the instance, whether via ssh or Remote Desktop (VNC).

Wait until the instance is initialized.

Accessing an instance
======================

First, connect with SSH to your instance:

.. code-block:: bash

    chmod 600 path/to/key.pem
    ssh -i path/to/key.pem ubuntu@12.34.56.78 #(use your IP)

Instances created from AMIs need some time to 'warm up' so performance might be poor at the begining (explanation `[here] <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-initialize.html>`_). If you want to avoid it you could follow instructions there.

Access the remote desktop from a browser with the URL and password provided in the console.

This is how the remote desktop will look like:

.. figure:: /docs/images/cloud/noVNC-desktop.png
   :align: center
   :width: 800
   :alt: noVNC-desktop

To disconnect from the session click on the little arrow that appears on the left (see menu below) and click on the last option:


.. figure:: /docs/images/cloud/noVNC-menu.png
   :align: center
   :width: 250
   :alt: noVNC-menu

IMPORTANT!! Do not disconnect from the top right corner, as in a physical machine. If you do so the machine will need to be reboot

There is a shortcut for Scipion on the desktop.

The following software is installed on the machine:

* Ubuntu 20.04
* Scipion on /usr/local/scipion3 (alias scipion3): git installation branch release-3.0 with the following EM plugins:

  * Gctf 1.18
  * Gautomatch 0.56
  * Eman 2.91
  * Motioncor2 1.4.2
  * Relion 3.1.2
  * Spider 26.06
  * Xmipp 3.21.06.1
  * chimeraX 1.2.5
  * cryolo 1.7.6
  * cryosparc2 3.2.0

* Nvidia driver version 460
* CUDA 10.1
* TurboVNC 2.2.6 on
* anaconda3 
* noVNC

Managing instances
====================

* Login to `AWS console <https://aws.amazon.com/>`_
* Go to EC2 services
* Select Instances on the left side menu
* Select the instance, click on ‘Instance state’ and the action required.

  * Stop: the instance is turned off, but everything related to it is kept (IP address change unless you use Elastic IPs). You can power it on again with the Start command. While an instance is off, you are only charged for disk use.
  * Terminate: the instance is deleted, and all non-permanent storage disappear. You should make sure all your data is safe before terminating an instance.
  * Change instance type: This can be useful when running EM workflows since one could start with a small machine for the preprocessing steps, or even with a GPU machine if needed, and then switch to a more powerful machine with higher memory for the classification and refinement steps. In order to change the type the VM needs to be stopped first, then click on "Actions" and select "Instance settings / Change instance type".

Using external EBS volumes
==========================

Latest ScipionCloud image has a default disk of 75 Gb, which is clearly insufficient for storing real processing EM data.
When creating a Virtual Machine through the EC2 console, it is possible to specify a bigger disk for the VM, but you have to take into account that this disk cannot be resized and will disappear when the VM is terminated.
To avoid this problem it is a good practise to work with external EBS volumes attached to the VM, which can be used to store data and/or Scipion projects.

For a single instance of Scipion you can attach an EBS volume when creating the Virtual Machine from the EC2 console as explained on the section above.

Then log in the machine and follow these instructions:

* If the EBS volume has not been formatted run (assuming your EBS volume is attached on /dev/sdf device):

.. code-block:: bash

    sudo mkfs -t ext4 /dev/xvdf`

* Mount the EBS disk

Create the folder where you would like to mount the disk (for instance /data).

.. code-block:: bash

    sudo mount /dev/xvdf /data`

You could also create the EBS volume once the VM is up and running and attach it.
Go to the EC2 console and click on Elastic Block Store / Volumes, select Create Volume and choose size and the same Availability region where the VM is running.
Once the volume is created select it and choose Attach Volume action. Select the VM to which the volume will be attached and device (for instance /dev/sdf).

Then we can proceed with the same instructions as explained above.

Costs on AWS
============
Using ScipionCloud on AWS will have the following costs:

Computing (instances)
----------------------
AWS instance types prices can be found `here <https://aws.amazon.com/ec2/pricing/on-demand/>`_. This is charged by minute only when the instance is running.

Storage
-------
As described above on the `Using external EBS volumes` section, it is recommended
to attach an EBS volume big enough to store raw data and project.

EBS storage costs 0.11$ per GB (2021) which makes around 113$ per TB per month.

If the amount of movies to be processed require many TBs there are some
strategies to reduce the bill:

* Process movies locally and transfer only micrographs to the cloud.
* Transfer movies to a big EBS but as soon as they are aligned use a smaller EBS disk to continue processing (you could even have two disks, one for raw data and one for project and discard the first one when movies are aligned). You then should be certain that movies will not be required afterwards or you will have to transfer them again (it could compensate anyway if they are needed only at the end of processing). Another possibility will be to move movies from EBS to S3 or Glacier (another cheaper storage on AWS) while you do not need them and retrieve them if needed again.
* Process on streaming: use Scipion streaming mode to process movies as they are transferred. You should take care of removing movies from disk as they are processed since Scipion will not do it.

Transfer data
-------------
AWS does not charge for uploading data into the cloud but it does for downloading
data from it.
First GB per month is free but then it costs 0.09$ per GB (up to 10 TB,
then price slowly decrease). (2021)

For a more detailed evaluation of costs and performance you could have a look at
paper `ScipionCloud: An integrative and interactive gateway for large scale
cryo electron microscopy image processing on commercial and academic clouds <https://doi.org/10.1016/j.jsb.2017.06.004>`_.

Also, AWS has the following `tool <https://calculator.s3.amazonaws.com/index.html>`_ to estimate your costs beforehand.

