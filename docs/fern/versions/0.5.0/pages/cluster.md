---
title: "Set Up Clusters"
---
This guide explains how to run NeMo RL with Ray on Slurm or Kubernetes.

## Use Slurm for Batched and Interactive Jobs

 The following code provides instructions on how to use Slurm to run batched job submissions and run jobs interactively.

### Batched Job Submission

```sh
# Run from the root of NeMo RL repo
NUM_ACTOR_NODES=1  # Total nodes requested (head is colocated on ray-worker-0)

COMMAND="uv run ./examples/run_grpo_math.py" \
CONTAINER=YOUR_CONTAINER \
MOUNTS="$PWD:$PWD" \
sbatch \
    --nodes=${NUM_ACTOR_NODES} \
    --account=YOUR_ACCOUNT \
    --job-name=YOUR_JOBNAME \
    --partition=YOUR_PARTITION \
    --time=1:0:0 \
    --gres=gpu:8 \
    ray.sub
```

> [!TIP]
> Depending on your Slurm cluster configuration, you may or may not need to include the `--gres=gpu:8` option in the `sbatch` command.

> [!NOTE]
> For GB200 systems with 4 GPUs per node, use `--gres=gpu:4` instead of `--gres=gpu:8`.

Upon successful submission, Slurm will print the `SLURM_JOB_ID`:
```text
Submitted batch job 1980204
```
Make a note of the job submission number. Once the job begins, you can track its process in the driver logs which you can `tail`:
```sh
tail -f 1980204-logs/ray-driver.log
```

### Interactive Launching

> [!TIP]
> A key advantage of running interactively on the head node is the ability to execute multiple multi-node jobs without needing to requeue in the Slurm job queue. This means that during debugging sessions, you can avoid submitting a new `sbatch` command each time. Instead, you can debug and re-submit your NeMo RL job directly from the interactive session.

