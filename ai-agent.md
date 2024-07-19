# LLM Inference for AI Agent

## Agent System Overview
In a LLM-powered autonomous agent system, LLM functions as the agent’s brain, complemented by several key components:

### Planning
Subgoal and decomposition: The agent breaks down large tasks into smaller, manageable subgoals, enabling efficient handling of complex tasks.
Reflection and refinement: The agent can do self-criticism and self-reflection over past actions, learn from mistakes and refine them for future steps, thereby improving the quality of final results.

### Memory
Short-term memory: I would consider all the in-context learning (See Prompt Engineering) as utilizing short-term memory of the model to learn.
Long-term memory: This provides the agent with the capability to retain and recall (infinite) information over extended periods, often by leveraging an external vector store and fast retrieval.

### Tool use
The agent learns to call external APIs for extra information that is missing from the model weights (often hard to change after pre-training), including current information, code execution capability, access to proprietary information sources and more.

## Component One: Planning
A complicated task usually involves many steps. An agent needs to know what they are and plan ahead.

### Task Decomposition
Chain of thought (CoT; Wei et al. 2022) has become a standard prompting technique for enhancing model performance on complex tasks. The model is instructed to “think step by step” to utilize more test-time computation to decompose hard tasks into smaller and simpler steps. CoT transforms big tasks into multiple manageable tasks and shed lights into an interpretation of the model’s thinking process.

Tree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.

Task decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.

Another quite distinct approach, LLM+P (Liu et al. 2023), involves relying on an external classical planner to do long-horizon planning. This approach utilizes the Planning Domain Definition Language (PDDL) as an intermediate interface to describe the planning problem. In this process, LLM (1) translates the problem into “Problem PDDL”, then (2) requests a classical planner to generate a PDDL plan based on an existing “Domain PDDL”, and finally (3) translates the PDDL plan back into natural language. Essentially, the planning step is outsourced to an external tool, assuming the availability of domain-specific PDDL and a suitable planner which is common in certain robotic setups but not in many other domains.

### Self-Reflection
Self-reflection is a vital aspect that allows autonomous agents to improve iteratively by refining past action decisions and correcting previous mistakes. It plays a crucial role in real-world tasks where trial and error are inevitable.

ReAct (Yao et al. 2023) integrates reasoning and acting within LLM by extending the action space to be a combination of task-specific discrete actions and the language space. The former enables LLM to interact with the environment (e.g. use Wikipedia search API), while the latter prompting LLM to generate reasoning traces in natural language.

Reflexion (Shinn & Labash 2023) is a framework to equips agents with dynamic memory and self-reflection capabilities to improve reasoning skills. Reflexion has a standard RL setup, in which the reward model provides a simple binary reward and the action space follows the setup in ReAct where the task-specific action space is augmented with language to enable complex reasoning steps. After each action , the agent computes a heuristic and optionally may decide to reset the environment to start a new trial depending on the self-reflection results.

The heuristic function determines when the trajectory is inefficient or contains hallucination and should be stopped. Inefficient planning refers to trajectories that take too long without success. Hallucination is defined as encountering a sequence of consecutive identical actions that lead to the same observation in the environment.

Self-reflection is created by showing two-shot examples to LLM and each example is a pair of (failed trajectory, ideal reflection for guiding future changes in the plan). Then reflections are added into the agent’s working memory, up to three, to be used as context for querying LLM.

Chain of Hindsight (CoH; Liu et al. 2023) encourages the model to improve on its own outputs by explicitly presenting it with a sequence of past outputs, each annotated with feedback. Human feedback data is a collection of , where is the prompt, each is a model completion, is the human rating of, and is the corresponding human-provided hindsight feedback. Assume the feedback tuples are ranked by reward, The process is supervised fine-tuning where the data is a sequence in the form of , where. The model is finetuned to only predict where conditioned on the sequence prefix, such that the model can self-reflect to produce better output based on the feedback sequence. The model can optionally receive multiple rounds of instructions with human annotators at test time.

