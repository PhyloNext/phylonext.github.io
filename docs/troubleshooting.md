---
title: Troubleshooting
description: >-
    Common issues and solutions users might run into while using PhyloNext.
---

# Troubleshooting

Common issues and solutions users might run into while using PhyloNext.

!!! bug "Don't see your error/bug? Post an issue on GitHub"
    If you've encountered an error or bug not seen here, please post an issue at [PhyloNext GitHub Issues](https://github.com/vmikk/PhyloNext/issues). This will help greatly to track down the error and fix it!

## Pipeline errors

### Resume the workflow

!!! failure "Pipeline was interrupted"
    Any accidental failure - e.g., you closed the terminal, rebooted your computer, or the task was killed by a scheduler on HPC.

If the pipeline is interrupted, it can be resumed without having to start from scratch.  
Add the `-resume` flag to your command, and Nextflow will start the workflow from the last successfully executed process, 
retrieving all previous results from the cache.

### Specifying the full path for input data

When inputting data, ensuring the correct file path is essential for successful processing. 
Incorrect or incomplete paths will lead to errors.  

!!! failure "File path"
    `Not a valid path value`

To avoid this issue, it is recommended to use the absolute path to your input data. 
An absolute path specifies the exact location of a file in the file system, starting from the root directory.  

For instance, if your file is located in the current working directory, 
you can construct its absolute path by prefixing the file name with `$(pwd)/`. 
This command outputs the full path of the current directory, which, 
when combined with your file or directory name, provides the complete path required.  
Example: `$(pwd)/your_file_name`.  


### The number of available CPUs

!!! failure "Number of CPUs required"
    `Process requirement exceeds available CPUs -- req: 10; avail: 8`

Most likely, the number of CPUs configured for a process is larger than available on your system. 
To fix it, modify a config file and reduce the number of CPUs for workflow processes to fit your hardware. E.g., change
``` java
    withName:occ_filter{
        cpus = 10
    }
```
to 
``` java
    withName:occ_filter{
        cpus = 8
    }
```