To run interactively, launch the same command as [Batched Job Submission](#batched-job-submission), but omit the `COMMAND` line:
```sh
# Run from the root of NeMo RL repo
NUM_ACTOR_NODES=1  # Total nodes requested (head is colocated on ray-worker-0)

CONTAINER=YOUR_CONTAINER \
MOUNTS="$PWD:$PWD" \
sbatch \
    --nodes=${NUM_ACTOR_NODES} \
    --account=YOUR_ACCOUNT \
    --job-name=YOUR_JOBNAME \
    --partition=YOUR_PARTITION \
    --time=1:0:0 \
    --gres=gpu:8 \
    ray.sub
```

> [!NOTE]
> For GB200 systems with 4 GPUs per node, use `--gres=gpu:4` instead.

Upon successful submission, Slurm will print the `SLURM_JOB_ID`:
```text
Submitted batch job 1980204
```
Once the Ray cluster is up, a script will be created to attach to the Ray head node. Run this script to launch experiments:
```sh
bash 1980204-attach.sh
```
Now that you are on the head node, you can launch the command as follows:
```sh
uv run ./examples/run_grpo_math.py
```

### Slurm Environment Variables

All Slurm environment variables described below can be added to the `sbatch`
invocation of `ray.sub`. For example, `GPUS_PER_NODE=8` can be specified as follows:

```sh
GPUS_PER_NODE=8 \
... \
sbatch ray.sub \
   ...
```
#### Common Environment Configuration
<table>
  <thead>
    <tr>
      <th>Environment Variable</th>
      <th>Explanation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>CONTAINER</code></td>
      <td>(Required) Specifies the container image to be used for the Ray cluster.<br />Use either a docker image from a registry or a squashfs (if using enroot/pyxis).</td>
    </tr>
    <tr>
      <td><code>MOUNTS</code></td>
      <td>(Required) Defines paths to mount into the container. Examples:<br /><code>MOUNTS="$PWD:$PWD"</code> (mount in current working directory (CWD))<br /><code>MOUNTS="$PWD:$PWD,/nfs:/nfs:ro"</code> (mounts the current working directory and <code>/nfs</code>, with <code>/nfs</code> mounted as read-only)</td>
    </tr>
    <tr>
      <td><code>COMMAND</code></td>
      <td>Command to execute after the Ray cluster starts. If empty, the cluster idles and enters interactive mode (see the [Slurm interactive instructions](#interactive-launching)).</td>
    </tr>
    <tr>
      <td><code>HF_HOME</code></td>
      <td>Sets the cache directory for huggingface-hub assets (e.g., models/tokenizers).</td>
    </tr>
    <tr>
      <td><code>WANDB_API_KEY</code></td>
      <td>Setting this allows you to use the wandb logger without having to run <code>wandb login</code>.</td>
    </tr>
    <tr>
      <td><code>HF_TOKEN</code></td>
      <td>Setting the token used by huggingface-hub. Avoids having to run the <code>huggingface-cli login</code></td>
    </tr>
    <tr>
      <td><code>HF_DATASETS_CACHE</code></td>
      <td>Sets the cache dir for downloaded Huggingface datasets.</td>
    </tr>
  </tbody>
</table>

> [!TIP]
> When `HF_TOKEN`, `WANDB_API_KEY`, `HF_HOME`, and `HF_DATASETS_CACHE` are set in your shell environment using `export`, they are automatically passed to `ray.sub`. For instance, if you set:
>
> ```sh
> export HF_TOKEN=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
> ```
> this token will be available to your NeMo RL run. Consider adding these exports to your shell configuration file, such as `~/.bashrc`.

#### Advanced Environment Configuration
<table>
  <thead>
    <tr>
      <th>Environment Variable<br />(and default)</th>
      <th>Explanation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>UV_CACHE_DIR_OVERRIDE</code></td>
      <td>By default, this variable does not need to be set. If unset, <code>ray.sub</code> uses the<br /><code>UV_CACHE_DIR</code> defined within the container (defaulting to <code>/root/.cache/uv</code>).<br /><code>ray.sub</code> intentionally avoids using the <code>UV_CACHE_DIR</code> from the user&#x27;s host<br />environment to prevent the host&#x27;s cache from interfering with the container&#x27;s cache.<br />Set <code>UV_CACHE_DIR_OVERRIDE</code> if you have a customized <code>uv</code> environment (e.g.,<br />with pre-downloaded packages or specific configurations) that you want to persist<br />and reuse across container runs. This variable should point to a path on a shared<br />filesystem accessible by all nodes (head and workers). This path will be mounted<br />into the container and will override the container&#x27;s default <code>UV_CACHE_DIR</code>.</td>
    </tr>
    <tr>
      <td><code>CPUS_PER_WORKER=128</code></td>
      <td>CPUs each Ray worker node claims. Default is <code>16 * GPUS_PER_NODE</code>.</td>
    </tr>
    <tr>
      <td><code>GPUS_PER_NODE=8</code></td>
      <td>Number of GPUs each Ray worker node claims. To determine this, run <code>nvidia-smi</code> on a worker node.</td>
    </tr>
    <tr>
      <td><code>BASE_LOG_DIR=$SLURM_SUBMIT_DIR</code></td>
      <td>Base directory for storing Ray logs. Defaults to the Slurm submission directory ([SLURM_SUBMIT_DIR](https://slurm.schedmd.com/sbatch.html#OPT_SLURM_SUBMIT_DIR)).</td>
    </tr>
    <tr>
      <td><code>NODE_MANAGER_PORT=53001</code></td>
      <td>Port for the Ray node manager on worker nodes.</td>
    </tr>
    <tr>
      <td><code>OBJECT_MANAGER_PORT=53003</code></td>
      <td>Port for the Ray object manager on worker nodes.</td>
    </tr>
    <tr>
      <td><code>RUNTIME_ENV_AGENT_PORT=53005</code></td>
      <td>Port for the Ray runtime environment agent on worker nodes.</td>
    </tr>
    <tr>
      <td><code>DASHBOARD_AGENT_GRPC_PORT=53007</code></td>
      <td>gRPC port for the Ray dashboard agent on worker nodes.</td>
    </tr>
    <tr>
      <td><code>METRICS_EXPORT_PORT=53009</code></td>
      <td>Port for exporting metrics from worker nodes.</td>
    </tr>
    <tr>
      <td><code>PORT=6379</code></td>
      <td>Main port for the Ray head node.</td>
    </tr>
    <tr>
      <td><code>RAY_CLIENT_SERVER_PORT=10001</code></td>
      <td>Port for the Ray client server on the head node.</td>
    </tr>
    <tr>
      <td><code>DASHBOARD_GRPC_PORT=52367</code></td>
      <td>gRPC port for the Ray dashboard on the head node.</td>
    </tr>
    <tr>
      <td><code>DASHBOARD_PORT=8265</code></td>
      <td>Port for the Ray dashboard UI on the head node. This is also the port<br />used by the Ray distributed debugger.</td>
    </tr>
    <tr>
      <td><code>DASHBOARD_AGENT_LISTEN_PORT=52365</code></td>
      <td>Listening port for the dashboard agent on the head node.</td>
    </tr>
    <tr>
      <td><code>MIN_WORKER_PORT=54001</code></td>
      <td>Minimum port in the range for Ray worker processes.</td>
    </tr>
    <tr>
      <td><code>MAX_WORKER_PORT=54257</code></td>
      <td>Maximum port in the range for Ray worker processes.</td>
    </tr>
  </tbody>
</table>

> [!NOTE]
> For the most part, you will not need to change ports unless these
> are already taken by some other service backgrounded on your cluster.

## Kubernetes

TBD
