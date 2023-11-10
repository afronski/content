---
title: "Hey, computer: talk to me in Polish! Building a Polish-language speaking chatbot with Amazon SageMaker, Hugging Face, TRURL 2, and Streamlit"  
description: "Is it hard to use Large Language Models (LLMs), if you have to use non-English language? Let's put that to the test - by checking how hard would be to build a Polish-speaking chatbot on top of Amazon SageMaker, Hugging Face, and TRURL 2 - a fine-tuned version of LLaMA 2 that primary supports that language."
tags:
  - hugging-face
  - ai-ml
  - llm
  - sagemaker
  - python
spaces:
  - generative-ai
authorGithubAlias: afronski
authorName: Wojciech Gawroński
date: 2023-11-09
---

| ToC |
|-----|

Picture this: you have a brilliant idea for a product, that will need a conversational user interface e.g., a chatbot. As a builder, you know that *Large Language Models (LLMs)* can help you to tackle such challenge effectively. There is only one caveat: your idea requires, that conversations should happen in **your local language**.

In the technology world, we live in an English-centric landscape. If our products fit that definition or are built to be operated by tech-savvy people, it is extremely easy to forget that there are other languages needed. At the same time, technology should - and actually is - helping us to battle such challenges and gaps. There are models available already on the market that provide multilingual capabilities. So how hard or easy would be to cover a non-English language requirement? Let's check that in practice.

I may be biased - but for me, an obvious choice will be ... Polish language as I live in Poland (:poland:). It's definitely a choice in the middle, in terms of popularity - depending on the source - this language is around 25-30th place worldwide with a total population estimated around 44-45 million people. Then, the goal of this article is to explore *how challenging* will it be to build a chatbot that, will communicate with use of my native language.

## Hey, computer: talk to me, *but in Polish!*

For the majority of people that had experience with *Amazon Web Services* and technology, probably *TRURL 2* is the least familiar term in the title above - so allow me to start here. 

