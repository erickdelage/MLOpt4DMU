# How to access Digital Research Alliance of Canada clusters (Compute Canada/Calcul Quebec)

**All needed information is available at https://docs.alliancecan.ca/wiki/Technical_documentation **


## Creating a CCDB account

Start by creating an account on: https://www.computecanada.ca/research-portal/account-management/apply-for-an-account/.  While creating your account you will need to provide the CCRI of your sponsor/faculty member (Erick's CCRI is: iee-073-01). Your sponsor/Erick will then receive an email to approve your membership. Once your account is approved (which should be fast), you can login at the ccdb website at https://ccdb.alliancecan.ca/. You can use `My Account -> Request access to other clusters` to get access to Niagara and Mist, the two clusters you need to apply to access. You have automatically access Cedar, Narval, Beluga, and Graham.

## ssh

You can use ssh -keys to log in compute canada, refer to https://docs.alliancecan.ca/wiki/SSH_Keys but in short you need to go to https://ccdb.computecanada.ca/ssh_authorized_keys to add the keys on their website. It is **highly** recommended that you set up an ssh key. You can generate one with the `ssh-keygen` command and copy it to the cluster with `ssh-copy-id` (i.e. `ssh-copy-id user@mist.scinet.utoronto.ca`).


You can also setup shortcuts via `.ssh/config`, for instance
```
Host beluga cedar graham narval niagara
    Hostname %h.computecanada.ca
Host mist
    Hostname mist.scinet.utoronto.ca
Host !beluga  bc????? bg????? bl?????
	HostName %h.int.ets1.calculquebec.ca
    ProxyJump beluga
Host !cedar   cdr? cdr?? cdr??? cdr????
    ProxyJump cedar
Host !graham  gra??? gra????
    ProxyJump graham
Host !narval  nc????? ng?????
    ProxyJump narval
Host !niagara nia????
    ProxyJump niagara

Match Host *.computecanada.ca 
	User youraccountname
    IdentityFile ~/.ssh/id_rsa
Match Host *.calculquebec.ca
    User youraccountname
    IdentityFile ~/.ssh/id_rsa
Host nc*
	HostName %h.narval.calcul.quebec
	User youraccountname
	ProxyJump narval
	IdentityFile ~/.ssh/id_rsa
	Compression yes

Host bc*
	HostName %h.int.ets1.calculquebec.ca
	User youraccountname
	ProxyJump beluga
	IdentityFile ~/.ssh/id_rsa
	Compression yes
```

Will allow you to use commands like `ssh beluga` to connect to beluga. refer to https://docs.alliancecan.ca/wiki/SSH_configuration_file 

## Slurm 101

