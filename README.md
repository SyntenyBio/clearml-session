<div align="center">

<a href="https://app.community.clear.ml"><img src="https://github.com/allegroai/clearml/blob/master/docs/clearml-logo.svg?raw=true" width="250px"></a>

## **`clearml-session` </br> CLI for launching JupyterLab / VSCode on a remote machine**


[![GitHub license](https://img.shields.io/github/license/allegroai/clearml-session.svg)](https://img.shields.io/github/license/allegroai/clearml-session.svg)
[![PyPI pyversions](https://img.shields.io/pypi/pyversions/clearml-session.svg)](https://img.shields.io/pypi/pyversions/clearml-session.svg)
[![PyPI version shields.io](https://img.shields.io/pypi/v/clearml-session.svg)](https://img.shields.io/pypi/v/clearml-session.svg)
[![PyPI status](https://img.shields.io/pypi/status/clearml-session.svg)](https://pypi.python.org/pypi/clearml-session/)
[![Slack Channel](https://img.shields.io/badge/slack-%23clearml--community-blueviolet?logo=slack)](https://join.slack.com/t/allegroai-trains/shared_invite/zt-c0t13pty-aVUZZW1TSSSg2vyIGVPBhg)


</div>

**`clearml-session`** is a utility for launching detachable remote interactive sessions (MacOS, Windows, Linux)

### tl;dr 
CLI to launch remote sessions for JupyterLab / VSCode-server / SSH, inside any docker image!

### What does it do?
Starting a clearml (ob)session from your local machine triggers the following:
- ClearML allocates a remote instance (GPU) from your dedicated pool
- On the allocated instance it will spin **jupyter-lab** + **vscode server** + **SSH** access for
interactive usage (i.e., development)
- Clearml will start monitoring machine performance, allowing DevOps to detect stale instances and spin them down

### Use-cases for remote interactive sessions:
1. Development requires resources not available on the current developer's machines
2. Team resource sharing (e.g. how to dynamically assign GPUs to developers)
3. Spin a copy of a previously executed experiment for remote debugging purposes (:open_mouth:!)
4. Scale-out development to multiple clouds, assign development machines on AWS/GCP/Azure in a seamless way

## Prerequisites:
* **An SSH client installed on your machine** - To verify open your terminal and execute `ssh`, if you did not receive an error, we are good to go.
* At least one `clearml-agent` running on a remote host. See installation [details](https://github.com/allegroai/clearml-agent).

Supported OS: MacOS, Windows, Linux


## Secure & Stable
**clearml-session** creates a single, secure, and encrypted connection to the remote machine over SSH.
SSH credentials are automatically generated by the CLI and contain fully random 32 bytes password.

All http connections are tunneled over the SSH connection,
allowing users to add additional services on the remote machine (!)

Furthermore, all tunneled connections have a special stable network layer allowing you to refresh the underlying SSH
connection without breaking any network sockets!  

This means that if the network connection is unstable, you can refresh
the base SSH network tunnel, without breaking JupyterLab/VSCode-server or your own SSH connection
(e.h. debugging over SSH with PyCharm)  

---

## How to use: Interactive Session


1. run `clearml-session`
2. select the requested queue (resource)
3. wait until a machine is up and ready
4. click on the link to the remote JupyterLab/VSCode OR connect with the provided SSH details

**Notice! You can also**: Select a **docker image** to execute in, install required **python packages**, run **bash script**,
pass **git credentials**, etc.
See below for full CLI options.

## Frequently Asked Questions:

#### How Does Clearml enable this?

The `clearml-session` creates a new interactive `Task` in the system (default project: DevOps).

This `Task` is responsible for setting the SSH and JupyterLab/VSCode on the host machine.

The local `clearml-session` awaits for the interactive Task to finish with the initial setup, then
it connects via SSH to the host machine (see "safe and stable" above), and tunnels
both SSH and JupyterLab over the SSH connection.

The end results is a local link which you can use to access the JupyterLab/VSCode on the remote machine, over a **secure and encrypted** connection!

#### How can this be used to scale up/out development resources?

**Clearml** has a cloud autoscaler, so you can easily and automatically spin machines for development!

There is also a default docker image to use when initiating a task.

This means that using **clearml-session**s
with the autoscaler enabled, allows for turn-key secure development environment inside a docker of your choosing.

Learn more about it [here]()

#### Does this fit Work From Home situations?
**YES**. Install `clearml-agent` on target machines inside the organization, connect over your company VPN 
and use `clearml-session` to gain access to a dedicated on-prem machine with the docker of your choosing
(with out-of-the-box support for any internal docker artifactory).

Learn more about how to utilize your office workstations and on-prem machines [here]()

## Tutorials

### Getting started

Requirements `clearml` python package installed and configured (see detailed [instructions]())
``` bash
pip install clearml-session
clearml-session --docker nvcr.io/nvidia/pytorch:20.11-py3 --git-credentilas
```

Wait for the machine to spin up:
Expected CLI output would look something like:
``` console
Creating new session
New session created [id=3d38e738c5ff458a9ec465e77e19da23]
Waiting for remote machine allocation [id=3d38e738c5ff458a9ec465e77e19da23]
.Status [queued]
....Status [in_progress]
Remote machine allocated
Setting remote environment [Task id=3d38e738c5ff458a9ec465e77e19da23]
Setup process details: https://app.community.clear.ml/projects/64ae77968db24b27abf86a501667c330/experiments/3d38e738c5ff458a9ec465e77e19da23/output/log
Waiting for environment setup to complete [usually about 20-30 seconds]
..............
Remote machine is ready
Setting up connection to remote session
Starting SSH tunnel
Warning: Permanently added '[192.168.0.17]:10022' (ECDSA) to the list of known hosts.
root@192.168.0.17's password: f7bae03235ff2a62b6bfbc6ab9479f9e28640a068b1208b63f60cb097b3a1784


Interactive session is running:
SSH: ssh root@localhost -p 8022 [password: f7bae03235ff2a62b6bfbc6ab9479f9e28640a068b1208b63f60cb097b3a1784]
Jupyter Lab URL: http://localhost:8878/?token=df52806d36ad30738117937507b213ac14ed638b8c336a7e
VSCode server available at http://localhost:8898/

Connection is up and running
Enter "r" (or "reconnect") to reconnect the session (for example after suspend)
Ctrl-C (or "quit") to abort (remote session remains active)
or "Shutdown" to shutdown remote interactive session
```

Click on the JupyterLab link (http://localhost:8878/?token=xyz)
Open your terminal, clone your code & start working :)

### Leaving a session and reconnecting from the same machine

On the `clearml-session` CLI terminal, enter 'quit' or press Ctrl-C
It will close the CLI but leaves the remote session running

When you want to reconnect to it, execute:
``` bash
clearml-session
```

Then press "Y" (or enter) to reconnect to the already running session
``` console
clearml-session - launch interactive session
Checking previous session
Connect to active session id=3d38e738c5ff458a9ec465e77e19da23 [Y]/n?
```

### Shutting down a remote session

On the `clearml-session` CLI terminal, enter 'shutdown' (case-insensitive)
It will shut down the remote session, free the resource and close the CLI

``` console
Enter "r" (or "reconnect") to reconnect the session (for example after suspend)
Ctrl-C (or "quit") to abort (remote session rema
Yes of course, current SSO supports Google/GitHub/BitBucket/... + SAML/LDAP (Usually with user permissions fully integrated to the LDAP)
ins active)
or "Shutdown" to shutdown remote interactive session

shutdown

Shutting down interactive session
Interactive session ended
Leaving interactive session
```

### Connecting to a running interactive session from a different machine

Continue working on an interactive session from **any** machine.
In the `clearml` web UI, go to DevOps project, and find your interactive session.
Click on the ID button next to the Task name, and copy the unique ID.

``` bash
clearml-session --attach <session_id_here>
```

Click on the JupyterLab/VSCode link, or connect directly to the SSH session

### Debug a previously executed experiment

If you have a previously executed experiment in the system,
you can create an exact copy of the experiment and debug it on the remote interactive session.
`clearml-session` will replicate the exact remote environment, add JupyterLab/VSCode/SSH and allow you interactively
execute and debug the experiment, on the allocated remote machine.  

In the `clearml` web UI, find the experiment (Task) you wish to debug.
Click on the ID button next to the Task name, and copy the unique ID.

``` bash
clearml-session --debugging <experiment_id_here>
```

Click on the JupyterLab/VSCode link, or connect directly to the SSH session

## CLI options

``` bash
clearml-session --help
```

``` console
clearml-session - CLI for launching JupyterLab / VSCode on a remote machine
usage: clearml-session [-h] [--version] [--attach [ATTACH]]
                       [--debugging DEBUGGING] [--queue QUEUE]
                       [--docker DOCKER] [--public-ip [true/false]]
                       [--vscode-server [true/false]]
                       [--jupyter-lab [true/false]]
                       [--git-credentials [true/false]]
                       [--user-folder USER_FOLDER]
                       [--packages [PACKAGES [PACKAGES ...]]]
                       [--requirements REQUIREMENTS]
                       [--init-script [INIT_SCRIPT]]
                       [--config-file CONFIG_FILE]
                       [--remote-gateway [REMOTE_GATEWAY]]
                       [--base-task-id BASE_TASK_ID] [--project PROJECT]
                       [--disable-keepalive]
                       [--queue-excluded-tag [QUEUE_EXCLUDED_TAG [QUEUE_EXCLUDED_TAG ...]]]
                       [--queue-include-tag [QUEUE_INCLUDE_TAG [QUEUE_INCLUDE_TAG ...]]]
                       [--skip-docker-network] [--password PASSWORD]
                       [--username USERNAME]

clearml-session - CLI for launching JupyterLab / VSCode on a remote machine

optional arguments:
  -h, --help            show this help message and exit
  --version             Display the clearml-session utility version
  --attach [ATTACH]     Attach to running interactive session (default:
                        previous session)
  --debugging DEBUGGING
                        Pass existing Task id (experiment), create a copy of
                        the experiment on a remote machine, and launch
                        jupyter/ssh for interactive access. Example
                        --debugging <task_id>
  --queue QUEUE         Select the queue to launch the interactive session on
                        (default: previously used queue)
  --docker DOCKER       Select the docker image to use in the interactive
                        session on (default: previously used docker image or
                        `nvidia/cuda:10.1-runtime-ubuntu18.04`)
  --public-ip [true/false]
                        If True register the public IP of the remote machine.
                        Set if running on the cloud. Default: false (use for
                        local / on-premises)
  --vscode-server [true/false]
                        Install vscode server (code-server) on interactive
                        session (default: true)
  --jupyter-lab [true/false]
                        Install Jupyter-Lab on interactive session (default:
                        true)
  --git-credentials [true/false]
                        If true, local .git-credentials file is sent to the
                        interactive session. (default: false)
  --user-folder USER_FOLDER
                        Advanced: Set the remote base folder (default: ~/)
  --packages [PACKAGES [PACKAGES ...]]
                        Additional packages to add, supports version numbers
                        (default: previously added packages). examples:
                        --packages torch==1.7 tqdm
  --requirements REQUIREMENTS
                        Specify requirements.txt file to install when setting
                        the interactive session. Requirements file is read and
                        stored in `packages` section as default for the next
                        sessions. Can be overridden by calling `--packages`
  --init-script [INIT_SCRIPT]
                        Specify BASH init script file to be executed when
                        setting the interactive session. Script content is
                        read and stored as default script for the next
                        sessions. To clear the init-script do not pass a file
  --config-file CONFIG_FILE
                        Advanced: Change the configuration file used to store
                        the previous state (default: ~/.clearml_session.json)
  --remote-gateway [REMOTE_GATEWAY]
                        Advanced: Specify gateway ip/address to be passed to
                        interactive session (for use with k8s ingestion / ELB)
  --base-task-id BASE_TASK_ID
                        Advanced: Set the base task ID for the interactive
                        session. (default: previously used Task). Use `none`
                        for the default interactive session
  --project PROJECT     Advanced: Set the project name for the interactive
                        session Task
  --disable-keepalive   Advanced: If set, disable the transparent proxy always
                        keeping the sockets alive. Default: false, use
                        transparent socket mitigating connection drops.
  --queue-excluded-tag [QUEUE_EXCLUDED_TAG [QUEUE_EXCLUDED_TAG ...]]
                        Advanced: Excluded queues with this specific tag from
                        the selection
  --queue-include-tag [QUEUE_INCLUDE_TAG [QUEUE_INCLUDE_TAG ...]]
                        Advanced: Only include queues with this specific tag
                        from the selection
  --skip-docker-network
                        Advanced: If set, `--network host` is **not** passed
                        to docker (assumes k8s network ingestion) (default:
                        false)
  --password PASSWORD   Advanced: Select ssh password for the interactive
                        session (default: `randomly-generated` or previously
                        used one)
  --username USERNAME   Advanced: Select ssh username for the interactive
                        session (default: `root` or previously used one)

Notice! all arguments are stored as new defaults for the next session
```
