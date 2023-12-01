# Yi-200K-autogen-local-llm

# Install dependencies then run this code:

```python
import os

import autogen
from autogen import AssistantAgent, UserProxyAgent
from autogen.agentchat.contrib.math_user_proxy_agent import MathUserProxyAgent
from autogen.agentchat.contrib.retrieve_user_proxy_agent import RetrieveUserProxyAgent
from autogen.code_utils import extract_code

config_list = autogen.config_list_from_json(
    "OAI_CONFIG_LIST",
    file_location=".",
)
if not config_list:
    os.environ["MODEL"] = "<your model name>"
    os.environ["OPENAI_API_KEY"] = "<your openai api key>"
    os.environ["OPENAI_BASE_URL"] = "<your openai base url>" # optional

    config_list = autogen.config_list_from_models(
        model_list=[os.environ.get("MODEL", "gpt-35-turbo")],
    )

llm_config = {
    "timeout": 60,
    "cache_seed": 42,
    "config_list": config_list,
    "temperature": 0,
}

def termination_msg(x):
    _msg = str(x.get("content", "")).upper().strip().strip("\n").strip(".")
    return isinstance(x, dict) and (_msg.endswith("TERMINATE") or _msg.startswith("TERMINATE"))

def _is_termination_msg(message):
    if isinstance(message, dict):
        message = message.get("content")
        if message is None:
            return False
    cb = extract_code(message)
    contain_code = False
    for c in cb:
        # todo: support more languages
        if c[0] == "python":
            contain_code = True
            break
    return not contain_code

agents = []


agent = UserProxyAgent(
    name="User_Proxy",
    is_termination_msg=termination_msg,
    human_input_mode="TERMINATE",
    system_message="""""",
    default_auto_reply="Good, thank you. Reply `TERMINATE` to finish.",
    max_consecutive_auto_reply=5,
    code_execution_config=False,
)


agents.append(agent)


agent = AssistantAgent(
    name="Assistant_Agent",
    system_message="""You are a helpful AI assistant.
Solve tasks using your coding and language skills.
In the following cases, suggest python code (in a python coding block) or shell script (in a sh coding block) for the user to execute.
    1. When you need to collect info, use the code to output the info you need, for example, browse or search the web, download/read a file, print the content of a webpage or a file, get the current date/time, check the operating system. After sufficient info is printed and the task is ready to be solved based on your language skill, you can solve the task by yourself.
    2. When you need to perform some task with code, use the code to perform the task and output the result. Finish the task smartly.
Solve the task step by step if you need to. If a plan is not provided, explain your plan first. Be clear which step uses code, and which step uses your language skill.
When using code, you must indicate the script type in the code block. The user cannot provide any other feedback or perform any other action beyond executing the code you suggest. The user can't modify your code. So do not suggest incomplete code which requires users to modify. Don't use a code block if it's not intended to be executed by the user.
If you want the user to save the code in a file before executing it, put # filename: <filename> inside the code block as the first line. Don't include multiple code blocks in one response. Do not ask users to copy and paste the result. Instead, use 'print' function for the output when relevant. Check the execution result returned by the user.
If the result indicates there is an error, fix the error and output the code again. Suggest the full code instead of partial code or code changes. If the error can't be fixed or if the task is not solved even after the code is executed successfully, analyze the problem, revisit your assumption, collect additional info you need, and think of a different approach to try.
When you find an answer, verify the answer carefully. Include verifiable evidence in your response if possible.
Reply "TERMINATE" in the end when everything is done.
    """,
    llm_config=llm_config,
    is_termination_msg=termination_msg,
)


agents.append(agent)


init_sender = None
for agent in agents:
    if "UserProxy" in str(type(agent)):
        init_sender = agent
        break

if not init_sender:
    init_sender = agents[0]


recipient = agents[1] if agents[1] != init_sender else agents[0]

if isinstance(init_sender, (RetrieveUserProxyAgent, MathUserProxyAgent)):
    init_sender.initiate_chat(recipient, problem="I would like to make a complete newsletter about how AI improve enterprise business process performance !")
else:
    init_sender.initiate_chat(recipient, message="I would like to make a complete newsletter about how AI improve enterprise business process performance !")

```