The Alliance uses [slurm](https://slurm.schedmd.com/documentation.html) to manage jobs. It is **highly** recommended that you read https://docs.alliancecan.ca/wiki/Running_jobs before submitting jobs. Understanding Bash is a must.

Every job in Slurm has a series of requirements like number of nodes (computers), cpus per nodes (cores), memory per node or core, run time, account, and generic resources for jobs that need GPU. It is important to have a good estimate of the resources needed it will affect the time it takes you job to start running. For instance `salloc --time=2:00:00 --gres=gpu:1 --account=rrg-adulyasa --cpus-per-task=8 --mem=32G` will give an interactive session for 2 hours, one gpu, on the `rrg-adulyasa` allocation, with 8 cores, and 32G of memory. It's only important to an account if you have access to an allocation.


Some useful command
- `sbatch script.sh` will the run the scripts, it's possible to not need to specify the commands each time by storing them as comments that start with `#SBATCH` for instance, assuming the following file is named `a.sh`,
```bash
#!/bin/bash
#SBATCH --mem=32G
#SBATCH --cpus-per-task=8
#SBATCH --output=slurm/output.%j.out
#SBATCH --gres=gpu:1
#SBATCH --time=2-12:00:00
#SBATCH --account=rrg-adulyasa

ulimit -n `ulimit -H -n`
. "/home/sobhanmp/miniconda3/etc/profile.d/conda.sh"
conda activate torch3.9
wandb offline
python ab.py
```
`sbatch a.sh` is equivalent to `sbatch --mem=32G a.sh` but with the second line remove.
`sbatch 
- `scancel 88000` can be used to cancel jobs.
- `squeue` List all queued jobs. You can specify the user with the `-u` argument or `--me` to show your jobs. **DONT RUN `squeue` with no argument frequently**.
- `squeue --me -t RUNNING` Lists all running jobs for a user.
- `squeue --me -t PENDING` Lists all pending jobs for a user.
- `scontrol show jobid -dd your_jobid` Lists detailed information for a job (useful for troubleshooting).
- `srun` can be used to run jobs that block or if used inside a script, it will define a step that can be used to debug the program.
- `salloc` is for allocating interactive sessions.

## Small note on GPU-years and CPU-years

How much resources you use, therefore your priority is determinited by how many of each you use. See https://docs.alliancecan.ca/wiki/Allocations_and_compute_scheduling#What_is_a_GPU_equivalent_and_how_is_it_used_by_the_scheduler? for an in-depth explanation but basically, using more cores per gpu, or ram per core than listed in that page will make you use priority as if you had used more cores.

## Python on the clusters

According to Compute Canada, it is preferable to use virtualenv to create your virtual environment instead of conda. Using conda on compute Canada is possible but presents difficulties. Refer to `https://docs.alliancecan.ca/wiki/Python` on the recommended version of running python. If for some reason, like specific dependency, you are not able to satisfy you needs through CC's wheels, you need to install conda, add the following to your `.bashrc` file:
```
unset PYTHONPATH
unset PIP_CONFIG_FILE
```
then install conda. If conda segfaults, use `setrpaths.sh --path <path to conda's bin folder>` https://docs.alliancecan.ca/wiki/Installing_software_in_your_home_directory#Installing_binary_packages to fix conda.


## The file system and moving files

Highly recommended wiki article: https://docs.alliancecan.ca/wiki/Storage_and_file_management

There are 4 file systems you have access to that are available as variables (i.e. `$SCRATCH` is the path to your scratch)
- `HOME` is the best place to store code, however submitting jobs from may is not possible on some clusters.
- `PROJECT` is the best place to archive and share code with people of the same group
- `SCRATCH` is the faster than both `HOME` and `PROJECT` but files not accessed for more than 90 days may be purged
- `SLURM_TMPDIR` is a local folder (unlike the other 3 that are on a shared file system), it is the fastest.



Wiki article: https://docs.alliancecan.ca/wiki/Transferring_data

The recommended way of moving code is via `git` and github. For small files (few gigabytes) you can use `scp` and `rsync` to sync directories. If you use rsync **read** the wiki articles for the recommended options. You can use the `sftp` (ftp over ssh) protocol in file zila and simlar software to transfer data.

# Useful Commands/Tips

- Before you start using any cluster, make sure to use the right cluster for your needs:
    - **Niagara** accepts jobs of up to 24 hours run-time
    - **Béluga** up to 7 days.
    - **Cedar** and **Graham** up to 28 days.
    
  According to our experience, we would recommend using Cedar or Graham over Niagara, as these clusters have an easier and less restrictive environment for development.
    

    
- Example of creating a script launch_script.sh and running a python file through it using **sbatch**. This script is set up to run on Niagara but we encourage you to be using Beluga with the account rrg-adulyasa.
```
#!/bin/bash
#SBATCH --job-name=RLTest
#SBATCH --mail-user=your_email
#SBATCH --account=def-edelage #use either rrg-adulyasa (Beluga), rrg-edelage (Cedar), or def-edelage (others)
#SBATCH --ntasks=1
#SBATCH --mem-per-cpu=2G
#SBATCH --cpus-per-task=1
#SBATCH --output=/scratch/e/edelage/fathanab/output.txt 
#SBATCH --time=1:00:00
#SBATCH --gres=gpu:TitanX:1
python python_test.py
```

Once you run `sbatch launch_script.sh`, it will run your `python_test.py` file and its output will be logged in `output.txt`. 

- In the above file:
    - RLTest will be the name of my task (arbitrary value).
    - Provide your email to receive notification once your script finishes running.
    - You provide how many tasks and how much memory per task and how many CPUs to use per task.
    - `--time` is for the time allowed for your script to run, otherwise it will be 15 minutes by default and your script will be stopped.
    - `--gres=gpu:TitanX:1` requests one `TitanX` gpu. `--gres=gpu` works fine if you can run your code on all gpus.
    
After launching the job with `sbatch`, you can check the status of the job:
![](Picture6.png)
You could also check the intermediate results and output of your commands logged in output.txt
![](Picture7.png)
![](Picture8.png)

- Example of other commands possible depending on your use case:
    - `srun jupyter nbconvert --ExecutePreprocessor.timeout=-1 --to notebook --inplace --execute test_srun.ipynb`
    - `srun python autoencoder.py SCRATCH/model.hdf5 $SCRATCH/autoencoder-labeller/$RUN_NAME -n-epoch 20 -batch-size 2 -image-every-n 50 -learning-rate-labeller 1e-2 -learning-rate-encoder 1e-2 -batch-limit 50`
    

  
- Using Tensorflow:
```bash         
module load python/3.6
virtualenv $HOME/jupyter_py3
source $HOME/jupyter_py3/bin/activate
pip install --no-index tensorflow_gpu # you could specify your specific version here
pip install jupyter notebook #Optional for useful use of jupyter remotely (Debug, etc.)
echo -e '#!/bin/bash\nunset XDG_RUNTIME_DIR\njupyter notebook --ip $(hostname -f) --no-browser' > $VIRTUAL_ENV/bin/notebook.sh
chmod u+x $VIRTUAL_ENV/bin/notebook.sh
```
        
  On Niagara cluster, if you still need to use conda or you experience problems using Tensorflow or using a specific version that is unavailable with pip, you could load Conda and install tensorflow:
```  
module load anaconda3
conda create -n tfenv python=3.6
conda activate tfenv
conda install -c https://public.dhe.ibm.com/ibmdl/export/pub/software/server/ibm-ai/conda/ tensorflow-gpu==1.14.0
echo ". /scinet/niagara/software/2018a/opt/base/anaconda3/2018.12/etc/profile.d/conda.sh" >> ~/.bashrc # You may need to adapt to your own path
locate ~/.bashrc # Optional: If ever you need to locate your bashrc file.
```     

- Start a jupyter session:
        `salloc --time=3:0:0 --ntasks=1 --cpus-per-task=2 --mem-per-cpu=1024M --account=def-edelage srun $VIRTUAL_ENV/bin/notebook.sh`
- Start a ssh tunnel to connect to jupyter notebook: 
    `sshuttle --dns -Nr username@cedar.computecanada.ca`
    Now you can go to your browser and use the http address of your session.
  
- Example of slurm files to run a jupyter notebook and a python file:

  On Cedar (no use of conda):
```
#!/bin/bash
#SBATCH --gres=gpu:v100l:1        # request GPU, here V100
#SBATCH --cpus-per-task=8   # maximum CPU cores per GPU request: 6 on Cedar, 16 on Graham.
#SBATCH --mem=0               # Request the full memory of the node
#SBATCH --account=rrg-edelage
#SBATCH --job-name=Name_of_your_task
#SBATCH --mail-user=Your_email
#SBATCH --mail-type=ALL
#SBATCH --ntasks=1
#SBATCH --time=27-13:00      # time (DD-HH:MM)
#SBATCH --output=$SCRATCH/outputs/%N-%j.out  # %N for node name, %j for jobID

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
source $HOME/jupyter_py3/bin/activate
cd $SCRATCH/Your_project_folder
srun jupyter nbconvert --ExecutePreprocessor.timeout=-1 --to notebook --inplace --execute your_jupyter_file.ipynb
```

On Niagara (use conda tfenv on top of your virtual environment):
```
#!/bin/bash
#SBATCH --job-name=Your_test
#SBATCH --mail-user=Your_email
#SBATCH --ntasks=1
#SBATCH --mem-per-cpu=2G
#SBATCH --cpus-per-task=1
#SBATCH --output=/scratch/e/edelage/username/output_%N-%j.txt
# #SBATCH --gres=gpu:TitanX:1

module load anaconda3
source /home/e/edelage/username/.bashrc
source /home/e/edelage/username/tensorflow/bin/activate
conda activate tfenv
cd $SCRATCH #Choose your working directory
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/e/edelage/username/.conda/envs/tfenv/lib
srun python $SCRATCH/tensorflow-test.py
 ```   
 - On Niagara, it is possible to have easy access to your files and modify/execute them through jupyter via this link: https://jupyter.scinet.utoronto.ca
 
   You will first have to create a symbolic link to your $SCRATCH folder by executing the following command:
    - `ln -sT $SCRATCH $HOME/scratch`
        
 - Compress/Extract compressed directory:
        - `tar -czvf file.tar.gz directory # To compress directory into file.tar.gz`
        - `tar xvzf file.tar.gz # To extract file.tar.gz`
        - `tar -xvf file.tar.bz2 # To extract file.tar.bz2`
        
 - Useful links:
    - [Getting started with Compute Canada](https://docs.computecanada.ca/wiki/Getting_started)
    - [Getting started with Slurm](https://docs.computecanada.ca/wiki/Using_GPUs_with_Slurm)
    - [Using modules](https://docs.scinet.utoronto.ca/index.php/Using_modules)
    - [Using Jupyter](https://docs.computecanada.ca/wiki/Jupyter)
    - [Using Cedar](https://docs.computecanada.ca/wiki/Cedar)
    - [Running jobs on Slurm](https://docs.computecanada.ca/wiki/Running_jobs)
    - [Details about srun](https://slurm.schedmd.com/srun.html)
    - [Getting started with Niagara](https://docs.computecanada.ca/wiki/Niagara_Quickstart)
    - [Using Jupyter Hub](https://docs.scinet.utoronto.ca/index.php/Jupyter_Hub)
    - [Using Tensorflow on CC](https://docs.computecanada.ca/wiki/TensorFlow)
    - [Getting started with Mist](https://docs.scinet.utoronto.ca/index.php/Mist#TensorFlow_and_Keras)
        


# Setting up a virtual environment for python and running a job on GPU

For users that want to use python for their projects, it is recommended to first install a virtual environment, install their required packages in this environment, and then run their jobs. Here is the instruction for creating a virtual environment using python 3.6:
    - Move to the main directory of your user account in computecanda
    - Load python 3.6 on the server by using this command: module load python/3.6
    - Create the virtual environment by using this command: `virtualenv --no-download ~/userenv36`  (Note that you can choose another name instead of userenv36 for your environment)
    - Activate the virtual environment by using this command: `source ~/userenv36/bin/activate`
    - Upgrade pip by using this command: `pip install --no-index --upgrade pip`
    - Whenever needed you can deactivate the virtual environment using this command: `deactivate`

After creating the virtual environment, you can install different packages you need without affecting the libraries that are installed on the main server. In order to install different libraries you can simply activate the virtual environment that you just created and then use pip to install them. Here is an example to install pandas: pip install pandas

Note that all your files should be put in the directory "scratch"

It is recommended to use Github for transferring files to computecanda. The benefit of doing so is that every time you make a change in your project you can simply replace the new files with old ones in computecanada by running a script that downloads your files from github. Here is the instruction for downloading a github project on your local computer on windows, making changes in the project, and then updating the project in computecanada:
    - Download and install the git bash app from this link: https://git-scm.com/downloads
    - Create a directory in your local computer for the specific project that you want to download.
    - Go to the github page of the project and copy the url. For example we can download the current project by using this url: https://github.com/erickdelage/CRCDMU_public
    - Change the current directory to the newly created one and choose "Git bash here" from the right click menu so the command window opens up.
    - Type "git clone https://github.com/erickdelage/CRCDMU_public.git"
    - This will download the project into the created directory
    - In order to create the same project in compute canada, move to the "scratch" folder, and run the same command there. This will download the same project in computecanada.
    - Now you can make changes into different files of the project including .py files.
    - Now you need to commit the changes you have already made into the github. To do so, type the following commands in order:
    -   `git add .`
    -   `git commit -m "update python"`
    -   `git push origin master`
    - Now we can update the project in computecanada.
    - Move to the directory where this prject is downloaded in computecanada and run the following command: git pull origin master
    - This command will update the project in computecanada according to the changes you made in your local computer.


Finally here is an example of a .sh file that will run a job on GPU. We assume in the project that you have downloaded into your computecanada account in "scratch" there is a runModel.py file that runs your project:
```
#!/bin/bash
#SBATCH --job-name=RLTest
#SBATCH --mail-user=user@hec.ca
#SBATCH --ntasks=1
#SBATCH --mem-per-cpu=8G
#SBATCH --cpus-per-task=1
#SBATCH --output=output.txt
#SBATCH --time=1:00:00
#SBATCH --gres=gpu:p100:1
module load python/3.6
source ~/userenv36/bin/activate
srun python runModel.py
```



## Access to Special Software

The use of commercial software such as Matlab, Cplex, or Gurobi is possible.

- Matlab: You could use Octave, free and open source, to execute your Matlab code, described in this [link](https://docs.scinet.utoronto.ca/index.php/Octave). Or you can use Matlab directly by compiling your code and copying the executables into Niagara and running them there, or use your License information to execute the Matlab code directly. Everything is described in this [link](https://docs.scinet.utoronto.ca/index.php/MATLAB). This [link](https://docs.scinet.utoronto.ca/index.php/SSH_Tunneling) could also be useful for you to tunnel to your license server.
- Gurobi: Described [here](https://docs.scinet.utoronto.ca/index.php/Gurobi).
- Cplex: Instructions [here](https://docs.computecanada.ca/wiki/CPLEX/en).

- In case you need technical assistance or you don't find your available version, you could contact the Niagara/Compute Canada support teams to have it installed for you: 
    - support@scinet.utoronto.ca
    - niagara@computecanada.ca


