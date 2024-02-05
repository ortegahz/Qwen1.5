

# Qwen2-beta

<p align="center">
    <img src="https://qianwen-res.oss-accelerate.aliyuncs.com/assets/blog/qwen2-beta/logo_qwen2.jpg" width="400"/>
<p>


<!-- <p align="left">
    <a href="https://huggingface.co/Qwen">Hugging Face</a>&nbsp&nbsp | &nbsp&nbsp <a href="https://modelscope.cn/organization/qwen">ModelScope</a>&nbsp&nbsp |  &nbsp&nbsp <a href="https://qwenlm.github.io">Blog</a>&nbsp&nbsp |  &nbsp&nbsp <a href="assets/wechat.png">WeChat</a>&nbsp&nbsp | &nbsp&nbsp<a href="https://discord.gg/z3GAxXZ9Ce">Discord</a>&nbsp&nbsp
</p> -->

<p align="center">
        🤗 <a href="https://huggingface.co/Qwen">Hugging Face</a>&nbsp&nbsp | &nbsp&nbsp🤖 <a href="https://modelscope.cn/organization/qwen">ModelScope</a>&nbsp&nbsp | &nbsp&nbsp 📑 <a href="https://qwenlm.github.io">Blog</a> &nbsp&nbsp ｜ &nbsp&nbsp🖥️ <a href="https://modelscope.cn/studios/qwen/Qwen-72B-Chat-Demo/summary">Demo</a>
<br>
<a href="assets/wechat.png">WeChat (微信)</a>&nbsp&nbsp | &nbsp&nbsp<a href="https://discord.gg/z3GAxXZ9Ce">Discord</a>&nbsp&nbsp ｜  &nbsp&nbsp<a href="https://dashscope.aliyun.com">API</a> 
</p>


Visit our Hugging Face or ModelScope organization (click links above), search checkpoints with names starting with `Qwen2-beta-`, and you will find all you need! Enjoy!



## Introduction
This time, we upgrade Qwen to Qwen2-beta, the beta version of Qwen2. Similar to Qwen, it is still a decoder-only transformer model with SwiGLU activation, RoPE, multi-head attention. At this moment, we have achieved:
* 6 model sizes: 0.5B, 1.8B, 4B, 7B, 14B, and 72B;
* Significant model quality improvements in chat models;
* Strengthened multilingual capabilities in both base and chat models;
* All models support the context length of `32768` tokens;
* System prompts enabled for all models, which means roleplay is possible.
* No need of `trust_remote_code` anymore.

We have not integrated GQA and mixture of SWA and full attention in this version and we will add the features in the future version.


## News
* 2024.02.05: We released the Qwen2-beta series.

## Performance
Detailed evaluation results are reported in this <a href="https://qwenlm.github.io"> 📑 blog</a>.


## Requirements
* `transformers>=4.37.0`. 

> [!Warning]
> <div align="center">
> <b>
> 🚨 This is a must because `transformers` integrated Qwen2 codes in `4.37.0`.
> </b>
> </div>

## Quickstart

### 🤗 Hugging Face Transformers
For a quickstart, I advise you to use chat models. We often use base models only for post-training. Here is a simple code snippet:
```python
from transformers import AutoModelForCausalLM, AutoTokenizer
device = "cuda" # the device to load the model onto

model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen2-beta-7B-Chat", device_map="auto")
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2-beta-7B-Chat")

prompt = "Give me a short introduction to large language model."

messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": prompt}
]

text = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)

model_inputs = tokenizer([text], return_tensors="pt").to(device)

generated_ids = model.generate(model_inputs.input_ids, max_new_tokens=512, do_sample=True)

generated_ids = [output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)]

response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]
```

For quantized models, we advise you to use the GPTQ, AWQ, and GGUF correspondents, namely `Qwen-beta-7B-Chat-GPTQ`, `Qwen-beta-7B-Chat-AWQ`. 

### 🤖 ModelScope
We strongly advise users especially those in mainland China to use ModelScope. `snapshot_download` can help you solve issues concerning downloading checkpoints.

### Run locally

#### llama.cpp
Download our provided GGUF files or create them by yourself, and you can directly use them with the latest `llama.cpp` with a one-line command:
```shell
./main -m <path-to-file> -n 512 --color -i -cml -f prompts/chat-with-qwen.txt
```