See the `Configuration` section in the [usage documentation](https://phylonext.github.io/usage/#configuration-file).

### Pipeline revision version

!!! failure "Pipeline revision version"
    ```
    Project \`vmikk/PhyloNext\` is currently stickied on revision: main -- you need to explicitly specify a revision with the option \`-r\` in order to use it
    ```

Solution - specify a version number of the pipeline.  
To run the latest version, add `-r main` to your command.  
Alternatively, you may specify the exact version (or tag) you wish, e.g. `-r v.0.0.2`.  

### Lack of node ages in the phylogenetic tree for target taxon

Currently, OpenTree may not contain age estimates for all nodes within the synthetic tree, 
which can impede the automatic retrieval of a phylogenetic tree for your taxon of interest.  

To address this limitation, we outline several strategies below.  

!!! failure "Missing age estimate for subtree root"
    `ERROR: no age estimate for subtree root could be found in datastore. Please  re-run query with a max root age using argument 'max_age'.`

When the database lacks age data for a clade, 
you can manually set a maximum age for the clade using the `--maxage` parameter. 
It's advisable to use a credible value, such as one derived from scholarly articles 
(e.g., through a Google Scholar search for published age estimates).  


!!! failure "Inadequate data for date estimation"
    `ERROR: only 2 or fewer age calibrations found for this tree. We have insufficient data to estimate dates.`

In situations with insufficient data for accurate date estimation, 
acquiring a custom tree may be necessary. 
To derive node age estimates, you can use `Chronosynth` 
and apply `Bladj` to smooth out the node ages.
For instructions, visit [McTavishLab's GitHub repository](https://github.com/McTavishLab/GBIF-Biodiverse-OpenTree/tree/main/dates).  


## Installation problems

### Nextflow not found

!!! failure "Command not found"
    `command not found: nextflow`

If you installed Nextflow in the `~/bin` directory, most likely, it is not added to your PATH environment variable. 
To fix it, run the following command:

``` bash
export PATH="$HOME/bin:$PATH"
```
and try to execute Nextflow again.  

If it works, you may add this command to the `~/.bashrc` file to make these changes permanent. 
To do it from the command line, run:  

``` bash
echo "export PATH=$HOME/bin:$PATH" >> ~/.bashrc
```

!!! example "Notes on shells other than bash"
    If you are using a shell other than `bash`, please modify configuration file specific for your shell.  
    E.g., for `zsh` it should be `~/.zshrc`.  
    To know which shell are you using, run `echo $SHELL`

### GitHub API rate limits

!!! failure "GitHub API rate limits"
    ```
    Checking vmikk/PhyloNext ...  
    WARN: Cannot read project manifest -- Cause: API rate limit exceeded -- Provide your GitHub user name and password to get a higher rate limit  
    API rate limit exceeded -- Provide your GitHub user name and password to get a higher rate limit
    ```

Without authentication, GitHub allows only a limited number of connections per hour (60 unauthenticated requests/hr). 
Most likely, you've hit this problem because users from your organization (e.g., university) send too many requests to GitHub.  

To overcome this limitation, you may generate a personal access token at [https://github.com/settings/tokens](https://github.com/settings/tokens) (the minimum required scopes are `repo`, `read:org`, and `workflow`).  
To authenticate, you may use the GitHub client `gh` ([https://cli.github.com/](https://cli.github.com/)).  
Run `gh auth login --with-token <YOURTOKEN>` to login. This way you will have a higher API rate limit.  

Alternatively, you may wait for a while until the recovery of API limits. 
Or, if you are using PhyloNext locally, you may change your IP address 
(for example, connect your laptop to another wi-fi hotspot, e.g., to a smartphone hotspot) 
and try to pull the pipeline once again. 
You need to download the pipeline from GitHub only once. 
Then, Nextflow will cache the pipeline code locally in the `~/.nextflow/assets/` directory 
and will re-use it later without the need to connect to GitHub.  

As another option, you may just manually clone the git repository into Nextlow assets directory:  
``` bash
mkdir -p ~/.nextflow/assets/vmikk/
cd ~/.nextflow/assets/vmikk/
git clone https://github.com/vmikk/PhyloNext
```

!!! example "Rate limit information"
    To find the current rate limit information (for unauthenticated requests), run  
    `curl -I https://api.github.com/users/octocat`

!!! warning "Token security"
    It is important to keep your token secure and not share it with others.  
    Also make sure to revoke the token if it is compromised or no longer needed.

### Docker permissions

!!! failure "Docker - permission denied"
    `docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/create?name=nxf-Cz6yqNjGI9AhRpsqkU4mSAHC": dial unix /var/run/docker.sock: connect: permission denied.`

Most likely, you forgot to add your user to the `docker` group and you need a super-user privileges to execute the container.  
To fix this error, run:  

``` bash
# Create the docker group.
sudo groupadd docker

#  Add your user to the docker group.
sudo usermod -aG docker ${USER}

# Log out and log back in so that your group membership is re-evaluated.
# Alternatively, to skip logging out, type the following command:  su -s ${USER}

# Verify that you can run docker commands without sudo
docker run hello-world
```

### Docker and WSL2

!!! failure "Docker and WSL2"
    `The command 'docker' could not be found in this WSL 2 distro. We recommend to activate the WSL integration in Docker Desktop settings.`

Try to enable `Use the WSL2 based engine` in the `Docker Desktop` settings.  

<a class="zoom" href="/assets/DokerDesktop_WSL2.webp">
![WSL2 Docker Engine](/assets/DokerDesktop_WSL2.webp)
</a>


### PhyloNext containers

If for some reason Nextflow is not able to download containers for PhyloNext, 
it's possible to pull them manually using:

``` bash
docker pull vmikk/rarrow:1.0.0
docker pull vmikk/biodiverse:1.0.0
docker pull vmikk/opentree:0.0.2
```
