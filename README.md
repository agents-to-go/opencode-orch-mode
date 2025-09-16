# Agent Orchestration (ORCH) Workflow Documentation

## Overview

This workflow provides a structured approach to issue resolution through collaborative planning and execution. It's specifically designed to **make smaller and less capable models follow plans precisely** without going wild or making random changes. The system ensures nothing gets left behind and everything is implemented exactly as planned through continuous review and quality assurance loops.

First run the install.sh to put the slash command in proper global opencode config location.

> **Note:** For the sub-agents, you should check out [this repo](https://github.com/aptdnfapt/opencode-parallel-agents) and get the md files for your sub-agents in the proper location.

## Phase 1: Manual Issue Creation

### Step 1: Create Issue File
1. Create a new file in the `./issue/` directory
2. Name your file descriptively (e.g., `fix-auth-bug.md`, `add-user-profile.md`)
3. Include in your issue file:
   - Problem description
   - Error logs (if applicable)
   - Your suggestions or ideas
   - Any specific requirements
   - Expected outcomes


## Phase 2: Interactive Planning with Main Agent

### Step 2: Start Planning Discussion
1. Open an opencode session with your main agent
2. Tell the agent about your issue file:
   ```
   I created an issue file at ./issue/fix-auth-bug.md, please look at it and help me come up with a plan
   ```

### Step 3: Back-and-Forth Planning
1. **Main agent analyzes** your issue file and codebase
2. **Main agent proposes** an initial plan
3. **You review and provide feedback**:
   - Explain your thought process
   - Clarify requirements
- Suggest alternative approaches
- User asks "did you get my idea? Tell me what you understood"
4. **Main agent refines** the plan based on your input
5. **Continue this loop** until you're satisfied with the plan


### Step 5: Create Plan File
When satisfied with the plan:
1. Tell the main agent: (future goal is to turn this into a / command)
   ```
   Please create a plan file in ./plan/ directory based on our discussion. This should contain 2 sections. 1st section (no fluff, just pure English explanation, no code): what's wrong and how you plan to fix it (use arrows to explain better in this section). 2nd section is more detailed with which file would be edited and why, what will be the new project structure, and so on. 2nd section is detailed plan.
   ```
2. Main agent creates a plan file (e.g., `./plan/fix-auth-bug.md`)

## Phase 3: Clear Context

### Step 6: Start New Chat
1. Use the `/new` command to clear the context window
2. This frees up memory/context window and starts fresh for the execution phase.

## Phase 4: Automated Execution

### Step 7: Run ORCH Workflow Command
In the new chat context, run:
```
/orch "check the issue at ./issue/fix-auth-bug.md and the plan at ./plan/fix-auth-bug.md and start the ORCH workflow using @grok as agent_1 and @glm as agent_2"
```

### What Happens During Execution

**The main agent will:**

1. **Setup Agent-Loop Directory**
   - Create `./agent-loop/[plan-context]/` directory based on plan file name
   - Initialize sequential file numbering system

2. **Phase 1 - Execute Plan with @agent_1**
   - Spawn @agent_1 (your first specified agent)
   - @agent_1 reads the issue file, plan file, and any previous agent summaries
   - @agent_1 implements the plan exactly as written
   - @agent_1 creates `./agent-loop/[plan-context]/01-code.md` with implementation summary
   - @agent_1 does NOT commit changes

3. **Phase 2 - Review Implementation with @agent_2**
   - Spawn @agent_2 (your second specified agent)
   - @agent_2 reads the issue file, plan file, and ALL previous agent summaries
   - @agent_2 checks git diff and compares implementation against plan requirements
   - @agent_2 calculates a compliance score (0-100%)
   - @agent_2 creates `./agent-loop/[plan-context]/02-review.md` with detailed review

4. **Phase 3 - Loop Until 90%+ Compliance (if needed)**
   - If score < 90%: Main agent spawns new @agent_1 to fix issues
   - Each agent reads ALL previous summaries before starting work
   - @agent_1 creates `./agent-loop/[plan-context]/03-code.md`, `05-code.md`, etc.
   - @agent_2 creates `./agent-loop/[plan-context]/04-review.md`, `06-review.md`, etc.
   - Loop continues until 90%+ compliance is achieved

5. **Phase 4 - Final Completion**
   - Main agent provides final summary:
     - Number of iterations required
     - Final compliance score
     - Summary of what was implemented
     - Any remaining minor deviations
   - Workflow completes without committing changes

### Agent-Loop Directory Structure

The system creates a persistent conversation history:

```
agent-loop/
└── [plan-context-directory]/          # Named based on plan file
    ├── 01-code.md                     # First implementation
    ├── 02-review.md                   # First review
    ├── 03-code.md                     # Fixes implementation
    ├── 04-review.md                   # Fixes review
    └── ...                            # Continues as needed
```

**Benefits:**
- **Persistent context**: Each agent knows exactly what previous agents did
- **Conversation history**: Complete record of the development process
- **Debugging friendly**: Easy to trace issues and understand decisions
- **Quality assurance**: Agents build on previous work rather than starting fresh
- **Model control**: Forces smaller models to follow plans precisely through continuous oversight
- **Complete implementation**: 90% compliance threshold ensures nothing gets missed

## Key Features

### Quality Assurance
- **90% Compliance Threshold**: Ensures high-quality implementation
- **Independent Review**: Separate agent for unbiased evaluation
- **Iterative Improvement**: Loop until quality standards are met

### Agent Roles
- **Main Agent**: Coordinates workflow, never codes
- **@agent_1**: Implementation specialist, executes plans
- **@agent_2**: Quality reviewer, evaluates compliance

### Safety Features
- **No Automatic Commits**: You decide when to commit
- **Plan-Bound Execution**: Agents follow plans exactly
- **Clear Documentation**: All steps tracked and reviewed
- **Agent Memory**: Persistent conversation history prevents context loss
- **Sequential Tracking**: Each agent action is numbered and timestamped
- **Model Constraint**: Forces smaller/dumber models to stick to plans through continuous review
- **Quality Assurance**: 90% compliance threshold ensures nothing gets missed

## Best Practices

### For Issue Files
- Be specific about problems and requirements
- Include relevant error logs and context
- Structure your thoughts clearly

### For Planning Phase
- Take time to explain your thought process
- Ensure the main agent understands your vision, so continuously ask about its point of view. 
- Review the plan carefully before approval

### For Execution Phase
- Choose appropriate agents for your specific needs (fast model for execution like grok and good model with decent speed for review like glm4.5)
- Be patient with the quality loop process
- Review the final summary before committing changes
- Monitor the agent-loop directory to understand the conversation between agents
- Each agent reads previous work, reducing repetitive explanations and improving quality
- **Git Management**: Add `agent-loop/` to `.git/info/exclude` if you don't want to push these files to GitHub

## Example Commands

**Start planning:**
```
I created an issue file at ./issue/add-feature-x.md, please look at it and help me come up with a plan. Tell me what you understood. Don't present code. Tell me what's wrong and what can be the plan to fix this. Explain in plain English.
```

**Create plan file:**
```
   Please create a plan file in ./plan/ directory based on our discussion. This should contain 2 sections. 1st section (no fluff, just pure English explanation, no code): what's wrong and how you plan to fix it (use arrows to explain better in this section). 2nd section is more detailed with which file would be edited and why, what will be the new project structure, and so on. 2nd section is detailed plan. 

```

**Execute workflow:**
```
/orch "check the issue at ./issue/add-feature-x.md and the plan at ./plan/add-feature-x.md and start the ORCH workflow using @grok as agent_1 and @glm as agent_2"
```

**Exclude agent-loop from Git (optional):**
```
echo "agent-loop/" >> .git/info/exclude
```

## Troubleshooting

**If agents get stuck:**
- Git hard reset 
- Use `/new` to clear context and restart
- Ensure your issue and plan files are clear and specific
- Check the agent-loop directory to understand where agents lost context
- Delete the agent-loop directory to start with a clean slate

**If quality loop takes too long:**
- Make sure to use fast model like grok or qwen3 coder plus (check my free qwen proxy repo). Don't use something like gpt 5, but you can of course if you want to. If a task is very hard, check the parallel agent repo of mine on my GitHub repos for opencode. 
- Consider if your requirements are too complex for one iteration
- Break down large issues into smaller, manageable parts or even 2-3 issue files and make plan files one by one. 

This ORCH workflow ensures systematic issue resolution with quality assurance while maintaining human oversight throughout the process. It's particularly effective for constraining smaller models to follow plans precisely and ensuring complete implementation without random deviations.
