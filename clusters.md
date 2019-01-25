# Clustering (not the galaxy type)

*lights fade, Dean Winters shows up in an Allstate commercial*: "What's that? Your little dinky Macbook Air sounds like a 747 when you leave it running overnight to process your data, keeping you awake all night so that the next morning you snap during group meeting when you get asked what's the holdup on that plot? Oh wait, you just figured out the calculation you want to make takes 200 years to run on your laptop over your 1000s of galaxies?"

*cut to black, woman's cool voice-over that they use at the end of drug commercials*: "If you exhibit any of these symptoms, talk to your advisor about **clustering**."

## What is a cluster?

Basically, think of it like a whole bunch of hard drives strapped together on the same system. The basic unit, a single computer, is called a **node**, and the more nodes you have the more computing power you get -- it **scales**.

There are three different ways that the hardware can respond to changing conditions (i.e. you, the user, making the system do stuff):
* **Fail-over**: several nodes connected via a **heartbeat** connection, which is used to monitor how well the system is handling the stress. If one machine fails, the others try to take over.
* **Load-balancing**: when a request from a user comes in (ex. you try to SSH in), the cluster checks which machine is least busy and sends the request to that. Usually they also have features from fail-over.
* **High performance computing (HPC)**: extra bells and whistles to optimize performance for data centers. Has features of the other two. Usually enables parallelization of tasks.

Most of what you'll be interacting with will be HPC (e.g. Brazos Cluster, SLAC, TACC). Load-balancing and fail-over tend to be used for webfarms, databases, or firewalls.

## What about supercomputers?

Increasingly the line is being blurred between clusters and supercomputers, especially as clusters have begun to have more and better individual nodes and improved connection both node-node and node-world. Nearly all of the techniques you will use on a cluster will carry over onto a supercomputer, so it's a little moot from your perspective anyhow.  Just call/cite the system however they want to be (same rule as not calling a pirate's ship a "boat").

For the purposes of the rest of this document, I will just refer to clusters.

## Why do I need a cluster?

I alluded to this earlier, but there are three main reasons why people use clusters:
* They have a code that benefits from parallelization, either because
    * it is **designed to benefit** from parallelization (less common but possible, e.g. `emcee` has this functionality. You will know if this is the case)
    * they have **adapted their scripts** to run many independent analyses in parallel (more common, e.g. using a bash/scheduling script to run many occurrences of a Python script with different data or inputs).
* They are using data or software that is only located on the cluster (e.g. working with particle data in a cosmological simulation). This could mean running analysis scripts on the cluster itself (which may or may not require scheduling) or simply using shell commands to interact with the data
* They are running a "computationally expensive" analysis (air quotes because everything is relative). This can either be due to physical constraints (too much RAM needed for laptop to handle) or simply because they don't want their laptop running for 3 days straight and would rather have a single node on a cluster be doing that work instead

There is a common misconception that simply throwing one's code onto a cluster and hitting run will make things "work faster." While this may be true (especially if the person's main computer is old/slow), keep in mind that the gains will not be **too** significant (maybe a factor of a few).  Single nodes on a cluster are usually not *that* much more powerful than, say, a new-ish laptop.

## How do I use a cluster?

There are two modes to operate in when you are using a cluster:
* **interactive**: typically, this done via SSH-ing into the cluster and then interacting with it as you would any other computer. For example, I can: SSH into a server, run a quick Python script that grabs some of the useful data that is on the server and converts it to a .csv, then un-SSH and use SCP to download the file to my local machine
* **non-interactive**: typically handled via scheduling (see next section). You may SSH into the cluster, run something like `bsub job.lsf` (whatever the command is to schedule a job), and then be able to *completely disconnect* from the cluster and your code will still run (once it is told to do so by the scheduler)

Note that SSH-ing into a cluster and running `python really-long-script.py` is ***still interactive*** and is subject to all the same rules that SSH-ing is (timeouts, maintaining a live Internet connection, etc.). There are ways around some of these things (`SystemAliveInternal` will help with timeouts), but if you are running a long script on a cluster you probably should be using whatever job scheduler is available.

## Wait, what's SSH-ing?

SSH, or Secure Shell, is a protocol for doing network operations securely over an unsecure network. More bluntly, it's how you can access clusters (or more generally, any computer) that is not the one you are typing at. There's a lot to get into with SSH, but the main takeaways are:
* Generating long, complicated keys. Give your public key to anyone, ***NEVER HAND OUT YOUR PRIVATE KEY***. It's easier and usually more secure than passwords (some clusters still use passwords anyways)
* Once the initial setup is complete, you can run `ssh userid@server` (entering a password every time if using passwords) to connect
* Modify your `~/.ssh/config` file to easily save access or preferences
* Once you've connected, it's as if you had opened up a terminal on that computer. You can `ls`, `pwd`, or anything else to your heart's content

