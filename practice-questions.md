# CCA-F (CCAR-F) practice questions

Fifteen original practice questions for Claude Certified Architect – Foundations (CCAR-F, often written CCA-F), spread across the five domains by their weight in the [exam guide](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F6nizmqk8tpzpfjvt6qmmav7rh%2Fpublic%2F1783542750%2FClaude+Certified+Architect+%E2%80%93+Foundations+Exam+Guide.pdf): 4 on agentic architecture, 3 on tool design and MCP, 3 on Claude Code, 3 on prompt engineering and structured output, and 2 on context management and reliability. Each answer explains why the right option is right, why the others are wrong, and links the Anthropic or MCP documentation behind it.

> Unofficial study notes by Examice. Not affiliated with or endorsed by Anthropic. The practice questions are original and not taken from the exam.

### Question 1

*Domain 1: Agentic architecture and orchestration*

You are building a customer support agent on the Claude API with three client tools. Your loop ends as soon as a response contains a text block. In the logs, Claude often writes a short line such as "Let me look up that order" next to a `tool_use` block, and the loop exits before the tool runs. What should decide whether the loop continues?

- A. Instruct Claude to write "FINAL ANSWER" when it is done, and continue until that phrase appears.
- B. Run a fixed 10 iterations, then return the last text block.
- C. Continue while `stop_reason` is `"tool_use"`, and stop to handle any other stop reason.
- D. Continue while the response contains no text blocks.

<details><summary>Show answer</summary>

**C.** The documented loop runs the requested tools and calls the API again while `stop_reason` is `"tool_use"`. Any other value (`end_turn`, `max_tokens`, `stop_sequence`, `refusal`, and so on) means Claude finished or stopped for a reason your code must handle.

- A depends on Claude writing a phrase reliably, which is the kind of text parsing the stop reason exists to replace.
- B cuts off long tasks and wastes calls on short ones; a cap is a safety limit, not the way to detect completion.
- D is the same bug as the current loop in reverse: a response can carry text and `tool_use` blocks together.

Docs: [How tool use works](https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works), [Stop reasons](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons)

</details>

### Question 2

*Domain 1: Agentic architecture and orchestration*

A research system on the Claude Agent SDK has a coordinator, a search subagent, and a synthesis subagent. The search subagent returns 14 sources to the coordinator. The coordinator then calls the synthesis subagent with the prompt "Write the report from the sources we found." The report is generic and cites none of the 14 sources. What is the fix?

- A. Give the synthesis subagent the search tools so it can find the sources again.
- B. Put the findings, with each claim's source URL and date, directly in the prompt the coordinator sends to the synthesis subagent.
- C. Set the synthesis subagent to the same model as the coordinator so they share memory.
- D. Add "use the conversation history" to the synthesis subagent's description.

<details><summary>Show answer</summary>

**B.** A standard subagent starts in a fresh conversation. It receives its own system prompt and the task prompt from the parent, not the parent's conversation history or tool results. Anything it needs has to be in that prompt.

- A repeats work, adds latency, and gives a synthesis role tools outside its job, while the real gap (no context passed) remains.
- C does nothing for context: the model choice does not share history between agents.
- D cannot work, because the description tells the parent when to use the subagent; it does not give the subagent access to history it never received.