#### Ollama
We are now on Ollama, and you can use `pull` and `run` to make things work.
```shell
ollama pull qwen2-beta
ollama run qwen2-beta
```
You can also add things like `::14B` to choose different models. Visit [ollama.ai](https://ollama.ai/) for more information.

#### LMStudio
Qwen2-beta has already been supported by [lmstudio.ai](https://lmstudio.ai/). You can directly use LMStudio with our GGUF files.



## Web UI

#### Text generation web UI
You can directly use `text-generation-webui` for creating a web UI demo. If you use GGUF, remember to install the latest wheel of `llama.cpp` with the support of Qwen2-beta.


#### llamafile
Clone [`llamafile`](https://github.com/Mozilla-Ocho/llamafile), run source install, and then create your own llamafile with the GGUF file following the guide [here](https://github.com/Mozilla-Ocho/llamafile?tab=readme-ov-file#creating-llamafiles). You are able to run one line of command, say `./qwen.llamafile`, to create a demo.


## Deployment
Now, Qwen2-beta is supported by multiple inference frameworks. Here we demonstrate the usage of `vLLM` and `SGLang`.

### vLLM
We advise you to use `vLLM>=0.3.0` to build OpenAI-compatible API service. Start the server with a chat model, e.g. `Qwen2-beta-7B-Chat`:
```shell
python -m vllm.entrypoints.openai.api_server --model Qwen/Qwen2-beta-7B-Chat
```

Then use the chat API as demonstrated below:

```shell
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "Qwen/Qwen2-beta-7B-Chat",
    "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Tell me something about large language models."}
    ]
    }'
```
```python
from openai import OpenAI
# Set OpenAI's API key and API base to use vLLM's API server.
openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"

client = OpenAI(
    api_key=openai_api_key,
    base_url=openai_api_base,
)

chat_response = client.chat.completions.create(
    model="Qwen/Qwen2-beta-7B-Chat",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Tell me something about large language models."},
    ]
)
print("Chat response:", chat_response)
```

### SGLang
Please install `SGLang` from source. Similar to `vLLM`, you need to launch a server and use OpenAI-compatible API service. Start the server first:
```shell
python -m sglang.launch_server --model-path Qwen/Qwen2-beta-7B-Chat --port 30000
```
You can use it in Python as shown below:
```python
from sglang import function, system, user, assistant, gen, set_default_backend, RuntimeEndpoint

@function
def multi_turn_question(s, question_1, question_2):
    s += system("You are a helpful assistant.")
    s += user(question_1)
    s += assistant(gen("answer_1", max_tokens=256))
    s += user(question_2)
    s += assistant(gen("answer_2", max_tokens=256))

set_default_backend(RuntimeEndpoint("http://localhost:30000"))

state = multi_turn_question.run(
    question_1="What is the capital of China?",
    question_2="List two local attractions.",
)

for m in state.messages():
    print(m["role"], ":", m["content"])

print(state["answer_1"])
```

## Finetuning
We provide a colab notebook for you to experience the simplest way of finetuning. However, we advise you to turn to more advanced training frameworks, including [Axolotl](https://github.com/OpenAccess-AI-Collective/axolotl), [Llama-Factory](https://github.com/hiyouga/LLaMA-Factory), [Swift](https://github.com/modelscope/swift), etc.


## License Agreement
Check the license of each model inside its HF repo. It is NOT necessary for you to submit a request for commercial usage.

## Citation
If you find our work helpful, feel free to give us a cite.

```
@article{qwen,
  title={Qwen Technical Report},
  author={Jinze Bai and Shuai Bai and Yunfei Chu and Zeyu Cui and Kai Dang and Xiaodong Deng and Yang Fan and Wenbin Ge and Yu Han and Fei Huang and Binyuan Hui and Luo Ji and Mei Li and Junyang Lin and Runji Lin and Dayiheng Liu and Gao Liu and Chengqiang Lu and Keming Lu and Jianxin Ma and Rui Men and Xingzhang Ren and Xuancheng Ren and Chuanqi Tan and Sinan Tan and Jianhong Tu and Peng Wang and Shijie Wang and Wei Wang and Shengguang Wu and Benfeng Xu and Jin Xu and An Yang and Hao Yang and Jian Yang and Shusheng Yang and Yang Yao and Bowen Yu and Hongyi Yuan and Zheng Yuan and Jianwei Zhang and Xingxuan Zhang and Yichang Zhang and Zhenru Zhang and Chang Zhou and Jingren Zhou and Xiaohuan Zhou and Tianhang Zhu},
  journal={arXiv preprint arXiv:2309.16609},
  year={2023}
}
```

## Contact Us
If you are interested to leave a message to either our research team or product team, join our [Discord](https://discord.gg/z3GAxXZ9Ce) or [WeChat groups](assets/wechat.png)!

