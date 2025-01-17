# Quickstart


## Dependencies

You must have the following things:

- System with at least 48gb of VRAM
- Python >=3.10
- Docker with [gpu support](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
- OpenAI API key: This will be used very rarely to handle the finetuning. Expect about ~1000 gpt4o calls per month.
- Github Token
- Wandb account
- AWS credentials
- Huggingface token

We recommend using two GPUs, most preferably two A6000s. Furthermore, we recommend running on the same box as SN20, as both subnets when on the same box only need 2 gpus total.

## Getting started


## Installation

Ensure that you have Docker with GPU support, you can choose to follow either of the instructions:

- [Official Guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) 
- [Quick and Dirty Stack Overflow Guide](https://stackoverflow.com/questions/75118992/docker-error-response-from-daemon-could-not-select-device-driver-with-capab)



This repository requires python3.11, follow the commands below to install it if you do not already have it.

ONLY RUN THE FOLLOWING COMMANDS IF YOU DO NOT HAVE PYTHON INSTALLED
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.11 python3.11-venv
```

Ensure that your python version is 3.11 before continuing:
```bash
python3 --version
```

If the above doesnt return `python3.11` try using the command `python3.11` instead. If the cmd `python3.11` works, use that in place of every python command below. 

YOU WILL GET SOME ERRORS ABOUT THE PYTHON VERSION, IGNORE THEM.

After ensuring you have python run the following commands:
```bash
git clone https://github.com/brokespace/code
cd code
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install --use-deprecated=legacy-resolver -r requirements.txt
python3 -m pip install --use-deprecated=legacy-resolver -e .
python3 -m pip uninstall uvloop # b/c it causes issues with threading/loops
```


##### Run the LLM image

The following command will run vllm on gpu:0. The `--gpu-memory-utilization` flag signifies how much of the gpu vllm will claim. 


```bash
sudo docker run -d -p 8028:8000  --gpus device=0 --ipc host --name qwen14b docker.io/vllm/vllm-openai:latest --model Qwen/Qwen2.5-14B-Instruct-GPTQ-Int4 --max-model-len 16384 --dtype half --gpu-memory-utilization 0.3
```

#### Setup Wandb 

This is optional, but recommended make sure you login

```bash
wandb login
```


#### Setup your dotenv

Copy `.env.example` to `.env` - `cp .env.example .env`. Then edit the `.env` file with the github token you get below

#### Get a Github Token

We require github tokens, to get one follow the instructions [here](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens), or below.

1. Go to [Github](http://Github.com)
2. Open the top right menu and select `Settings`
3. Go to the bottom left and select `Developer Settings`
4. Go to either `Tokens (classic)` or `Fine-grained tokens`
5. Generate a new token and place it in the .env

#### Get an OpenAI Key

To use OpenAI's services, you need to obtain an API key. Follow the steps below to get your OpenAI API key:

1. Go to the [OpenAI website](https://www.openai.com/).
2. Sign up for an account if you don't already have one, or log in if you do.
3. Navigate to the API section of your account.
4. Generate a new API key.
5. Copy the API key and store it in a secure location.

Once you have your OpenAI API key, add it to your `.env` file like this:

```
OPENAI_API_KEY=<your openai api key>
```


#### Setup AWS

We use AWS to download the github dataset. Follow the steps below to setup your AWS credentials.

##### 1. Sign In to AWS Management Console
Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in with your AWS credentials.

##### 2. Navigate to My Security Credentials
- Click on your account name at the top right corner of the AWS Management Console.
- Select "Security Credentials" from the dropdown menu.

##### 3. Create New Access Key
- In the "My Security Credentials" page, go to the "Access keys" section.
- Click on "Create Access Key".
- A pop-up will appear showing your new Access Key ID and Secret Access Key.

##### 4. Download Credentials
- Download the `.csv` file containing these credentials or copy them to a secure location.
  - **Important**: This is the only time you will be able to view the secret access key. If you lose it, you will need to create new credentials.

##### 5. Alternative - create dedicated user (more secure)
- Navigate to IAM
- Create New User - name the user `sn-45` or similar
- Attach `AmazonS3FullAccess` policy to user or apply the following permissions
- Or provide the following permissions:

```json
{
          "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::softwareheritage",
                "arn:aws:s3:::softwareheritage/*"
            ]
        }
```

##### 6. Place the credentials in your .env file
The access key id and secret access key should look like this within your `.env` file:

```
AWS_ACCESS_KEY_ID=<your access key id>
AWS_SECRET_ACCESS_KEY=<your secret access key>
```

#### Setup huggingface 

You must get access to the stack v2 datasets, to do so go [here](https://huggingface.co/datasets/bigcode/the-stack-v2), and [here](https://huggingface.co/datasets/bigcode/the-stack-v2-train-smol-ids).

Once you have access, get your huggingface token:

1. Go to [huggingface](https://huggingface.co/)
2. Click on your avatar in the top right corner and select `Settings`
3. Go to the bottom left and select `Access Tokens`
4. Click on `New token`
5. Ensure the token has `Read access to contents of all public gated repos you can access` permissions.
6. Copy the token and place it in the `.env` file

The huggingface token should look like this within your `.env` file:

```
HF_TOKEN=<your huggingface token>
```


#### Start the validator



```bash
source .venv/bin/activate
python3 scripts/start_validator.py
    --netuid 45
    --subtensor.network <finney/local/test>
    --neuron.device cuda
    --wallet.name <your wallet> # Must be created using the bittensor-cli
    --wallet.hotkey <your hotkey> # Must be created using the bittensor-cli
    --logging.debug # Run in debug mode, alternatively --logging.trace for trace mode
    --axon.port # VERY IMPORTANT: set the port to be one of the open TCP ports on your machine
    --neuron.model_url # OPTIONAL, if you are hosting the model somewhere else other then port 8028
    --neuron.vllm_api_key # OPTIONAL, only use if your vllm instance has an api key requirement
    --wandb.on True # default is true but you can disable
    --neuron.finetune_gpu_id 0 # Defaults to 0, if using two gpu's set to 1
```

