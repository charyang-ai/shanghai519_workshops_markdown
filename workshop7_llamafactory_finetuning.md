# LLM Fine-tuning with LLaMA Factory

Efficient fine-tuning is vital for adapting large language models (LLMs) to downstream tasks. LLaMA Factory is an open-source and user-friendly platform that streamlines the training and fine-tuning of large language models (LLMs) and multimodal models. It allows users to customize hundreds of pre-trained models locally with minimal coding. 
This workshop teaches you how to fine tune LLMs using LLaMA Factory on your AMD GPU.

---

## Workshop Setup

### docker solution 

#### Pull the llama factory workshop docker image

The workshop is based on the preinstalled ROCm Pytorch docker image with Ubuntu24.04 + ROCm7.2. Llama factory has been installed successfully in this image as the steps of [LLaMA-Factory Installation](https://llamafactory.readthedocs.io/en/latest/getting_started/installation.html#llama-factory)
Developers can pull this image from [llama_factory_workshop](https://hub.docker.com/r/nzhangnju/llama_factory_workshop)
 
NOTE: Suppose it has been done by AMD Remote Machine, Developer could skip this part

```
# Pull the llama factory workshop docker image
docker pull nzhangnju/llama_factory_workshop
```


#### Launch the workshop image 

Run the below command to launch the workshop image

```
docker run -itd --rm --device /dev/kfd --device /dev/dri  --privileged=true --group-add video --ipc=host --network=host --shm-size 32G --cap-add=SYS_ADMIN --cap-add=SYS_PTRACE --security-opt seccomp=unconfined  --name=llamafactory_demo nzhangnju/llama_factory_workshop
```

#### Attach the workshop docker container 

Developer can attach the running workshop container as the below sample command, and navigate to the llama factory root directory
```
docker attach llamafactory_demo
```
How to install llama factory can be found at its official document.For this workshop, we have installed it to save time. You can quickly verify if the installation was successful by using llamafactory-cli version. 
```
cd /LlamaFactory
llamafactory-cli version
```
### non-docker solution 

#### Install Llama Factory
we install llama factory for this workshop by source codes building: 

```
git clone --recursive https://github.com/zhangnju/LlamaFactory.git
cd LlamaFactory
pip install -e .
pip install -r requirements/metrics.txt
```
#### Download Model To Test

```
hf download Qwen/Qwen3-4B-Instruct-2507
```

#### Check llama factory Version 
```
llamafactory-cli version
```

## Workshop Steps 

This section will cover how to prepare fine-tuning datasets, configure LoRA/QLoRA parameters, run LoRA fine-tuning, test and export the fine-tuned models. 

### Dataset Preparation

LLaMA Factory supports fine-tuning datasets in the Alpaca format and ShareGPT format. All the available datasets have been defined in the `dataset_info.json`. If you are using a custom dataset, please make sure to add a dataset description in `dataset_info.json` and specify the dataset name before training. Details can be found in llama factory official documents.

In this workshop, we will use the identity datasets as an example.

### Fine-tuning parameter configuration

LLaMA Factory supports multiple fine-tuning schemes.

| Fine-Tuning schemes | LLaMA Factory Examples |
|-----------|------|
| Full-Parameter    | examples/train_full |
| LoRA fine-tuning  | examples/train_lora |
| QLoRA fine-tuning | examples/train_qlora |

These example configuration files have specified model parameters, fine-tuning method parameters, dataset parameters, evaluation parameters, and more. You can configure them according to your own needs. In this workshop, we will use [qwen3_lora_sft.yaml](https://github.com/hiyouga/LlamaFactory/blob/main/examples/train_lora/qwen3_lora_sft.yaml). 

**Key parameters explained:**
- `model_name_or_path` - HuggingFace Model name or local model file path.
- `stage` - Training stage. Options: rm (reward modeling), pt (pretrain), sft (Supervised Fine-Tuning), PPO, DPO, KTO, ORPO.
- `do_train` - true for training, false for evaluation
- `finetuning_type` - Fine-tuning method. Options: freeze, lora, full
- `lora_rank` - The dimensionality of the low-rank matrix used in LoRA, typical values: 4, 6, 8, 16 (smaller values = fewer parameters = faster fine-tuning; larger values = better task adaptation but higher resource usage).
- `lora_target` - Target modules for LoRA method. Default: all.
- `dataset` - Dataset(s) to use. Use “,” to separate multiple datasets
- `output_dir` - File-tuning Output path
- `logging_steps` - Logging interval in steps
- `save_steps` - Model checkpoint saving interval.
- `overwrite_output_dir` - Whether to allow overwriting the output directory.
- `per_device_train_batch_size` - Training batch size per device.
- `gradient_accumulation_steps` - Number of gradient accumulation steps.
- `learning_rate` - Learning rate
- `num_train_epochs` - Number of training epochs
- `lr_scheduler_type` - Learning rate schedule. Options: linear, cosine, polynomial, constant, etc.
- `warmup_ratio` - Learning rate warmup ratio

In this workshop, We use the default configurations to run fine-tuning on AMD Radeon™ GPUs.


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
echo "What's good to eat in Shanghai? Please reply in Chinese." | llamafactory-cli chat examples/inference/qwen3_lora_sft.yaml
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
