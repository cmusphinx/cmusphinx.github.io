---
layout: post
status: publish
published: true
title: Training CMU Sphinx with LibriSpeech
author:
  display_name: David Huggins-Daines
  email: dhdaines@gmail.com
author_email: dhdaines@gmail.com
excerpt_separator: <!--more-->
---

**Executive Summary: Training is fast, easy and automated, but the
accuracy is not good.  You should not use CMU Sphinx for
large-vocabulary continuous speech recognition.**

The simplest way to train a CMU Sphinx model is using a single machine
with multiple CPUs.  It may not be as cost-effective as a cluster, but
is quite a bit simpler, as all of the cloud HPC cluster solutions seem
to unnecessarily difficult to set up and incredibly badly documented.
This is a real shame, and once I figure out how to use one of them,
I'll write a document which explains how to actually make it work.

To get access to a machine, a good option is Microsoft Azure, though
there are many others.  The free credits you get on signing up are
more than sufficient to train a few models, but after that it can
start to get expensive quickly.  I have tried to optimize this process
so that "resources" (virtual machines, disks) can be used only as long
as they are needed.

First you will need to install software and (possibly) download the
data.  This is going to take a while no matter what, and doesn't need
multiple CPUs, so when you initially create your VM, you can use a
very small one.

## Setting up the software

The Azure portal is slow and unwieldy, and the cloud shell isn't much
better, so it's worth [setting up the Azure CLI
locally](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) -
then log in with:

    az login

We will put everything in a "resource group" so that we can track
costs and usage.

    az group create --name librispeech-100 --location canadacentral
    
Now create the VM (assuming here you already have an SSH public key
that you will use to log in - otherwise omit the `--ssh-key-values`
line):

    az vm create --resource-group librispeech-100 --name compute \
    --image UbuntuLTS --size Standard_B1ls \
    --ssh-key-values ~/.ssh/id_rsa.pub

This will print a blob of JSON with the information for the new VM.
The `publicIpAddress` key is the one you want, and now you can log
into the newly created server with it using `ssh`.  One way to
automate this (surely there are many) is:

    ipaddr=$(az vm list-ip-addresses -g librispeech-100 -n compute \
        --query [0].virtualMachine.network.publicIpAddresses[0].ipAddress \
        | tr -d '"')
    ssh $ipaddr

Run the usual OS updates:

    sudo apt update
    sudo apt upgrade

And install Docker, which we'll need later:

    sudo apt install docker.io
    sudo adduser $USER docker
    
## Setting up data

We will set up a virtual disk to hold the data.  While it's possible
to put it in a storage account, this is unsuitable for storing large
numbers of small files and just generally annoying to set up.  Azure
has cheap disks which don't cost more than network storage and are
still *way* faster, so we'll use one of those.  Log out of the VM,
create the disk, and attach it (75GB is enough for the full 960 hours
of LibriSpeech audio, you could make it smaller if you want):

    az disk create --resource-group librispeech-100 --name librispeech \
        --size-gb 75  --sku Standard_LRS
    az vm disk attach --resource-group librispeech-100 \
        --vm-name compute --name librispeech

The `Standard_LRS` option is very important here, as the default disk
type is extremely expensive.  Now log back in, partition, attach, and
mount it.  The disk should be available as `/dev/sdc`, but this isn't
guaranteed, so run `lsblk` to find it:

    $ lsblk
    NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    ...
    sdc       8:16   0   75G  0 disk 

You can now use `parted` to create a partition:

    sudo parted /dev/sdc
    mklabel gpt
    mkpart primary ext4 0% 100%

And `mkfs.ext4` to format it:

    $ sudo mkfs.ext4 /dev/sdc1
    mke2fs 1.44.1 (24-Mar-2018)
    Discarding device blocks: done                            
    Creating filesystem with 19660288 4k blocks and 4915200 inodes
    Filesystem UUID: e8acc3b5-eec7-441e-96e8-50fa200471bb
    ...