Relatedly, SCP (secure copy protocol) lets you copy files from one computer/cluster to another. It piggy-backs on SSH to do this.

Here's an example SSH config file.
```
### default for all ##
Host *
     ForwardAgent no
     ForwardX11 no
     ForwardX11Trusted yes
     User nixcraft
     Port 22
     Protocol 2
     ServerAliveInterval 60
     ServerAliveCountMax 30\

## Login AWS Cloud ##
Host aws.apache
     HostName 1.2.3.4
     User wwwdata
     IdentityFile ~/.ssh/aws.apache.key

## Login to internal lan server at 192.168.0.251
Host uk.gw.lan uk.lan
     HostName 192.168.0.251
     User nixcraft
```

## What is job scheduling?

Ok, remember how if you SSH into a load-balancing cluster the system will hook you up into whichever login node is least busy? Same concept, but for jobs that you make the system execute. This is what you should do for any scripts that take a long time or consume a lot of resources (memory/CPU hours).

Typically, a job scheduler will have commands you can type at the command line to add a job, check the jobs status (in queue/scheduled/running/done), list what jobs you have active, and your current usage (e.g. the total memory the jobs you are running right now are taking up).

Unfortunately, there seems to be a wide variety of possibe job schedulers out there, each with their own command scheme. You'll just need to find documentation for your specific cluster and read up on that specific scheduler.

**NOTE**: Usually a cluster that has a job scheduler has certain limits on what interactive login nodes are allowed to do. If you exceed a time/memory/CPU limit, the cluster will disconnect you from the system and will wipe whatever script you were running at the time. **Use the job scheduler**.

## What does a scheduling script look like?

Here's an example LSF batch script that I ran on the SLAC server a few times for my Fermi project.

```csh
#!/bin/csh
setenv RUN "moon_bpl"

#BSUB -J moon_bpl
#BSUB -W 60:00
#BSUB -oo /u/ki/strigari/debris_disk/lsf_outputs/%J.output
#BSUB -u alexriley@tamu.edu
#BSUB -N

cd $HOME
source setup-pass8.csh
echo "\n\n"

cd debris_disk/eridani_sim/$RUN
gtobssim pars=$RUN.input.xml
```

The script does the following:
* Establishes which shell I'm running in with `#!/bin/csh` (I probably could have done this in bash, I imagine many/most systems support both)
* Set up the RUN variable, which I use to tell the script which files to run and where to store the outputs
* Sets up the parameters for the job with the `#BSUB` commands. These include the name of the job, how many CPU clock hours its allowed to take, where to put the LSF outputs (essentially status updates and any print statements), and I tell it to send me an email when the job is complete
* The rest is normal stuff you can find in a shell script that is meant to run a program. In this case, I run a script that does all the setup for the Fermi ScienceTools (`source setup-pass8.csh`), then run the function `gtobssim` which simulates a gamma-ray observation based on the parameters file I passed to it

The main part of the script was the `#BSUB` commands, which told the LSF scheduler the parameters it was allowed (CPU time, memory allocation, etc.).  Based on those parameters, it placed my job into the queue, eventually executed it, then sent me an email when it was done!

## Sounds great! Looking forward to my infinite usage of Brazos/SLAC/TACC!

Lol nope. Just imagine if everyone abused the crap out of their clusters, scheduling ridiculously high-resolution simulations and putting a copy of the full SDSS DR15 in each subdirectory on each node.

There are a few limitations you need to keep in mind when scheduling jobs:
* **Storage**: typically there are two numbers to keep in mind here. Your **home** quota is how much data you're allowed to store in your home directory (the directory you are placed into by default when you log on). This is typically backed up on a frequent basis and is meant for code and small amounts of important data. There may be an additional **data** directory that is more generous and meant for storing data that doesn't necessarily need to be backed up. On the COSMA system in Durham, my home quota is 5 GB and is backed up each night, while my data quota is 5 TB and is not backed up regularly
* **Time**: The other main way maintainers regulate usage is by giving each user a fixed quota of CPU hours. This may be set by how much time you got in your proposal and is very fixed (TACC), or something that your advisor can request for more and be granted fairly easily (Brazos)
* **Memory**: upper limit typically set by the system. The job scheduler will let you know if you exceed this limit (usually by crashing)
* **CPU**: once again, typically set by the system.  You probably won't reach it unless you're doing something you weren't supposed to

All of these limits will be available either in documentation or in emails that you get when your account is being set up. Just be careful whenever you are setting up your jobs: if you use up all your computing time from a proposal you probably won't get it back!