Docs: [Subagents in the Agent SDK](https://code.claude.com/docs/en/agent-sdk/subagents)

</details>

### Question 3

*Domain 1: Agentic architecture and orchestration*

Your support agent can call `issue_refund`. Policy says any refund over $500 needs a human. The system prompt states the rule clearly, yet logs show 3 of the last 1,000 refunds over $500 went through without review. What is the most reliable fix?

- A. A `PostToolUse` hook on `issue_refund` that alerts a manager when the amount was over $500.
- B. Repeat the rule in capital letters at the start and end of the system prompt.
- C. Add five examples to the prompt that show the agent escalating large refunds.
- D. A `PreToolUse` hook on `issue_refund` that denies the call when the amount is over $500 and returns a reason telling the agent to escalate.

<details><summary>Show answer</summary>

**D.** A rule with no exceptions belongs in configuration the harness applies every time, not in text the model may weigh against other context. A `PreToolUse` hook runs before the call executes and can return a `deny` decision with a reason the agent sees, so it can route the case to a human.

- A runs after the refund already went out. It detects the violation instead of preventing it.
- B and C are still instructions the model interprets. They may lower the rate, but they cannot guarantee zero when money is at stake.

Docs: [Agent SDK hooks](https://code.claude.com/docs/en/agent-sdk/hooks), [Claude Code best practices: enforce the rules that can't have exceptions](https://code.claude.com/docs/en/best-practices)

</details>

### Question 4

*Domain 1: Agentic architecture and orchestration*

Every invoice your team processes goes through the same three steps in the same order: extract the fields, convert the amounts to USD, and write a short summary for accounting. The order never changes. A colleague proposes one autonomous agent that holds all three tools and decides what to do for each invoice. Which design fits best?

- A. A prompt chain: three calls in a fixed order, with a check in code between steps.
- B. An orchestrator that plans the work per invoice and delegates to three worker subagents.
- C. One agent loop with all three tools and a limit of 10 iterations.
- D. An evaluator loop where a second model grades each step and the first retries until it scores 10 out of 10.

<details><summary>Show answer</summary>

**A.** Anthropic separates workflows, where code defines the path, from agents, where the model directs its own steps. Prompt chaining suits a task that breaks cleanly into fixed subtasks, and it trades a little latency for more accurate, easier steps. Checks between steps catch a bad extraction before it reaches the summary.

- B adds planning and delegation for a task whose plan never changes.
- C lets the model choose an order the business has already fixed, which adds variance with no benefit.
- D adds an extra model call per step and an arbitrary score target; evaluator loops pay off when quality improves with feedback, not for a deterministic sequence.

Docs: [Building effective agents](https://www.anthropic.com/research/building-effective-agents)

</details>

### Question 5

*Domain 2: Tool design and MCP integration*

Your MCP server exposes `search_tickets` ("Searches tickets") and `search_kb` ("Searches articles"). When customers ask about the status of their own ticket, the agent often calls `search_kb`. What is the most effective first change?

- A. Put a keyword router in front of the model that picks the tool before each turn.
- B. Expand both descriptions: what each one searches, when to use it and when not to, what each parameter means, and what it does not return.
- C. Merge both into one `search` tool that queries both systems and returns every match.
- D. Add `input_examples` to both tools and leave the descriptions as they are.

<details><summary>Show answer</summary>

**B.** Anthropic's tool guidance calls detailed descriptions by far the most important factor in tool performance, and suggests at least 3–4 sentences per tool covering purpose, when to use it, parameters, and limits. Two descriptions of two words each give the model almost nothing to tell the tools apart.

- A replaces the model's judgment with brittle keyword rules and adds a component to maintain, before the cheap fix has been tried.
- C returns results from both systems on every call, which bloats the context; the docs ask for responses that carry only what the next step needs.
- D helps with complex inputs or inputs whose format matters, but the docs say to prioritize descriptions; examples do not explain when each tool applies.

Docs: [Define tools: best practices](https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools)

</details>

### Question 6

*Domain 2: Tool design and MCP integration*

An MCP tool `create_booking` receives a departure date in the past. Today the server throws an exception, the client reports a JSON-RPC error, and the agent gives up. What should the server return so the agent can correct the request itself?

- A. A JSON-RPC error with code `-32602`.
- B. An empty successful result, so the conversation keeps going.
- C. A tool result with `isError: true` and a message saying the date must be in the future, including today's date.
- D. A successful result whose text says "Operation failed".

<details><summary>Show answer</summary>

**C.** The MCP specification splits errors in two. Protocol errors (unknown tool, malformed request) go back as JSON-RPC errors. Tool execution errors, including invalid input and business rule failures, go back inside the tool result with `isError: true`, and clients should pass them to the model so it can fix the input and retry. A specific message tells the agent what to change.

- A is the protocol error path, meant for problems the model is unlikely to fix, so recovery becomes less likely.
- B reports a failure as a success with no data; the agent cannot tell "no booking possible" from "booking made".
- D is marked as a success and says nothing about the cause, so the agent has nothing to act on.

Docs: [MCP tools specification: error handling](https://modelcontextprotocol.io/specification/latest/server/tools)

</details>

### Question 7

*Domain 2: Tool design and MCP integration*

Your team wants every developer who clones the repository to get the same MCP server for the internal issue tracker in Claude Code. The server needs each developer's own API token. What should you do?

- A. Ask each developer to run `claude mcp add` with the default scope.
- B. Add the server at user scope in your `~/.claude.json` with your token, and send the file to the team.
- C. Commit `.mcp.json` with a shared team token in plain text, since the repository is private.
- D. Define the server in `.mcp.json` at the project root with the token written as `${TRACKER_TOKEN}`, commit the file, and have each developer set the variable.

<details><summary>Show answer</summary>

**D.** Project scope stores servers in `.mcp.json` at the project root, which is checked into version control so everyone gets the same servers. Claude Code expands `${VAR}` and `${VAR:-default}` in that file, so the shared configuration carries no secret.

- A uses local scope, the default, which is private to each developer and stored in their own `~/.claude.json`. Nothing is shared, and the setup drifts.
- B shares your personal token, and user scope is not distributed through the repository.
- C puts a secret in version control, where every clone and every past commit keeps it.

Docs: [Claude Code MCP: installation scopes](https://code.claude.com/docs/en/mcp)

</details>

### Question 8

*Domain 3: Claude Code configuration and workflows*

You want Claude Code to use your personal staging URL and test account when you work in one repository. Your teammates should not see these values, and they should not apply in your other projects. Where do they go?

- A. `~/.claude/CLAUDE.md`.
- B. `CLAUDE.local.md` at the project root, added to `.gitignore`.
- C. `./CLAUDE.md` at the project root.
- D. `.claude/rules/staging.md` in the project.

<details><summary>Show answer</summary>

**B.** `CLAUDE.local.md` holds personal instructions for one project. It loads next to the project's CLAUDE.md, and you keep it out of version control with `.gitignore`.

- A applies to you in every project, which breaks the second requirement.
- C is the team's shared file, checked in for everyone.
- D lives in the project's `.claude/` directory, which teams commit, so the rule would be shared along with the other project rules.

Docs: [How Claude remembers your project](https://code.claude.com/docs/en/memory)

</details>

### Question 9

*Domain 3: Claude Code configuration and workflows*

Your root CLAUDE.md contains a 250 line release procedure that the team uses twice a month. It takes up context in every session, including unrelated bug fixes. You want the procedure to run only when someone types `/release`. What should you do?

- A. Move it to `.claude/skills/release/SKILL.md` with `disable-model-invocation: true` in the frontmatter.
- B. Move it to `.claude/rules/release.md` with no `paths` frontmatter.
- C. Keep it in CLAUDE.md under a heading that says "Read only during releases".
- D. Move it to `~/.claude/CLAUDE.md`.

<details><summary>Show answer</summary>

**A.** A skill loads its full content only when it is invoked, so the procedure stops costing context in unrelated sessions. `disable-model-invocation: true` stops Claude from loading it on its own, so it runs only when someone types `/release`. A project skill is committed with the repository, so the whole team has it.

- B loads at launch: rules without `paths` frontmatter load every session with the same priority as `.claude/CLAUDE.md`.
- C still loads all 250 lines every session; a heading is guidance, not a loading rule.
- D still loads every session, and only for you, so the team loses the procedure.

Docs: [Skills: frontmatter reference](https://code.claude.com/docs/en/skills), [How Claude remembers your project](https://code.claude.com/docs/en/memory), [Claude Code best practices: put team procedures in skills](https://code.claude.com/docs/en/best-practices)

</details>

### Question 10

*Domain 3: Claude Code configuration and workflows*

A CI job runs Claude Code on each pull request, and a script posts every finding as an inline review comment. The script parses Claude's prose and breaks whenever the wording changes. How do you give the script input it can rely on?

- A. Run `claude` interactively in the CI container and pipe `yes` into it.
- B. Run `claude -p` with `--output-format text` and parse the result with regular expressions.
- C. Run `claude -p` with `--output-format json` and a `--json-schema` that defines each finding's file, line, severity, and message, then read the `structured_output` field.
- D. Add "always answer in JSON" to CLAUDE.md and parse the text output.

<details><summary>Show answer</summary>

**C.** Print mode (`-p`) runs without prompts and exits, which CI needs. With `--output-format json` and `--json-schema`, Claude Code returns output validated against your schema in the `structured_output` field, next to metadata such as the session ID. The script should still check that the field is present, because a run that ends without valid output leaves it out.

- A keeps an interactive session waiting for input; piping text into it is a workaround, not a supported mode for automation.
- B is the current approach with a different flag: prose parsed by patterns breaks when wording changes.
- D is an instruction the model reads, not a validated format, so a malformed or chatty response can still reach the script.

Docs: [Run Claude Code programmatically](https://code.claude.com/docs/en/headless), [CLI reference](https://code.claude.com/docs/en/cli-reference)

</details>

### Question 11

*Domain 4: Prompt engineering and structured output*

A classifier sorts support emails into `billing`, `bug`, `feature_request`, or `other`. An enum in the schema keeps every label valid, and the instructions define each category. Still, ambiguous emails, such as a refund request caused by a bug, get different labels on different runs. What change most improves consistency?

- A. Add "Be very careful and consistent" to the instructions.
- B. Ask Claude to rate its confidence from 1 to 10 next to each label.
- C. Add two more categories so fewer emails are ambiguous.
- D. Add 3–5 varied examples in `<example>` tags, including ambiguous cases with the correct label and a short reason.

<details><summary>Show answer</summary>

**D.** Anthropic calls examples one of the most reliable ways to steer output, and recommends 3–5 that are relevant, diverse, and wrapped in `<example>` tags. Examples of the hard cases, with the reason for the label, show the decision rule the prose leaves open.

- A is a general exhortation; it gives no rule for the cases that vary.
- B adds a number without changing how the label is chosen, and a model's own confidence rating is a weak signal of correctness.
- C moves the boundary instead of defining it: new categories create new ambiguous edges, and downstream systems must change.

Docs: [Prompting best practices: use examples effectively](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices), [Increase output consistency](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/increase-consistency)

</details>

### Question 12

*Domain 4: Prompt engineering and structured output*

Your invoice extraction schema marks `purchase_order_number` as a required string. About 30% of invoices have no purchase order number, and in those cases the output contains plausible numbers that appear nowhere in the document. What is the fix?

- A. Retry each extraction until it returns a purchase order number.
- B. Make the field optional or nullable, and tell Claude to return null when the invoice has no purchase order number.
- C. Turn on strict tool use so the value must match the schema.
- D. Add examples that show purchase order numbers in the correct format.

<details><summary>Show answer</summary>

**B.** A required field tells the model a value must exist, so when the document has none it fills the gap. Making the field optional or nullable, and saying that null is the right answer when the value is absent, gives Claude permission not to know, which Anthropic lists as a direct way to cut hallucinations.

- A asks repeatedly for data that is not in the document; retries help with format errors, not missing information.
- C guarantees the value fits the schema, which invented strings already do; schema validity says nothing about truth.
- D teaches the format of a value the model should not be producing, which can make the invented numbers more convincing.

Docs: [Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations), [Structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)

</details>

### Question 13

*Domain 4: Prompt engineering and structured output*

You are moving a weekly job that summarizes 30,000 support tickets from standard Messages API calls to the Message Batches API. Which two statements should shape your design? Choose two.

- A. Every batch is guaranteed to finish within 1 hour.
- B. Results can come back in any order, so give each request a unique `custom_id` and match results on it.
- C. Requests that expire after 24 hours are billed at the batch rate.
- D. Batch requests cost about 50% less than the same standard requests.

<details><summary>Show answer</summary>

**B and D.** The Message Batches API cuts cost by 50% for work that can wait, and its results may not match the order you submitted them in, so the docs tell you to use `custom_id` to pair each result with its request.

- A overstates the docs: most batches finish within 1 hour, but a batch can take up to 24 hours, and requests still waiting at that point expire.
- C is false: expired, canceled, and errored requests are not billed. Your design should still resubmit them, by `custom_id`.

Docs: [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing)

</details>

### Question 14

*Domain 5: Context management and reliability*

Your contract review prompt starts with the instructions and the question, followed by 9 contracts (about 120,000 tokens) pasted as plain text one after another. Answers miss clauses and mix up which contract says what. What is the best restructure?

- A. Keep the order, and repeat the question after each contract.
- B. Summarize each contract in one paragraph first, and send only the summaries.
- C. Put the contracts first, each in a `<document>` tag with `<source>` and `<document_content>`, and put the instructions and question at the end.
- D. Move all 9 contracts into the system prompt as one block of text.

<details><summary>Show answer</summary>

**C.** For long inputs, Anthropic recommends putting the documents at the top and the query at the end, which improved response quality by up to 30% in its tests, especially with several documents. Wrapping each document with its source in XML tags keeps the model clear about which contract a clause came from.

- A multiplies the question without fixing the order or separating the documents.
- B throws away the detail in each clause that the review needs; summaries lose exactly the facts that get missed.
- D still gives one undivided block with no source labels, so the mix-ups remain.

Docs: [Prompting best practices: long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

</details>

### Question 15

*Domain 5: Context management and reliability*

Before you reduce human review of a summarization feature, you need to score 2,000 summaries for faithfulness to their source on every release. Exact match does not apply, and having people grade all 2,000 takes a week. Which grading approach fits?

- A. Grading by a model, with a detailed rubric and a fixed output such as pass or fail, checked first against a sample that people graded.
- B. String match on key phrases from each source document.
- C. People grading all 2,000 summaries on every release.
- D. Ask the model that wrote each summary to rate its own work.

<details><summary>Show answer</summary>

**A.** Anthropic's eval guidance ranks grading in code as fastest but limited to checks that follow fixed rules, human grading as the most flexible but slow and expensive, and grading by a model as fast, flexible, and scalable for complex judgments once you have tested its reliability. A detailed rubric and a fixed scale make the grades consistent, and comparing against human grades on a sample is that reliability test.

- B cannot judge faithfulness: a summary can repeat key phrases and still misstate the source.
- C is the bottleneck you are trying to remove, and the guidance says to avoid it where possible.
- D asks the writer to grade its own work, the weakest check; Anthropic's Claude Code guidance favors review by an instance that did not write the output.

Docs: [Define success and build evaluations](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests), [Claude Code best practices: give Claude a check to run](https://code.claude.com/docs/en/best-practices)

</details>

## More practice

The [Examice CCA-F practice bank](https://examice.com/exams/anthropic/cca-f?utm_source=github&utm_medium=social&utm_campaign=cca-f-study-guide-and-practice-questions&utm_content=link_3) has 130+ questions in this style, each with a written explanation and a discussion. The first 5 questions are free, explanations included.