The `Filesystem UUID` (which will not match the one above) is
important, as it allows you to make the disk persistent.  Add a line
to `/etc/fstab` using the `UUID` that was printed when you formatted
the partition:

    UUID=e8acc3b5-eec7-441e-96e8-50fa200471bb # CHANGE THIS!!!
    echo "UUID=$UUID /data/librispeech auto defaults,nofail 0 2" \
        | sudo tee -a /etc/fstab

And, finally, mount it and give access to the normal user:

    sudo mkdir -p /data/librispeech
    sudo mount /data/librispeech
    sudo chown $(id -u):$(id -g) /data/librispeech

I *promise* you, all that was still way easier than using a storage
account.

Now download and unpack the data directly to the data disk:

    cd /data/librispeech
    curl -L https://www.openslr.org/resources/12/train-clean-100.tar.gz \
        | tar zxvf -
    curl -L https://www.openslr.org/resources/12/dev-clean.tar.gz \
        | tar zxvf -

## Setting up training

Make a scratch directory (we are going to save it and restore it when we restart the VM later):

    sudo mkdir /mnt/work
    sudo chown $(id -u):$(id -g) /mnt/work

Now set up the training directory and get a few extra files:

    mkdir librispeech
    cd librispeech
    docker run -v $PWD:/st dhdaines/sphinxtrain -t librispeech setup
    ln -s /data/librispeech/LibriSpeech wav
    cd etc
    wget https://www.openslr.org/resources/11/3-gram.pruned.1e-7.arpa.gz
    wget https://www.openslr.org/resources/11/librispeech-lexicon.txt

Edit a few things in `sphinx_train.cfg`.  It is *not* recommended to
train PTM models for this amount of data, as the training is quite
slow, probably unnecessarily so.

    $CFG_WAVFILE_EXTENSION = 'flac';
    $CFG_WAVFILE_TYPE = 'sox';
    $CFG_HMM_TYPE  = '.cont.';
    $CFG_FINAL_NUM_DENSITIES = 16;
    $CFG_N_TIED_STATES = 4000;
    $CFG_NPART = 16;
    $CFG_QUEUE_TYPE = "Queue::POSIX";
    $CFG_G2P_MODEL= 'yes';
    $DEC_CFG_LANGUAGEMODEL  = "$CFG_BASE_DIR/etc/3-gram.pruned.1e-7.arpa.gz";
    $DEC_CFG_LISTOFFILES    = "$CFG_BASE_DIR/etc/${CFG_DB_NAME}_dev.fileids";
    $DEC_CFG_TRANSCRIPTFILE = 
        "$CFG_BASE_DIR/etc/${CFG_DB_NAME}_dev.transcription";
    $DEC_CFG_NPART = 20;

