# 手把手搭建DeepSeek本地大模型

不得不说，最近大模型火得一塌糊涂，尤其是DeepSeek大杀四方。

奈何DeepSeek与黑客日夜大战，影响我使用大模型。于是乎，我把它塞进了我的PC机。

在正式开始部署本地大模型之前，先对齐基本概念。

## 基本概念

### 大型语言模型（LLM）

大型语言模型（Large Language Models，简称LLM），是一种基于深度学习的自然语言处理模型，通过大规模语料库的训练，能够生成、理解和处理复杂的文本数据。

### DeepSeek

![alt text](images/deepseek.png)

[DeepSeek](https://www.deepseek.com/)是深度求索公司开发的深度学习模型搜索引擎，当前最新版本DeepSeek-R1。号称在GPU受限的前提下精进算法参数就足以吊打闭源的ChatGPT-4o。到底有没有这么神奇，还需要时间的检验。另外需要知道的一点是，参数集规模。比如DeepSeek-R1:1.5B，这里的1.5B指的是参数集规模，B是Billion（十亿）的缩写，1.5B指的是15亿参数集规模。参数集规模越大，意味着回答更精准，但同时对于机器的硬件要求也就越高，尤其是GPU性能和内存容量。

### Ollama

<img  src="images/ollama.png" width="300" />

Ollama提供了一种简化的方式，使得开发者能够在没有云计算依赖的情况下直接在本地机器上运行强大的语言模型，从而降低延迟并提高隐私保护。简单来说，Ollama是一个容器，或者说它就是搭载本地大模型的一艘货轮。想要把DeepSeek、Qwen、LLaVA等等大模型都装到本机上运行，就需要把Ollama先配好。当然，这个可能是最不困难的事情。

### Open WebUI

<img  src="images/openwebui.png" width="300" />

Open WebUI是一个开源的 Web 用户界面（UI）框架，用于简化 Web 应用程序的开发和管理。它通常用于提供可自定义的、易于操作的界面，支持用户通过浏览器与应用进行交互。Ollama提供的是一个极客风格命令行聊天界面，还需要给它换身衣服才好出来见人，这就需要Open WebUI来套个壳。当然，有了Open WebUI之后，我们还能很方便的选择其它大模型，或者与大模型语音交互。又或者一不留神，就开发了一个AI语音聊天机器人。

### Docker

<img  src="images/docker.png" width="300" />

Docker是一个开源的容器化平台，用于自动化应用程序的部署、扩展和管理。它通过将应用程序及其所有依赖项（包括操作系统库和环境）打包到一个标准化的容器中，使得应用能够在任何环境中一致运行，无论是开发、测试还是生产环境。换言之，Docker是另一艘货轮，用来运载已经打包好的服务，比如我们后面要讲的Open WebUI。

### Portainer

<img  src="images/Portainer.png" width="300" />

Portainer 是一个基于 Web 的轻量级 Docker 容器管理工具，它使得管理和监控 Docker 环境变得更加简单和直观。通过 Portainer，用户可以通过图形化界面对 Docker 容器、镜像、网络、数据卷等资源进行管理，无需使用命令行，从而提高了使用 Docker 的便捷性和效率。

## 部署Ollama

部署OLlama有3种方式：  

### 1.官网下载后手动安装

适用于绝大多数桌面操作系统，但**不适合Ubuntu 22 版本系统**。

Ollama官网[下载](https://ollama.com/download)安装包，直接安装。

### 2.命令安装

同样适用于绝大多数桌面操作系统，并且有一定技术基础的老手，但仍旧**不适合Ubuntu 22 版本系统**。

执行以下代码，命令方式安装OLlama。

``` shell
curl -fsSL https://ollama.com/install.sh | sh
```

### 3.Docker部署

适用于**Ubuntu 22版本系统**用户。

``` shell
docker run -d \
    -p 11434:11434\
    --runtime=nvidia \
    -e NVIDIA_VISIBLE_DEVICE=all \
    --name ollama_container \
    ollama/ollama
```

之后可通过如下方式进入Ollama容器。

``` shell
docker exec -it ollama_container /bin/bash
```

本地浏览器打开(<http://localhost:9000>)页面，看到如下提示，就说明Ollama正常运行了。

``` test
Ollama is running
```

## 部署DeepSeek

DeepSeek-R1是最近发布的大模型，号称超越ChatGPT-4o。

为方便演示，使用DeepSeek-R1的1.5B小参数集模型。

执行以下代码，拉取DeepSeek-R1:1.5B版本。

``` shell
ollama pull deepseek-r1:1.5b
```

``` tex
pulling manifest
pulling aabd4debf0c8... 100% ▕████████████████▏ 1.1 GB
pulling 369ca498f347... 100% ▕████████████████▏  387 B
pulling 6e4c38e1172f... 100% ▕████████████████▏ 1.1 KB
pulling f4d24e9138dd... 100% ▕████████████████▏  148 B
pulling a85fe2a2e58e... 100% ▕████████████████▏  487 B
verifying sha256 digest
writing manifest
success
```

现在可以跟DeepSeek聊天了，启动Ollama并运行DeepSeek-R1大模型。

``` shell
ollama run deepseek-r1:1.5b "天空为什么是蓝色的？"
```

``` shell
ollama run deepseek-r1:1.5b "讲讲哪吒闹海的故事"
```

或者，你可以执行以下命令，进行多轮对话。

``` shell
ollama run deepseek-r1:1.5b
```

## 搭建Web服务

终端极客版本的大模型用起来看着不那么友好，还是Web界面操作更有亲和力。

### 部署Docker

附上Docker桌面版下载（命令安装）地址：

- [Mac版本下载](https://docs.docker.com/desktop/setup/install/mac-install/)  
- [Windows版本下载](https://docs.docker.com/desktop/setup/install/windows-install/)  
- [Linux版本命令安装](https://docs.docker.com/desktop/setup/install/linux/)  

注意区分CPU架构版本，只要能正常启动Docker就可以部署Open WebUI了。

### 部署Portainer

Portainer同样也是一个Docker镜像，执行以下命令直接拉取Docker镜像并部署容器。

``` shell
docker volume create portainer_data
docker run -d -p 9000:9000 -p 9443:9443 --name=portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

然后登陆(<http://localhost:9000>)注册并登录后就可以进入管理界面。

<img  src="images/portainer-1.png" width="500" />

### 部署Open WebUI

执行以下代码，拉取Docker镜像，并启动Open WebUI服务

``` shell
docker run -d \
    -v open-webui:/app/backend/data \
    --add-host=host.docker.internal:host-gateway \
    -p 8080:8080 \
    --name open-webui \
    --restart always ghcr.io/open-webui/open-webui:main
```

或者

``` shell
docker run -d \
    -v open-webui:/app/backend/data \
    -e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
    -p 8080:8080 \
    --name open-webui \
    --restart always ghcr.io/open-webui/open-webui:main
```

第一次启动Open WebUI时，需要一个等待的过程，在后台日志中会看到如下内容。

``` text
WARNI [open_webui.env] 
WARNING: CORS_ALLOW_ORIGIN IS SET TO '*' - NOT RECOMMENDED FOR PRODUCTION DEPLOYMENTS.
INFO  [open_webui.env] Embedding model set: sentence-transformers/all-MiniLM-L6-v2
WARNI [langchain_community.utils.user_agent] USER_AGENT environment variable not set, consider setting it to identify your requests.
  ___                    __        __   _     _   _ ___
 / _ \ _ __   ___ _ __   \ \      / /__| |__ | | | |_ _|
| | | | '_ \ / _ \ '_ \   \ \ /\ / / _ \ '_ \| | | || |
| |_| | |_) |  __/ | | |   \ V  V /  __/ |_) | |_| || |
 \___/| .__/ \___|_| |_|    \_/\_/ \___|_.__/ \___/|___|

|
v0.5.3 - building the best open-source AI user interface.
https://github.com/open-webui/open-webui

Fetching 30 files:   0%|          | 0/30 [00:00<?, ?it/s]
Fetching 30 files: 100%|██████████| 30/30 [00:00<00:00, 291946.91it/s]
INFO:     Started server process [1]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8080 (Press CTRL+C to quit)
```

现在可以直接打开(<http://localhost:8080>)访问Open WebUI了。

第一次使用Open WebUI需要注册管理员账号。

<img  src="images/first_use_openwebui.png" width="300" />

## 配置大模型

第一次使用Open WebUI时，可以在左上角`选择一个模型`点开，键入需要拉取的模型名称，点击`从Ollama.com拉取...`即可。

<img  src="images/configure_llm.png" width="300" />

接下来就可以使用Open WebUI进行大模型对话了。

## 附录：推荐模型

能聊天的大模型，除了DeepSeek，还是阿里云的通义千问（Qwen2/Qwen2.5），好玩的模型还有很多。LLaVA-Phi3能够识别图像，配合其它工具还能使用DeepSeek-Coder写代码。能在Ollama中运行的大模型还有很多。

| 模型名称      | 提供方        | 官网地址                                             | 说明                                                       |
|---------------|---------------|----------------------------------------------------|------------------------------------------------------------|
| LLaMA3        | Meta AI       | [LLaMA3官网](https://ai.facebook.com/tools/llama/) | 大型语言模型，专注于高效的自然语言处理任务。                       |
| DeepSeek-R1 | DeepSeek      | [DeepSeek官网](https://www.deepseek.com/)            | 深度求索公司开发的深度学习模型搜索引擎，当前最新版本DeepSeek-R1。        |
| TinyLlama     | 开源社区      | [TinyLlama官网](https://github.com/tinyllama)       | 轻量级的语言模型，适用于资源受限的环境。                           |
| Qwen2/Qwen2.5  | 阿里云        | [Qwen官网](https://qwen.ai/)                        | 多模态语言模型，支持文本和图像处理。                              |
| Phi-3         | 微软          | [Phi-3官网](https://www.microsoft.com/phi-3)         | 小型语言模型，专注于高效能和低资源消耗。                           |
| Gemma         | 谷歌          | [Gemma官网](https://ai.google/gemma)                | 开源语言模型，适用于多种自然语言处理任务。                          |
| WizardLM2     | 微软          | [WizardLM2官网](https://github.com/WizardLM)        | 专注于对话生成和任务导向对话的语言模型。                           |
| Orca Mini     | 开源社区      | [Orca Mini官网](https://github.com/orca-mini)       | 小型语言模型，专注于对话生成和问答任务。                           |
| Codellama     | Meta AI       | [Codellama官网](https://ai.meta.com/)               | 基于Llama 2的大型代码语言模型，提供多种版本，具备填充能力、支持大型输入上下文，以及编程任务的零样本指令跟随能力。 |
| StarCoder2    | BigCode       | [StarCoder2官网](https://huggingface.co/docs/transformers/main/model_doc/starcoder2) | 代码生成模型，支持600多种编程语言。                              |
| DeepSeek Coder| DeepSeek      | [DeepSeek官网](https://www.deepseek.com/)            | 一个强大的代码语言模型系列，适用于代码生成、代码补全等多种任务。        |
| LLaVA         | 开源社区      | [LLaVA官网](https://github.com/haotian-liu/LLaVA)   | 结合了视觉编码器（如 CLIP）和大型语言模型（如 LLaMA 或 Vicuna），能够处理图像和文本的联合任务，适合高性能设备。 |
| LLaVA-Phi3  | 微软 + 开源社区  | [LLaVA-Ph3官网](https://github.com/haotian-liu/LLaVA)  | 继承了 LLaVA 的多模态能力，同时在性能和效率上进行了优化，适合资源受限的环境。 |