To avoid overfitting, CoH adds a regularization term to maximize the log-likelihood of the pre-training dataset. To avoid shortcutting and copying (because there are many common words in feedback sequences), they randomly mask 0% - 5% of past tokens during training.

## AI Agent for SRE
We have @tools, we need LLM to call differnt tools to fix different SRE issues. 

If there is a healthcheck fail, like "Ensure ip forwarding is disabled", when detect the healthcheck fail, then call check_nessus_report() to find which server fail for the healthcheck, after find the server, then use tool search_script() to check if there is existing code to fix the failed healthcheck, if yes, then call check_server() tool to verify the server is safe to apply or not, if yes, call check_conflict() tool to check if there are other tasks running for the server, if yes, quit and plan next time to apply, if no, the server is ready to apply, then call fix_healthcheck_CVE() tool to apply the fix on target server. 

### The tools we provide to LLM are:
* check_nessus_report()
* generate_task()
* search_script()
* check_server()
* check_conflict()
* fix_healthcheck_CVE()
* find_evidence()
* create_change()
* search_verification()
* verify_healthcheck_CVE()

### Details explaination for these tools:

* check_nessus_report() – Check fail healthchecks or CVE, input is nessus report, return fail healthchecks or CVE

* generate_task() – based on failed healthchecks or CVE to return solution to fix it, input is failed healthchecks, return solution tasks.

* search_script() – based on failed healthchecks or CVE to find matched scripts that can fix them, input is failed healthchecks or CVE, return matched automation scripts. 

* check_server() – based on server name to get its details information, input is server name, return the server info, like it's non-prod server, it's db2 server etc.

* check_conflict() – Check if there are conflict activities for the target server, like it's in patching maintenance reboot or not etc., input is server name, return none or event lists

* fix_healthcheck_CVE() – Run matched automation scripts to fix healthchecks or CVE input is matched scrips, tag and target server name

* find_evidence() – based on failed healthchecks or CVE and hostname to search similar services requests or change requests in ticket system, and return the most similarity one. 

* create_change() – create change request, input is failed healthchecks or CVE and hostname and evidence find by find_evidence()

* search_verification() – based on failed healthchecks or CVE to search and match verification scripts 

* verify_healthcheck_CVE() – execute verification scripts returned by search_verification() tool to verify the fixing. 

### Generally we can follow below procedure to handle a failed healthchecks or CVE:

General tool call procedure for non-prod servers:

* check_nessus_report() -> generate_task() -> search_script() -> check_server() -> check_conflict() -> fix_healthcheck_CVE() -> search_verification() -> verify_healthcheck_CVE()

General tool call procedure for prod servers:

* check_nessus_report() -> generate_task() -> search_script() -> check_server() -> check_conflict() -> find_evidence() -> create_change() -> fix_healthcheck_CVE() -> search_verification() -> verify_healthcheck_CVE()

But based on different status or situation, we need to take different tools and procedure to fix issues. We need LLM to planning and take correct actions based on different scenarios.

### Example:
Detect one healthcheck failed on server db2server1, the failed healthcheck is "Ensure RPC is not enabled", then LLM should think and choose check_nessus_report() tool to get more details info about the failed healthcheck, to get the server name, the solution, if LLM knows the fix scripts exists, then LLM can directly call check_server() tool and check_conflict() tool to decide if it's ready to apply fix. If LLM don't know if the fixing scripts exists or not, it should call search_script() tool to check. If scripts is ready and server is reasdy, and server is non-prod server, no need to create change request, then LLM should directly call fix_healthcheck_CVE() tool to apply on target server. If script and server is ready, but it's prod server, then LLM should call find_evidence() tool and create_change() tool to create change. After applied fixing, LLM should call search_verificatioin() tool and verify_healthcheck_CVE() tool to verify the fixing. 