You will need the scripts from `templates/librispeech` in SphinxTrain,
which unfortunately don't get copied automatically... Check out the
SphinxTrain source and copy them from there or simply [download them
from
GitHub](https://github.com/cmusphinx/sphinxtrain/tree/master/templates/librispeech).

Create the transcripts and OOV list:

    python3 make_librispeech_transcripts.py \
        -l etc/librispeech-lexicon.txt --100 wav

Create dictionaries:

    python3 make_librispeech_dict.py etc/librispeech-lexicon.txt
    
Now we will save this setup to the system disk because we will resize
the VM in order to run compute on more CPUs:

    cd /mnt
    tar zcf ~/work.tar.gz work

Finally pull the Docker image we'll use for training:

    docker pull dhdaines/sphinxtrain

## Running training

We can now run training!  Let's resize the VM to 16 CPUs (you can use
more if you want, but remember to set `$CFG_NPART` to at least the
number of CPUs - also, certain stages of training won't use them all):

    az vm resize --resource-group librispeech-100 --name compute \
        --size Standard_F16s_v2

Log into the updated VM (it will have a different IP address) and
remake the scratch directory:

    ipaddr=$(az vm list-ip-addresses -g librispeech-100 -n compute \
        --query [0].virtualMachine.network.publicIpAddresses[0].ipAddress \
        | tr -d '"')
    ssh $ipaddr
    cd /mnt
    sudo tar xf ~/work.tar.gz

Now run training.  Note that in addition to the scratch directory, we
also need to "mount" the data directory inside the Docker image so it
can be seen:

    docker run -v $PWD:/st -v /data/librispeech/LibriSpeech:/st/wav \
        dhdaines/sphinxtrain run

This should take about 5 hours including decoding (actually the
training only takes an hour and a half...)  You should obtain a word
error rate of approximately 18.5%, which, it should be said, is pretty
terrible.  For comparison, a [Kaldi
baseline](https://github.com/kaldi-asr/kaldi/blob/master/egs/librispeech/s5/RESULTS)
with this training set and language model gives 11.69%, the best Kaldi
system with this language model (but trained on the full 960 hours of
LibriSpeech) gets 5.99%, and using a huge neural network acoustic
model and an even huger 4-gram language model, Kaldi can go as low as
4.07% at the moment.

Of course, wav2vec2000, DeepSpeech42, HAL9000 and company are
somewhere under 2%.

**What's missing from CMU Sphinx?** Well, it should be noted that what
we've done here is the degree zero of automatic speech recognition,
using strictly 20th-century technology.  Kaldi is more parameter
efficient, as it allows a different number of Gaussians for each
phone, and its baseline model already includes speaker-adaptive
training and feature-space speaker adaptation, which make a big
difference.  The Kaldi decoder is also faster and more accurate and
supports rescoring with larger and more accurate language models
(4-gram, 5-gram, RNN, etc).

On the other hand, SphinxTrain is easier to use than the Kaldi
training scripts ;-)

You should, therefore, probably not use CMU Sphinx.

## Saving the training setup

You will probably want to use the acoustic models (in
`model_parameters/librispeech.cd_ptm_5000`) for something.  You may
also wish to rerun the training with different parameters.  The most
obvious solution, provided you have a couple gigabytes to spare (for
librispeech-100 you need about 3G) and a sufficiently fast connction,
is to copy it to your personal machine if you have one:

    ipaddr=$(az vm list-ip-addresses -g librispeech-100 -n compute \
        --query [0].virtualMachine.network.publicIpAddresses[0].ipAddress \
        | tr -d '"')
    rsync -av --exclude=__pycache__ --exclude='*.html' \
        --exclude=bwaccumdir --exclude=qmanager --exclude=logdir \
        $ipaddr:/mnt/work/librispeech .

Another option is to create a file share and mount it:

    az storage share-rm create --resource-group librispeech-100 \
        --storage-account $STORAGE_ACCT --name librispeech --quota 1024
    sudo mkdir -p /data/store
    sudo mount -t cifs //$STORAGE_ACCT.file.core.windows.net/librispeech /data/store \
        -o uid=$(id -u),gid=$(id -g),credentials=/etc/smbcredentials/$STORAGE_ACCT.cred,serverino,nosharesock,actimeo=30

You could also use an Azure Storage blob with azcopy, or other things
like that.  Now copy your training directory and models to a tar file
(no need to compress it):
    
    tar --exclude=bwaccumdir --exclude=logdir --exclude=qmanager \
        --exclude='*.html' -cf /data/store/librispeech-100-train.tar \
        -C /mnt/work librispeech
        
Notice that it is considerably smaller than the original dataset.

## Shutting down

Now you must either deallocate the VM or convert it back to a cheap
one to avoid paying for unused time:

    az vm deallocate --resource-group librispeech-100 --name compute
    # or
    az vm resize --resource-group librispeech-100 \
        --name compute --size Standard_B1ls

Note that in either case *your scratch directory will be erased*.
Note also that you will continue to pay a small amount of money for
the data disk you created (as well as the storage account, if you have
one).  You can make a free snapshot of the data disk which will allow
you to deallocate it and stop paying for it.  I don't know how to do
that, though...
