# Advanced Containers and GPUs Example

After working with general purpose containers, we might be interested in using containers specifically built for running on GPUs.

NVIDIA provides a [container catalog](https://catalog.ngc.nvidia.com/search?filters=resourceType|Container|container) with some of the most up-to-date, cutting edge codes used for machine learning and artificial intelligence workflows.  We highly recommend perusing this catalog and utilizing these containers if they fit your needs.

In this tutorial, we demonstrate how to locate containers with the software we want to run, pull them, and then run them on CCR's GPU nodes.

## Running Containers on CCR's GPU Nodes
1. Request access to a compute node:
``
salloc --partition=general-compute --qos=general-compute --mem=64G --time=12:00:00 --gpus-per-node=1
``
**NOTE**: GPU nodes can be in high demand.  To reduce your wait times, we recommend you reduce the time in this example to 2 hours, unless you plan to use the node for additional testing.  Other alternatives to speed up this process: utilize the debug partition (this has a time limit of 1 hour) and/or increase the number of CPUs requested as Apptainer will make use of more cores.
``--partition=debug --qos=debug --time=1:00:00
--ntasks-per-node=8``  (or 10)

2. Once on the compute node, we'll set the Apptainer cache directory to our job's scratch directory and pull down a Pytorch container from NVIDIA:
- Pulling a container from NVIDIA notes: we need to be careful with getting containers from NVIDIA, 
if you go over to the Tags section, the suggested URI's architecture must match the node, `arm64`containers must use the `arm64` partition

the `general-compute` partition allows only `AMD64`

- Before pulling: set the Apptainer cache directory
``
export APPTAINER_CACHEDIR=$SLURMTMPDIR
``

**NOTE**: Downloading and building the container will push you over quota in your home directory, so set the cache directory to your job's Slurm temporary directory (`$SLURMTMPDIR`), which is `/scratch/$JOBID` on the compute node, may result in faster container downloads.  It is also automatically deleted when your job ends.  If your research group is using the same container for multiple builds, you'll want to use your shared project directory for the `APPTAINER_CACHE` location instead. 

- Paste the URI and pull the container as normal, this process could take a while
``
apptainer pull pytorch.sif docker:[URI obtained from NVIDIA, example: /nvcr.io/nvidia/pytorch:25.01-py3]
``

3. Once the download completes, we can test running a simple script with our container
``
apptainer exec --nv pytorch.sif python gpu_test.py
``
the output should show that we are using a more current version of python than the 3.9 available on the cluster
2nd param: pytorch can detect A100 gpu in our session
``

``
you may find that
NGC does not offer containers that suit all of your computing needs, in these cases you can bootstrap specific containers or the base CUDA NGC container and build your own. you can find more info about building your own containers in CCR's documentation

4. Once the download completes, we can test running nvidia-smi with our container:
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