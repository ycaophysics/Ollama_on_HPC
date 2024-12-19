# Installing and Running Ollama on NERSC Perlmutter

This guide provides step-by-step instructions for installing and running [Ollama](https://ollama.ai) on NERSC’s Perlmutter system. Since Perlmutter is a shared HPC environment with no `sudo` access and no user-level systemd, we will perform a user-space installation and configure Ollama to store models in `$SCRATCH`.

## Prerequisites

- **NERSC Account & Environment:**  
  Ensure you have an active NERSC account and can log into Perlmutter.
  
- **HPC Considerations:**  
  - You don’t have `sudo` privileges.
  - No `systemd` services for users.
  - Models and data should be stored in `$SCRATCH` due to limited `$HOME` quota.
  
- **GPU Acceleration:**  
  Perlmutter provides NVIDIA A100 GPUs. Ollama supports GPU acceleration, so ensure you use a GPU node and the CUDA environment.

## Steps

### 1. Allocate a GPU Node (Optional for Testing)
To fully utilize GPUs, request an interactive compute node with GPUs:
```bash
salloc -N 1 -C gpu -G 4 -t 60 -A <YOUR_ACCOUNT> --qos=interactive
module load cudatoolkit
```
This setup allows you to leverage Ollama’s capabilities and NERSC’s HPC infrastructure for large-scale model inference, data analysis, and research-oriented workflows.

If you just want to set up Ollama on the login node (for installation only), you can skip this step. For actual model inference or pulling large models, using a compute node is recommended.

### 2. Choose an Installation Directory in $SCRATCH
Since $HOME has a small quota, install Ollama in $SCRATCH. Adjust the path as needed:
```
mkdir -p /pscratch/sd/y/<user_id>/Ollama
cd /pscratch/sd/y/<user_id>/Ollama
```

### 3. Download and Extract Ollama Binaries
Download the Linux AMD64 tarball and extract it locally:
```
curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
tar -xzf ollama-linux-amd64.tgz
```

This creates a usr directory with bin and lib inside.
Now you have ollama binaries in /pscratch/sd/y/<user_id>/Ollama/bin.

### 4. Update Your PATH
Add Ollama’s bin directory to your PATH:

```
echo 'export PATH="/pscratch/sd/y/<user_id>/Ollama/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
Ensure ollama is now accessible:
```
which ollama
```
### 5. Set OLLAMA_MODELS Directory to $SCRATCH
By default, Ollama tries to store models in ~/.ollama/models, which may exceed $HOME quota. Instead, point it to $SCRATCH:
```
mkdir -p /pscratch/sd/y/ycao910/Ollama/models
echo 'export OLLAMA_MODELS="/pscratch/sd/y/<user_id>/Ollama/models"' >> ~/.bashrc
source ~/.bashrc
```
### 6. Start Ollama
Before downloading models, start the Ollama server (in the same session where OLLAMA_MODELS is set):
```
ollama serve
```
Leave this running. Open another terminal (with the same environment setup) for the next steps.

Note: On HPC systems, ollama serve runs as a foreground process. Use tmux or screen if you want to keep it running in the background.

### 7. Pull and Run Models
From another terminal, load CUDA and confirm environment:
```
module load cudatoolkit
```
Pull a model (e.g., llama3.3):
```
ollama pull llama3.3
```
This should now store the model in /pscratch/sd/y/<user_id>/Ollama/models instead of ~/.ollama/models.

Run a quick test:
```
ollama run llama3.3 "Hello, how can I optimize Python code for HPC?"
```

### 8. Custom Instructions (System Prompt)
To customize instructions or the “system” message, create a Modelfile in Ollama DSL (not YAML) format, for example:
```
cat > custom_llama.Modelfile <<EOF
FROM llama3.3
SYSTEM """
I am a research scientist working at the intersection of AI and high-performance computing...
(Your full system prompt here)
"""
EOF
```
Then create a new model:
```
ollama create ai4sci -f custom_llama.Modelfile
```
Run the custom model:
```
ollama run ai4sci "How can I parallelize my simulation code on NERSC?"
```
You will now get responses influenced by the custom system instructions.

### 9. Notes
Always ensure OLLAMA_MODELS is set before starting ollama serve.
GPU access requires compute node allocation (salloc).
$SCRATCH may be purged periodically; re-download models as needed.
Use tmux or screen to keep ollama serve running persistently.
