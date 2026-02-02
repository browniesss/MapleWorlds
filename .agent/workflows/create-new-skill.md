---
description: Create a new skill for the agent. Use this when you need to add a new capability or specialized knowledge to the agent.
---

1. Ask the user for a description of the skill they want to create and any reference documents or tech stacks involved.
2. Perform a web search to gather relevant data and best practices based on the user's description.
3. Generate the skill content using the gathered data and the following template. Ensure the content is optimized for the Google Antigravity environment.

    ```markdown
    ---
    name: <SKILL_NAME>
    description: <Short description of what the skill does and when to use it>
    ---

    # <Skill Name> (My Skill)

    <Detailed instructions for the agent go here>

    ## 언제 이 스킬을 사용하는가 (When to use this skill)

    - <Condition 1>
    - <Condition 2>

    ## 사용 방법 (How to use)

    <Step-by-step guide, conventions, and patterns for the agent>
    ```

4. Refactor the generated skill content to strictly adhere to the following Best Practices:
    - **Keep it Focused**: Ensure the skill does ONE thing well. Split complex tasks into separate skills.
    - **Clear Descriptions**: The `description` field determines when the agent uses this skill. Be specific about WHAT it does and WHEN to use it.
    - **Treat Scripts as Black Boxes**: If the skill involves potential scripts, instruct the agent to run them with `--help` rather than reading the source code.
    - **Include Decision Trees**: For complex skills, add a section guiding the agent to the right approach based on the situation.

5. Create the new skill file at `.agent/skills/<skill-name>/SKILL.md`.
   - You may need to create the directory `.agent/skills/<skill-name>` first.
   - Replace `<skill-name>` with a relevant, kebab-case name for the skill.
