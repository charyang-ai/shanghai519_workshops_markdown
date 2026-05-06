# LLM Fine-tuning with LLaMA Factory

Efficient fine-tuning is vital for adapting large language models (LLMs) to downstream tasks. LLaMA Factory is an open-source and user-friendly platform that streamlines the training and fine-tuning of large language models (LLMs) and multimodal models. It allows users to customize hundreds of pre-trained models locally with minimal coding. 
This workshop teaches you how to fine tune LLMs using LLaMA Factory on your AMD GPU.

---

## Workshop Setup

### Pull the llama factory workshop docker image

The workshop is based on the preinstalled ROCm Pytorch docker image with Ubuntu24.04 + ROCm7.2. Llama factory has been installed successfully in this image as the steps of [LLaMA-Factory Installation](https://llamafactory.readthedocs.io/en/latest/getting_started/installation.html#llama-factory)
Developers can pull this image from [llama_factory_workshop](https://hub.docker.com/r/nzhangnju/llama_factory_workshop)
 
NOTE: Suppose it has been done by AMD Remote Machine, Developer could skip this part

```
# Pull the llama factory workshop docker image
docker pull nzhangnju/llama_factory_workshop
```


### Launch the workshop image 

Run the below command to launch the workshop image

```
docker run -itd --rm --device /dev/kfd --device /dev/dri  --privileged=true --group-add video --ipc=host --network=host --shm-size 32G --cap-add=SYS_ADMIN --cap-add=SYS_PTRACE --security-opt seccomp=unconfined  --name=llamafactory_demo nzhangnju/llama_factory_workshop
```

### Run jupyter notebook inside the docker container 

Developer can attach the running workshop container as the below sample command,
```
docker attach llamafactory_demo
```

After attaching the container, developers can navigate to the workshop directory and then run jupyter notebook
```
cd /llama-factory-workshop/
jupyter-lab --ip=0.0.0.0 --port=8888 --no-browser --allow-root
```

when Jupyter notebook is launched successfully,copy and paste the notebook URL to the web browser of AMD GPU machine 


## Workshop Steps 

Open llama-factory-finetuning.ipynb and begin to run the below steps of this workshop.

NOTE: The below steps should be run inside jupyter notebook. 

### Check Llama factory version 

Download the source code from LLaMA Factory official GitHub repository, and install its dependencies. How to install llama factory can be found at its official document.For this workshop, we have installed it to save the time. You can quickly verify if the installation was successful by using llamafactory-cli version. 
```
cd /LlamaFactory
llamafactory-cli version
```
---

### Run LLaMA Factory Fine-tuning

llamafactory-cli is the official command-line interface (CLI) tool for LLaMA Factory, developed to simplify end-to-end LLM workflows (data preparation → fine-tuning → evaluation → deployment) without writing complex code.

For training/fine-tuning, llamafactory-cli train is the core subcommand of the LLaMA Factory CLI. It abstracts fine-tuning workflows (data preprocessing, hyperparameter tuning, hardware optimization) into a single CLI command, supporting multiple fine-tuning paradigms (LoRA/QLoRA/Full Fine-Tuning) and optimized for low-resource GPUs (e.g., QLoRA on 16GB VRAM). 

For this workshop, developer can run the below command to run llama factory fine-tuning
```
llamafactory-cli train examples/train_lora/qwen3_lora_sft.yaml
```
---

### Test the Fine-tuned Model

llamafactory-cli chat is designed for interactive chat/inference with LLMs (both base models and LoRA-fine-tuned models). LLaMA Factory provides the sample configuration to run inference of fine-tuned models in examples/inference. You can also modify this sample configuration to change the settings such as the inference backend.

For this workshop, developer can run the below command to test the model
```
echo "上海有什么好吃的？" | llamafactory-cli chat examples/inference/qwen3_lora_sft.yaml
```
---

### Export the Fine-tuned Model

For production use-cases, the pre-trained model and the LoRA adapter need to be merged and exported into a single model. This merged model can be used as a normal HuggingFace model file. LLaMA Factory provides the sample configurations in examples/merge_lora.

Use the following command to export the Qwen3 finetuned model:
```
llamafactory-cli export examples/merge_lora/qwen3_lora_sft.yaml
```
---


*Workshop duration: ~30 minutes for this llama factory work.*
