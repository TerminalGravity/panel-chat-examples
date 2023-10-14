# Mistral

## Chat

Demonstrates how to use the ChatInterface widget to create a chatbot using
Mistral through CTransformers.

<video controls poster="..\assets\thumbnails\mistral_chat.png" >
    <source src="..\assets\videos\mistral_chat.mp4" type="video/mp4"
    style="max-height: 400px; max-width: 600px;">
    Your browser does not support the video tag.
</video>



<details>

<summary>Source code for <a href='..\examples\mistral\mistral_chat.py' target='_blank'>mistral_chat.py</a></summary>

```python
"""
Demonstrates how to use the ChatInterface widget to create a chatbot using
Mistral through CTransformers.
"""

import panel as pn
from ctransformers import AutoConfig, AutoModelForCausalLM, Config

pn.extension(design="material")

llms = pn.state.cache["llms"] = pn.state.cache.get("llms", {})

INSTRUCTIONS = "You are a friendly chat bot willing to help out the user."


def apply_template(instructions, contents):
    text_row = f"""<s>[INST]{instructions} {contents}[/INST]"""
    return text_row


async def callback(contents: str, user: str, instance: pn.widgets.ChatInterface):
    if "mistral" not in llms:
        instance.placeholder_text = "Downloading model; please wait..."
        config = AutoConfig(
            config=Config(
                temperature=0.5, max_new_tokens=2048, context_length=2048, gpu_layers=1
            ),
        )
        llms["mistral"] = AutoModelForCausalLM.from_pretrained(
            "TheBloke/Mistral-7B-Instruct-v0.1-GGUF",
            model_file="mistral-7b-instruct-v0.1.Q4_K_M.gguf",
            config=config,
        )

    llm = llms["mistral"]
    response = llm(apply_template(INSTRUCTIONS, contents), stream=True)
    message = ""
    for token in response:
        message += token
        yield message


chat_interface = pn.widgets.ChatInterface(
    callback=callback,
    callback_user="Mistral",
    reset_on_send=True,
)
chat_interface.send(
    "Send a message to get a reply from Mistral!", user="System", respond=False
)
chat_interface.servable()
```
</details>


## With Memory

Demonstrates how to use the ChatInterface widget to create a chatbot using
Mistral thru CTransformers that includes a memory of the conversation history.

<video controls poster="..\assets\thumbnails\mistral_with_memory.png" >
    <source src="..\assets\videos\mistral_with_memory.mp4" type="video/mp4"
    style="max-height: 400px; max-width: 600px;">
    Your browser does not support the video tag.
</video>



<details>

<summary>Source code for <a href='..\examples\mistral\mistral_with_memory.py' target='_blank'>mistral_with_memory.py</a></summary>

```python
"""
Demonstrates how to use the ChatInterface widget to create a chatbot using
Mistral thru CTransformers that includes a memory of the conversation history.
"""

import panel as pn
from ctransformers import AutoConfig, AutoModelForCausalLM, Config

pn.extension(design="material")

SYSTEM_INSTRUCTIONS = "Do what the user requests."


def apply_template(history):
    history = [entry for entry in history if entry.user != "System"]
    prompt = ""
    for i, entry in enumerate(history):
        if i == 0:
            prompt += f"<s>[INST]{SYSTEM_INSTRUCTIONS} {entry.value}[/INST]"
        else:
            if entry.user == "Mistral":
                prompt += f"{entry.value}</s>"
            else:
                prompt += f"""[INST]{entry.value}[/INST]"""
    return prompt


async def callback(contents: str, user: str, instance: pn.widgets.ChatInterface):
    if "mistral" not in llms:
        instance.placeholder_text = "Downloading model; please wait..."
        config = AutoConfig(
            config=Config(
                temperature=0.5, max_new_tokens=2048, context_length=2048, gpu_layers=1
            ),
        )
        llms["mistral"] = AutoModelForCausalLM.from_pretrained(
            "TheBloke/Mistral-7B-Instruct-v0.1-GGUF",
            model_file="mistral-7b-instruct-v0.1.Q4_K_M.gguf",
            config=config,
        )

    llm = llms["mistral"]
    history = [entry for entry in instance.value]
    prompt = apply_template(history)
    response = llm(prompt, stream=True)
    message = ""
    for token in response:
        message += token
        yield message


llms = {}
chat_interface = pn.widgets.ChatInterface(
    callback=callback,
    callback_user="Mistral",
)
chat_interface.send(
    "Send a message to get a reply from Mistral!", user="System", respond=False
)
chat_interface.servable()
```
</details>