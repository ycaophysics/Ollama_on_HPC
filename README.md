# Running Ollama on NERSC Perlmutter
This guide provides a step-by-step process for installing and running Ollama in a user-space environment on NERSC’s Perlmutter supercomputer. Since Perlmutter’s HPC environment doesn’t allow sudo or user-managed systemd services, you’ll install and run Ollama entirely in your home or scratch directories, using your user environment.

Prerequisites
NERSC Account & Access to Perlmutter: You need an active NERSC account and SSH access to Perlmutter.
No sudo: Installation is done in user space.
No systemd Services: You will run ollama serve manually.
GPU Access: If you plan to use GPUs, request a compute node allocation (salloc) and ensure CUDA is available.
Choosing an Installation Directory
Perlmutter’s home directory ($HOME) has limited quota (~20GB), which may not be sufficient for large model files. It’s recommended to use $SCRATCH or CFS for storage-intensive installations.

For this example, we’ll use $SCRATCH:

bash
Copy code
mkdir -p /pscratch/sd/y/ycao910/Ollama
cd /pscratch/sd/y/ycao910/Ollama
Installing Ollama in User Space
Download Ollama (Linux AMD64, for x86_64 front-ends):

bash
Copy code
curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
tar -xzf ollama-linux-amd64.tgz
# Move binaries and libs into a cleaner structure
mkdir -p bin lib
mv usr/bin/* bin/
mv usr/lib/* lib/
rm -r usr
Set Your PATH:

Add Ollama’s bin directory to your PATH:

bash
Copy code
echo 'export PATH="/pscratch/sd/y/ycao910/Ollama/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
Check that Ollama is in your path:

bash
Copy code
which ollama
# Should show /pscratch/sd/y/ycao910/Ollama/bin/ollama
CUDA Environment:

Perlmutter uses NVIDIA GPUs with CUDA toolkits provided. Load CUDA if needed:

bash
Copy code
module load cudatoolkit
nvidia-smi
# Should show GPU info if on a GPU node.
Running Ollama
Because you’ll be running on a shared HPC system:

Interactive Session With GPU:

Ollama benefits from GPU acceleration. Launch an interactive job on a GPU node:

bash
Copy code
salloc -N 1 -C gpu -G 4 -t 60 -A <YOUR_PROJECT> --qos=interactive
module load cudatoolkit
cd /pscratch/sd/y/ycao910/Ollama
ollama serve
Ollama will start serving on 127.0.0.1:11434.

In another terminal (with the same environment), you can verify:

bash
Copy code
ollama -v
If you see version info, Ollama is running.

Storing Models in Scratch
By default, Ollama stores models in ~/.ollama/models. To store them in $SCRATCH:

Create a models directory in $SCRATCH:

bash
Copy code
mkdir -p /pscratch/sd/y/ycao910/Ollama/models
Set the OLLAMA_MODELS environment variable before starting ollama serve:

bash
Copy code
echo 'export OLLAMA_MODELS="/pscratch/sd/y/ycao910/Ollama/models"' >> ~/.bashrc
source ~/.bashrc
Check:

bash
Copy code
echo $OLLAMA_MODELS
# should print /pscratch/sd/y/ycao910/Ollama/models
Stop and restart ollama serve to ensure it picks up the new OLLAMA_MODELS location:

bash
Copy code
pkill ollama
ollama serve
Now when you pull models:

bash
Copy code
ollama pull llama3.3
They will download into $SCRATCH instead of $HOME.

Creating a Custom Model With a System Prompt
Ollama uses a Modelfile format, not YAML, for custom model configurations. For example, to create a model with a custom system message:

Create a Modelfile (no .yaml extension, and note the instructions in uppercase):

bash
Copy code
cat > custom_llama.Modelfile <<EOF
FROM llama3.3

SYSTEM """
I am a research scientist working at the intersection of AI and high-performance computing...
(Your entire system prompt here)
"""
EOF
Important:

Do not use YAML syntax; follow the Modelfile instructions exactly.
FROM and SYSTEM must be uppercase and at the start of lines.
The system prompt goes in triple quotes.
Create the new model:

bash
Copy code
ollama create ai4sci -f custom_llama.Modelfile
Run the custom model:

bash
Copy code
ollama run ai4sci "How can I optimize my Python code for HPC simulations?"
Your custom system prompt will be applied automatically.

Troubleshooting
Disk Quota Exceeded: If you see a quota error, ensure OLLAMA_MODELS is set and ollama serve was restarted. Ollama must be started after setting OLLAMA_MODELS.
Parsing Errors in Modelfile:
Make sure you’re using the correct Modelfile instructions (FROM, SYSTEM, PARAMETER, etc.), no BOM, no YAML keys. Remove hidden characters or re-create the file using cat > filename method.
Summary
Install Ollama in $SCRATCH, no sudo needed.
Set PATH and OLLAMA_MODELS environment variables before starting ollama serve.
Run ollama serve in a GPU-allocated session on Perlmutter.
Use Modelfile DSL to create custom models with a system prompt.
Interact with Ollama via ollama run, ollama pull, and other commands from a separate terminal.
This setup allows you to leverage Ollama’s capabilities and NERSC’s HPC infrastructure for large-scale model inference, data analysis, and research-oriented workflows.
