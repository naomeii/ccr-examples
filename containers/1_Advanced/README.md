# Advanced Containers and GPUs Example

After working with general purpose containers, we might be interested in using containers specifically built for running on GPUs.

NVIDIA provides a [container catalog](https://catalog.ngc.nvidia.com/search?filters=resourceType|Container|container) with some of the most up-to-date, cutting edge codes used for machine learning and artificial intelligence workflows.  We highly recommend perusing this catalog and utilizing these containers if they fit your needs.

In this tutorial, we demonstrate how to locate containers with the software we want to run, pull them, and then run them on CCR's GPU nodes.

## Running Containers on CCR's GPU Nodes
1. Request access to a compute node:

``
$ salloc --partition=general-compute --qos=general-compute --mem=64G --time=12:00:00 --gpus-per-node=1 --no-shell
``

**NOTE**: GPU nodes can be in high demand.  To reduce your wait times, we recommend you reduce the time in this example to 2 hours, unless you plan to use the node for additional testing.  Other alternatives to speed up this process: utilize the debug partition (this has a time limit of 1 hour) and/or increase the number of CPUs requested as Apptainer will make use of more cores.

``--partition=debug --qos=debug --time=1:00:00
--ntasks-per-node=8``  (or 10)

Once the requested node is available, use the `srun` command to login to the compute node:

```
CCRusername@login1:~$ srun --jobid=22150445 --export=HOME,TERM,SHELL --pty /bin/bash --login
CCRusername@cpn-d01-06:~$
```

2. Set the Apptainer cache directory to your job's scratch directory

``
export APPTAINER_CACHEDIR=$SLURMTMPDIR
``

**NOTE**: Downloading and building the container will push you over quota in your home directory, so set the cache directory to your job's Slurm temporary directory (`$SLURMTMPDIR`), which is `/scratch/$JOBID` on the compute node, may result in faster container downloads.  It is also automatically deleted when your job ends.  If your research group is using the same container for multiple builds, you'll want to use your shared project directory for the `APPTAINER_CACHE` location instead. 

3. Pull the specified Pytorch container from NVIDIA:

Use the apptainer pull command, specifying the target container file to be `pytorch.sif` and and the source location where NVIDIA hosts its containers (in this example, we use `docker://nvcr.io/nvidia/pytorch:25.01-py3`).

``
apptainer pull pytorch.sif docker://nvcr.io/nvidia/pytorch:25.01-py3
``

3. Run script in container

Once the download completes, we can test running a simple script with our container.
In this example, we are using `gpu_test.py`, a Python script that prints the version of Python and the name of the available GPU.

``
apptainer exec --nv pytorch.sif python gpu_test.py
``

The output should confirm that you’re using a newer version of Python than the default 3.9 available on the cluster, and that PyTorch can detect the A100 GPU.

4. Verify GPU access with nvidia-smi in container

``
apptainer exec --nv pytorch.sif nvidia-smi
``

## Accessing the GPU within your container:
To access the proper NVIDIA driver and requisite files for running containers on GPU nodes, you will need to specify the --nv flag when running or execing the container.  For example:

`apptainer shell --nv mycontainer.sif` 

or

`apptainer exec --nv mycontainer.sif`

## ARM64 containers:
NVIDIA makes available containers for the `ARM64` CPU architecture. CCR has these available in the [arm64 partition](https://docs.ccr.buffalo.edu/en/latest/hpc/clusters/#ub-hpc-compute-cluster) of the UB-HPC cluster. Research groups interested in utilizing these nodes must [request an allocation](https://docs.ccr.buffalo.edu/en/latest/portals/coldfront/#request-an-allocation) in ColdFront for this partition. When attempting to pull `ARM64` containers (these are named with the suffix `-igpu`) from NVIDIA's container library, you must do this from an `ARM64` node or the build will fail.