Design and train from scratch a completely new *LLM* specialised in our language of choice, would be definitely beyond my skills. This is an extensive research work, nowadays done by sizeable teams. Luckily, you can explore with me available options - and a perfect place would be a vibrant *AI community* called [Hugging Face](https://huggingface.co). If you never heard about it, the best explanation (or rather a mental model) is that *Hugging Face* is like *GitHub, but for machine learning models*. Across the sea of available models, you can find there one called [TRURL 2](https://huggingface.co/Voicelab#models), which is a fine-tuned version of [LLaMA 2](https://ai.meta.com/llama/#inside-the-model). According to the authors, it is trained on over 1.7B tokens (970k conversational Polish and English samples) with a large context of 4096 tokens. To be precise, *TRURL 2* is not a single model, but a collection of fine-tuned generative text models with 7 billion and 13 billion parameters, optimized for dialogue use cases. You can read more about this model in the official [blog post](https://voicelab.ai/trurl-is-here) published after model's release, prepared by the authors. 

Speaking about the authors, model was created by a Polish (:poland:) company called [Voicelab.AI](https://voicelab.ai). They are based in [Gdańsk](https://en.wikipedia.org/wiki/Gda%C5%84sk) and specialise in developing solutions related to *[Conversational Intelligence](https://voicelab.ai/conversational-intelligence)* and *[Cognitive Automation](https://voicelab.ai/cognitive-automation)*. It looks like, this is exactly what we needed! But before we will move to the practical part, it is worth answering two questions. 

First question: what is *LLaMA 2*? I am almost certain that even if you are completely new in generative AI space, you have already heard that name, but if you have not - [LLaMA 2](https://ai.meta.com/resources/models-and-libraries/llama/) is a family of *Large Language Models (LLMs)* developed by [Meta](https://ai.meta.com). This collection of pretrained and fine-tuned models is ranging in scale from 7 billion to 70 billion parameters. As the name suggests it is a 2nd iteration, and those models are trained on 2 trillion tokens and have double the context length of [LLaMA 1](https://ai.meta.com/blog/large-language-model-llama-meta-ai).

Last, but not least: is there any reason for this very peculiar name? What or who is *Trurl*? Even that word may look like a set of arbitrary letters put together - it actually makes sense. *Trurl* is one of the characters in the novel written by a famous Polish science-fiction writer, [Stanislaw Lem](https://en.wikipedia.org/wiki/Stanis%C5%82aw_Lem). In the ["Cyberiad"](https://en.wikipedia.org/wiki/The_Cyberiad), *Trurl* according to the author of the book, is a robotic engineer :robot:, *a constructor* with almost godlike abilities. In one of the stories, he creates a machine called "*Elektrybałt*", which by description - resembles today’s [GPT](https://en.wikipedia.org/wiki/Generative_pre-trained_transformer) solutions. You can clearly see, that this particular name is definitely not a coincidence.

## Prerequisites and Setup

Let's dive into the practice then! To document properly the exploration and development process, I have prepared a *GitHub* repository with a complete example which is available here: https://github.com/build-on-aws/deploying-trurl-2-on-amazon-sagemaker 

In order to successfully execute all steps in the given repository, you need to have the following prerequisites:

- Pre-installed tools:
    - Most recent *AWS CLI*.
    - *AWS CDK* in version 2.104.0 or higher.
    - Python 3.10 or higher.
    - Node.js v21.x or higher.
- Configured profile in the installed *AWS CLI* with credentials for your *AWS IAM* user account of choice.

## How to use that repository?

First, we need to configure local environment - and here are the steps:

```shell
# Do those in the repository root, after checking out.
# Ideally you should do them in a single terminal session.

# Node.js v21 is not yet supported inside JSII, but it works for that example - so "shush", please.
$ export JSII_SILENCE_WARNING_UNTESTED_NODE_VERSION=true

$ make
$ source ./.env/bin/activate

$ cd ../infrastructure
$ npm install

# Or `export AWS_PROFILE=<YOUR_PROFILE_FROM_AWS_CLI>`
$ export AWS_DEFAULT_PROFILE=<YOUR_PROFILE_FROM_AWS_CLI>
$ export AWS_USERNAME=<YOUR_IAM_USERNAME>

$ cdk bootstrap
$ npm run package
$ npm run deploy-shared-infrastructure

# Answer a few questions from the AWS CDK CLI wizard, and wait for the completion.

# Now you can push code from this repository to the created AWS CodeCommit git repository remote.
#
# Here you can find an official guide how to configure your local `git` for AWS CodeCommit:
#   https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up.html

$ git remote add aws <HTTPS_REPOSITORY_URL_PRESENT_IN_THE_CDK_OUTPUTS_FROM_PREVIOUS_COMMAND>
$ git push aws main

$ npm run deploy

# Again, answer a few questions from the AWS CDK CLI wizard, and wait for the completion.
```

Provided *infrastructure as code* written in *AWS Cloud Development Kit (CDK)* creates for us *AWS IAM* roles, a dedicated *Amazon VPC*, *Amazon S3* bucket for temporary data, *AWS CodeCommit* code repository, and *Amazon SageMaker Studio* that we will use as our main place for exploration.

So now, you can go to the newly created *Amazon SageMaker Studio* domain and open studio prepared for the provided username.

![Let's clone the repository - step 1](./images/setup-1-clone-repository.png)
![Let's clone the repository - step 2](./images/setup-2-clone-aws-codecommit-repository.png)

After studio's successfully start, in the next step you should clone *AWS CodeCommit* repository via the *user interface* of the *SageMaker Studio*.

Then, we need to install all project dependencies, and it would be great to enable the *Amazon CodeWhisperer* extension in your *SageMaker Studio* domain. If you would like to do that in the future on your own, [here you can find the exact steps](https://docs.aws.amazon.com/codewhisperer/latest/userguide/sagemaker-setup.html). In our case, steps up to 4th point are automated by the provided *infrastructure as code* solution. To execute the last two steps, you should open the *Launcher tab*.

![Open launcher](./images/setup-3-open-launcher.png)
![Open a system terminal, not the Image terminal](./images/setup-4-open-system-terminal.png)

Then, in the *System terminal* (not the *Image terminal*, see the different on the image above), run the following scripts:

```shell
# Those steps should be invoked in the *System terminal* inside *Amazon SageMaker Studio*:

$ cd deploying-trurl-2-on-amazon-sagemaker
$ ./install-amazon-code-whisperer-in-sagemaker-studio.sh
$ ./install-project-dependencies-in-sagemaker-studio.sh
```

Force refresh the browser tab with *SageMaker Studio* and you are ready to proceed. Now it's time to follow the steps inside the notebook available in the `trurl-2` directory and explore capabilities of *TRURL 2* model that you will deploy from *SageMaker Studio* notebook as an *Amazon SageMaker Endpoint*, and *CodeWhisperer* will be our AI-powered coding companion throughout the process.

### Exploration via *Jupyter Lab 3.0* in *Amazon SageMaker Studio*

So now it's time to deploy the model as an *SageMaker Endpoint* and explore the possibilities, but first - you need to locate and open the notebook.

![Locate the notebook file and open it](./images/exploration-1-open-notebook.png)

After opening it for a first time, new dialog window will appear asking you to configure *kernel* (environment that will be executing our code inside the notebook). Please configure it accordingly to the screenshot below. If that window did not appear, or you've closed that by accident - you can always find that in the opened tab with notebook under the 3rd icon on the screenshot above.

![Configure kernel for your exploration notebook](./images/exploration-2-configure-kernel.png)

Now you should be able to follow instructions inside the notebook by executing each cell with code one after another (via toolbar or keyboard shortcut: `CTRL/CMD + ENTER`). Keep in mind that before you will execute the clean-up section and invoke cell with `predictor.delete_endpoint()` you should *stop* :stop:, as we will need the running endpoint for the next section. 

The purpose of the exploration is to deploy an *Amazon SageMaker Endpoint* with *TRURL 2* model that has 7B parameters, and play with that in various use cases. You do that to learn how to call *SageMaker API*, understand the nuances of prompt engineering, or how to tweak model parameters - everything before you actually move to developing an actual *chatbot*.  

### Developing a prototype chatbot application with *Streamlit*

Now, we would like to use the deployed *Amazon SageMaker Endpoint* with *TRURL 2* model to develop a simple *chatbot* application that will play in a game of *20 questions* with us. You can find an example of such application developed with the use of *Streamlit* that can be invoked locally (assuming you have configured *AWS* credentials properly for the account), or inside *Amazon SageMaker Studio*.

*Streamlit* is a *Python* library, which allows you to write a shareable web applications without any front‑end experience required. It is the simplest way to prepare user interface for our *chatbot*, and then expose that to a wider audience e.g., to perform a user-based quality evaluation.

![Final user interface in *Streamlit* with a sample '20 questions' conversation - this time he won! :(](./images/streamlit-1-final-conversation.png)

In order to run it locally, you need to invoke following commands:

```shell
# If you use the previously used terminal session, you can skip this line:
$ source ./.env/bin/activate

$ cd trurl-2
$ streamlit run chatbot-app.st.py
```

If you would like to run that on the *Amazon SageMaker Studio*, you need to open *System terminal* as previously, and start it a bit differently:

```shell
$ conda activate studio
$ cd deploying-trurl-2-on-amazon-sagemaker/trurl-2
$ ./run-streamlit-in-sagemaker-studio.sh chatbot-app.st.py
```

Then, open the URL marked with a green color as on the screenshot below:

![Final user interface in *Streamlit* with a sample '20 questions' conversation - this time he won! :(](./images/streamlit-2-run-on-sagemaker-studio.png)

Remember, that no matter then way you will choose to run it, in order to communicate with chatbot you need to pass the name of *Amazon SageMaker Endpoint* in the text field inside sidebar on the left-hand side. How to find that? You can either look it up in the *[Amazon SageMaker Endpoints tab](https://eu-west-1.console.aws.amazon.com/sagemaker/home#/endpoints)* (keep in mind it is neither *URL*, nor *ARN* - **only the name**) or refer to the provided `endpoint_name` value inside *Jupyter* notebook, when you invoked `huggingface_model.deploy(...)` operation.

If you are interested in detailed explanation how we can run *Streamlit* inside *Amazon SageMaker Studio*, have a look on [this blog post](https://aws.amazon.com/blogs/machine-learning/build-streamlit-apps-in-amazon-sagemaker-studio) from the official *AWS* blog. 

In terms of application code, you can review the whole implementation inside the attached [GitHub repository](https://github.com/build-on-aws/deploying-trurl-2-on-amazon-sagemaker), available [in a single *Python* file](https://github.com/build-on-aws/deploying-trurl-2-on-amazon-sagemaker/blob/main/trurl-2/chatbot-app.st.py). The main logic of the chatbot interaction is in between [the lines 111-125](https://github.com/build-on-aws/deploying-trurl-2-on-amazon-sagemaker/blob/main/trurl-2/chatbot-app.st.py#L111-L125, where we define how the conversation flow looks like:

```python
if st.session_state.messages[-1]["role"] != "assistant":
    with st.chat_message("assistant"):
        with st.spinner("Thinking..."):
            response = talk_with_trurl2(predictor, st.session_state.messages)
            placeholder = st.empty()
            full_response = ''

            for item in response:
                full_response += item
                placeholder.markdown(full_response)

            placeholder.markdown(full_response)

    message = {"role": "assistant", "content": full_response}
    st.session_state.messages.append(message)
```

Then, an actual call to the model is consisting of two steps - building a prompt and calling the model deployed on *Amazon SageMaker*: 

```python
def talk_with_trurl2(endpoint, dict_message):
    llama2_prompt = build_llama2_prompt(dict_message)
    output = call_sagemaker_endpoint(endpoint, llama2_prompt)
    return output
```

Returned value is then wrapped as a new conversation entry, with a specific role assigned (`"assistant"`) to visualize that in the conversation flow. For those of you who does not speak *Polish*, the main prompt sets a friendly conversational tone and asks the chatbot to play a game of 20 questions, where a player specifies the category to guess, as an entrypoint to the conversation.  

## Clean-up, Costs, and Conclusion

This one is pretty easy! Assuming that you have followed all the steps inside the notebook in the *SageMaker Studio*, the only thing you need to do to clean-up is delete *AWS CloudFormation* stacks via *AWS CDK* (remember to close all the *applications* and *kernels* inside *Amazon SageMaker Studio* to be on the safe side). However, *Amazon SageMaker Studio* leaves a bit more resources hanging around related to the *Amazon Elastic File Storage (EFS)*, so first you have to delete the stack with *SageMaker Studio*, invoke clean-up script, and then delete everything else:

```shell
# Those steps should be invoked locally, from the repository root:

$ export STUDIO_STACK_NAME="Environment-SageMakerStudio"
$ export EFS_ID=$(aws cloudformation describe-stacks --stack-name "${STUDIO_STACK_NAME}" --query "Stacks[0].Outputs[?OutputKey=='SharedAmazonSageMakerStudioDomainEFS'].OutputValue" --output text)
$ export DOMAIN_ID=$(aws cloudformation describe-stacks --stack-name "${STUDIO_STACK_NAME}" --query "Stacks[0].Outputs[?OutputKey=='SharedAmazonSageMakerStudioDomainId'].OutputValue" --output text)

$ (cd infrastructure && cdk destroy "${STUDIO_STACK_NAME}")

$ ./clean-up-after-sagemaker-studio.sh "${EFS_ID}" "${DOMAIN_ID}"

$ (cd infrastructure && cdk destroy --all)
```

Last, but not least - I would like to provide a quick summary in terms of *how much does that cost*. The biggest cost factor are obviously machines that we created for and inside *Jupyter Notebook*. Those are: compute for *Amazon SageMaker Endpoint* (1x `ml.g5.2xlarge`) and compute for *kernel* that was used by *Amazon SageMaker Studio Notebook* (1x `ml.t3.medium`). Assuming, that we have set up all infrastructure in `eu-west-1`, the total cost of using for 8 hours cloud resources from this code sample will be lower than $15 ([here you can find detailed calculation](https://calculator.aws/#/estimate?id=789a6cd85ffac96e8b4321cca2a9a4d53cdb5210)). Everything else that you created with the *infrastructure as code* (via *AWS CDK*) has much lower cost, especially within the discussed time constraints.   

Outside the successful delivery (as our chatbot speaks Polish), I hope that through the exploration and attached sample code, I have managed to convince you that a requirement of using a non-English *LLM* should not be a showstopper, even for the less popular languages - like *Polish*. Thanks to the seamless integration between *Amazon SageMaker* and *Hugging Face* - which aggregates models provided by the *ML* community members and companies all around the world, you could develop an *LLM*-powered chatbot application that communicates in your native language pretty easily. 

I also encourage you to dive deeper into the [provided code example](https://github.com/build-on-aws/deploying-trurl-2-on-amazon-sagemaker) to learn more about aspects not covered in details inside this post (e.g., using *AWS CDK* to automate setting up VPC-only configuration of *Amazon SageMaker Studio* or detailed implementation of the *Streamlit* example). Also, if you have more questions, please do not hesitate to [contact me directly](https://awsmaniac.com/contact).