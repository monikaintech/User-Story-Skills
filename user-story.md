


---
name: user-story
description: "Create well-structured user stories and epics for SaaS products, then push them directly to Jira. Use this skill aggressively whenever Monika asks to write, create, draft, or generate a user story, epic, ticket, or acceptance criteria — even if she just describes a feature and says \"let's make a story for this\" or \"write this up as a ticket\". Also trigger when she says things like \"we need to track X in Jira\", \"let's groom this\", \"add this to the backlog\", or describes a new feature/screen/flow and needs it documented. The skill interviews Monika to capture intent, generates a properly formatted user story with numbered acceptance criteria (including required/optional field labels and selector value lists), confirms with her, then creates the Jira issue with priority, epic link, and dependencies wired up."
---
 
# User Story Skill
 
You help Monika write clear, well-structured user stories and push them to Jira. The output is precise, developer-ready, and follows a consistent format.
 
---
 
## Step 1: Interview
 
Ask Monika these questions **in one message** (not one by one). Fill in anything she already told you from context — don't ask for info she's clearly already given.
 
```
To write a user story, I need a few things:
 
1. **Who is the user?** (e.g. admin, end customer, sales rep, system)
2. **What do they want to do?** (the action or goal)
3. **Why?** (the business value — what problem does this solve?)
4. **What fields/inputs are involved?** List them, including whether each is required or optional, and if it's a dropdown — what are the available values?
5. **Which epic does this belong to?** (or "create a new epic" if needed)
6. **Priority?** (Critical / High / Medium / Low)
7. **Any dependencies on other stories?** (story keys, e.g. ML-42)
```
 
If the user provides partial info, fill what you can and ask only about what's missing.
 
---
 
## Step 2: Generate the User Story
 
Use this exact structure:
 
```
**[Entity: Action]**
 
**As a** [user type] user,
**I want to** [goal],
**so that** [business value].
 
---
 
**Acceptance Criteria**
 
1. Form captures
   1. [Field name] — required / optional
   2. [Field name] — optional
      - Tooltip: [tooltip text if applicable]
   3. [Field name] — required, dropdown
      - Available values: Value A | Value B | Value C
   4. [Field name] — optional, selector (select from existing [entity])
2. On successful save, [success behavior — toast / redirect / message]
3. Required fields that are empty on submit show an inline error below the field
 
 
### Naming Convention
 
Title format: **Entity: Action**
 
Examples:
- `Customer: Create`
- `Customer: Edit`
- `Invoice: Send`
- `Report: Export`
- `User: Invite`
 
Keep it short, scannable, and consistent. The entity is the primary noun (the thing being acted on), the action is the verb.
 
### Writing the Story Body
 
- **As a**: be specific about the user type (not just "user") — admin, customer, finance manager, etc.
- **I want to**: describe the concrete action, not the end state ("create a customer record" not "have customers saved")
- **So that**: focus on business value, not technical outcome ("I can track customer activity" not "the data is in the database")
 
### Writing Acceptance Criteria
 
Each criterion should be testable — a QA engineer reading it should know exactly what to check.
 
**Form fields always use a hierarchical numbered list** — one line per field, not prose sentences. This makes it instantly scannable for developers and QA.
 
Format:
 
```
1. Form captures
   1. First name — required
   2. Middle name — optional
   3. Last name — required
   4. Email — required
   5. Phone number — optional
   6. External Customer ID — optional
      - Tooltip: An ID number that identifies this person in your organization
   7. Property — optional, selector (select from existing properties)
      - Available values: [list them if known]
```
 
Rules for field lines:
- Each field gets its own numbered sub-item
- Always state **required** or **optional** inline after the field name
- If the field is a **dropdown or selector**, list all available values as a sub-bullet beneath the field
- If the field has a **tooltip**, include it as a sub-bullet: `Tooltip: [text]`
- If the field is a **selector** (picking from existing records), note it: `selector (select from existing [entity])`
 
After the fields block, add separate numbered criteria for:
- Success state: what happens after a valid save (toast, redirect, confirmation message)
- Validation/error state: what happens when required fields are missing or invalid
 
---
 
## Step 3: Confirm Before Pushing
 
Show the formatted story to Monika and ask: "Does this look right? I'll create it in Jira once you confirm."
 
Make any edits she requests, then proceed to Step 4.
 
---
 
## Step 4: Push to Jira
 
Use the Jira MCP tools to create the issue.
 
### Find the right project
 
Call `getVisibleJiraProjects` to list available projects. If there's only one, use it. If multiple, ask Monika which one.
 
### Find the epic
 
If Monika named an epic, call `searchJiraIssuesUsingJql` with:
```
issuetype = Epic AND summary ~ "[epic name]" ORDER BY created DESC
```
 
If no matching epic exists and Monika said to create one, create the epic first using `createJiraIssue` with `issuetype: Epic`.
 
### Resolve dependencies
 
If Monika listed story keys as dependencies, look them up using `getJiraIssue` to get their IDs, then use `createIssueLink` after creating the story.
 
### Create the issue
 
Call `createJiraIssue` with:
- `summary`: the title (e.g. "Customer: Create")
- `issuetype`: Story
- `description`: the full formatted story body (As a... / Acceptance Criteria / Fields table)
- `priority`: mapped to Jira priority name (Critical → Highest, High → High, Medium → Medium, Low → Low)
- `parent` or `customfield` for epic link (check `getJiraIssueTypeMetaWithFields` for the right field name in the project)
 
### Link dependencies
 
After creating, call `createIssueLink` for each dependency with link type `"Depends"` or `"blocks"` as appropriate.
 
### Confirm
 
Tell Monika the created issue key and link: "Created **ML-123** — [Customer: Create](link)"
 
---
 
## Epics
 
If Monika asks to create an **epic** instead of a story, use this structure:
 
```
**[Theme: Goal]**
e.g. "Customer Management", "Billing & Invoicing"
 
**Goal:** One sentence describing what this epic delivers and why.
 
**Scope — included:**
- [Story 1 title]
- [Story 2 title]
- ...
**Scope — excluded:**
- [What is explicitly out of scope]
**Success criteria:**
1. [Measurable outcome]
2. ...
**Priority:** [Critical / High / Medium / Low]
**Dependencies:** [Other epic keys or None]
```
 
Push epics to Jira the same way — `issuetype: Epic`.
 
---
 
## Tips for Great Stories
 
- **One story = one deployable unit of value.** If a story covers two distinct user goals, split it.
- **Avoid implementation details** in the story body (no "backend API", "database column"). Those belong in technical tasks.
- **Acceptance criteria are not a to-do list** — they describe observable behavior, not implementation steps.
- **When in doubt, be more specific.** Vague ACs ("the form works correctly") are useless. Specific ones ("clicking Save with an empty required field shows an inline error below that field") are gold.
