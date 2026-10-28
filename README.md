# CCA-F (CCAR-F) study guide and practice questions

Claude Certified Architect – Foundations is Anthropic's certification for solution architects who design and ship production applications with Claude. The exam code is CCAR-F, and many study sites write it as CCA-F. It tests judgment about tradeoffs across the Claude API, the Claude Agent SDK, Claude Code, and the Model Context Protocol (MCP), through scenarios such as a customer support agent, a research system with several subagents, and Claude Code running in CI. This repository holds a study guide built from Anthropic's exam guide and documentation, a cheat sheet, a six week study plan, and 15 original practice questions.

> Unofficial study notes by [Examice](https://examice.com/exams/anthropic/cca-f?utm_source=github&utm_medium=social&utm_campaign=cca-f-study-guide-and-practice-questions&utm_content=link_1). Not affiliated with or endorsed by Anthropic. The practice questions are original and not taken from the exam.

## Contents

- [Exam at a glance](#exam-at-a-glance)
- [What the exam covers](#what-the-exam-covers)
- [Study plan](#study-plan)
- [Cheat sheet](#cheat-sheet)
- [Free official resources](#free-official-resources)
- [Practice questions](#practice-questions)
- [Exam day tips](#exam-day-tips)
- [FAQ](#faq)
- [Contributing](#contributing)
- [License](#license)

## Exam at a glance

| Fact | Detail |
| --- | --- |
| Credential | Claude Certified Architect – Foundations |
| Exam code | CCAR-F |
| Questions | 60 |
| Question types | Multiple choice and multiple response; each question says how many answers to pick |
| Structure | 4 scenarios, picked at random from the 6 published in the exam guide |
| Time limit | 120 minutes (plan for about 135 minutes in the seat, with check-in and a survey) |
| Passing score | 720 on a scaled range of 100–1,000 |
| Exam fee | $125 USD, before any partner tier discount |
| Delivery | Pearson VUE, online proctored or at a test center |
| Languages | English and Japanese |
| Who can register | People who work at a Claude Partner Network organization, using a company email, aged 18 or older |
| Score report | Pass or fail, the scaled score, and percent correct by domain |
| Validity | 12 months from the date the credential is awarded |
| Renewal | A free, unproctored assessment on the Anthropic Partner Academy, taken before the credential lapses |
| Retakes | Wait 14 days after a first fail, 30 after a second, 90 after a third; at most 4 attempts in a rolling 12 months; full fee each time |

Source: [Claude Certified Architect – Foundations exam guide](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F6nizmqk8tpzpfjvt6qmmav7rh%2Fpublic%2F1783542750%2FClaude+Certified+Architect+%E2%80%93+Foundations+Exam+Guide.pdf) (version 1.0, effective July 2026), the [certification page](https://anthropic-partners.skilljar.com/claude-certified-architect-foundations-certification), the [certification FAQ](https://anthropic-partners.skilljar.com/page/faq-certifications), and the [eligibility page](https://anthropic-partners.skilljar.com/page/certification-eligibility), checked October 28, 2026.

## What the exam covers

| Domain | Weight |
| --- | --- |
| 1. Agentic Architecture & Orchestration | 27% |
| 2. Tool Design & MCP Integration | 18% |
| 3. Claude Code Configuration & Workflows | 20% |
| 4. Prompt Engineering & Structured Output | 20% |
| 5. Context Management & Reliability | 15% |

Every question sits inside a scenario. The exam guide publishes all six: a customer support agent with MCP tools, code generation with Claude Code, a research system with a coordinator and subagents, a developer productivity agent built on the Agent SDK, Claude Code in a CI/CD pipeline, and structured data extraction from documents. You get four of them. The guide also lists what is out of scope, including fine-tuning, billing and authentication, computer use, vision, streaming, rate limits, and cloud provider setup.

### Domain 1: Agentic architecture and orchestration (27%)

- Drive the agentic loop: send the tools, read `stop_reason`, run the tools Claude asked for, send back `tool_result` blocks, and repeat while the stop reason is `tool_use`. [How tool use works](https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works), [Handle tool calls](https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls)
- Build a coordinator that splits the work, picks which subagents to call for a given request, and merges what they return. Know that a subagent does not see the parent's conversation, so its prompt must carry what it needs. [Subagents in the Agent SDK](https://code.claude.com/docs/en/agent-sdk/subagents), [How Anthropic built its research system](https://www.anthropic.com/research/multi-agent-research-system)
- Define subagents with a description, a prompt, a tool list, and a model, and give the parent the tool that spawns them. [Subagents in the Agent SDK](https://code.claude.com/docs/en/agent-sdk/subagents)
- Enforce rules that must always hold in code, with hooks that block or rewrite a tool call before it runs, and keep prompts for guidance. [Agent SDK hooks](https://code.claude.com/docs/en/agent-sdk/hooks), [Claude Code best practices](https://code.claude.com/docs/en/best-practices)
- Pick a decomposition: a fixed chain of prompts for predictable steps, a dynamic plan for open investigations. [Building effective agents](https://www.anthropic.com/research/building-effective-agents)
- Resume, fork, or restart sessions, and restart with a summary when earlier tool results are out of date. [Agent SDK sessions](https://code.claude.com/docs/en/agent-sdk/sessions), [CLI reference](https://code.claude.com/docs/en/cli-reference)

### Domain 2: Tool design and MCP integration (18%)

- Write tool descriptions that say what the tool does, when to use it and when not to, what each parameter means, and what it leaves out. Rename or split tools whose descriptions overlap. [Define tools](https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools), [Writing tools for agents](https://www.anthropic.com/research/writing-tools-for-agents)
- Return errors an agent can act on: the kind of failure, whether a retry can help, and what was attempted. Tell a failed lookup apart from a lookup that found nothing. [MCP tools: error handling](https://modelcontextprotocol.io/specification/latest/server/tools)
- Give each agent only the tools its role needs, and know the `tool_choice` options. [Define tools](https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools)
- Add MCP servers to Claude Code at the right scope, keep secrets out of version control with `${VAR}` expansion, and expose catalogs of data as MCP resources. [Claude Code MCP](https://code.claude.com/docs/en/mcp), [MCP server concepts](https://modelcontextprotocol.io/docs/learn/server-concepts)
- Choose among the built-in tools: Grep for file contents, Glob for file names, Read, Edit for a targeted change, Write for a full rewrite. [Tools reference](https://code.claude.com/docs/en/tools-reference)

### Domain 3: Claude Code configuration and workflows (20%)

- Put each instruction at the right level (managed, user, project, local, or subdirectory), split long files with `@path` imports, and move topics into `.claude/rules/`. [How Claude remembers your project](https://code.claude.com/docs/en/memory)
- Build skills and commands, and use the frontmatter fields `allowed-tools`, `context: fork`, and `argument-hint`. Decide between a skill (loaded on demand) and CLAUDE.md (loaded every session). [Skills](https://code.claude.com/docs/en/skills)
- Scope a rule to file globs with `paths` so it loads only when Claude works on matching files. [Path-specific rules](https://code.claude.com/docs/en/memory#path-specific-rules)
- Use plan mode for large or architectural changes and direct execution for small, clear ones. Send verbose discovery to the Explore subagent. [Permission modes](https://code.claude.com/docs/en/permission-modes), [Subagents](https://code.claude.com/docs/en/sub-agents)
- Iterate with concrete input and output examples, tests written first, and an interview where Claude asks you questions before it builds. [Claude Code best practices](https://code.claude.com/docs/en/best-practices)
- Run Claude Code in CI with `-p`, structured output flags, project context from CLAUDE.md, and a separate instance for review. [Run Claude Code programmatically](https://code.claude.com/docs/en/headless), [GitHub Actions](https://code.claude.com/docs/en/github-actions)

### Domain 4: Prompt engineering and structured output (20%)

- Replace vague instructions ("be conservative") with explicit criteria for what to report and what to skip. [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
- Use a few targeted examples to fix output format and to show how ambiguous cases should go. [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
- Get output that matches a schema through tool use, strict tool use, or structured outputs. Make fields optional when the source may not contain them, and remember a valid schema does not make the values correct. [Structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs), [Strict tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/strict-tool-use)
- Retry with the specific validation error, and recognize when a retry cannot help because the data is not in the document. [Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)
- Send work that can wait to the Message Batches API and match results by `custom_id`. [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing)
- Review with an independent instance that did not write the code, and split large reviews into passes per file plus a pass across files. [Claude Code best practices](https://code.claude.com/docs/en/best-practices)

### Domain 5: Context management and reliability (15%)

- Keep amounts, dates, IDs, and statuses in a block that every request carries, instead of trusting a running summary. Trim tool output to the fields that matter, and place key findings where the model reads them reliably. [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows), [Effective context engineering](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents)
- Escalate on clear triggers: the customer asks for a human, the policy does not cover the case, or the agent cannot make progress. Ask for another identifier when a lookup returns several matches.
- Pass structured errors and partial results up to the coordinator, and mark which parts of a report lack sources. [How Anthropic built its research system](https://www.anthropic.com/research/multi-agent-research-system)
- Explore large codebases with subagents, scratchpad files, and `/compact` so the main context stays usable. [Explore the context window](https://code.claude.com/docs/en/context-window)
- Check accuracy per document type and per field before you cut human review, and calibrate confidence thresholds on labeled data. [Define success and build evaluations](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests)
- Keep each claim tied to its source and date through synthesis, and report conflicting figures with both sources. [Citations](https://platform.claude.com/docs/en/build-with-claude/citations)

## Study plan

Six weeks at about 6–8 hours a week. The exam guide expects 6 or more months of practical work with the API, the Agent SDK, Claude Code, and MCP, so treat this plan as a structured review, and add a week wherever you have not built the thing yourself.

| Week | Focus | Tasks | Hours |
| --- | --- | --- | --- |
| 1 | Exam guide and the agentic loop | Read the exam guide in full, including its 12 sample questions. Build a loop with two client tools that runs until `stop_reason` is no longer `tool_use`, and return one tool error with `is_error`. | 6 |
| 2 | Tool design and MCP | Write an MCP server with two similar tools and one resource. Rewrite the descriptions until Claude picks the right tool on 10 ambiguous prompts. Add it to Claude Code at project scope with a `${VAR}` token. | 7 |
| 3 | Agent SDK orchestration | Build a coordinator with two subagents that run in parallel. Pass findings to the second subagent explicitly. Add a `PreToolUse` hook that blocks one action above a threshold. Resume and fork a session. | 8 |
| 4 | Claude Code configuration | Set up a project CLAUDE.md, a `CLAUDE.local.md`, two path-scoped rules, and a skill with `context: fork`. Try plan mode on a change that spans many files. Run `claude -p` in a CI job with `--output-format json`. | 7 |
| 5 | Prompts and structured output | Build an extraction pipeline with a JSON schema, optional fields, and a retry that sends back the validation error. Run 100 documents through the Message Batches API. Write a small eval graded both in code and by a model. | 7 |
| 6 | Context, reliability, and review | Practice with long inputs (documents first, question last), escalation rules with examples, and a report that keeps sources. Do the [practice questions](practice-questions.md), then reread the domains you missed. | 6 |

## Cheat sheet

### Agentic loop and stop reasons

| `stop_reason` | What your code does |
| --- | --- |
| `tool_use` | Run every requested tool, send all `tool_result` blocks in the next user message, and call the API again |
| `end_turn` | Claude finished; show or store the answer |
| `max_tokens` | Output was cut off; raise `max_tokens` or continue |
| `stop_sequence` | A stop sequence you set was hit |
| `pause_turn` | A server tool loop paused; send the conversation back to let it continue |
| `refusal` | Claude declined; read `stop_details` and handle it in your app |
| `model_context_window_exceeded` | The response filled the context window; treat it as truncated |

In the user message with tool results, the `tool_result` blocks come first and any text comes after. Source: [Stop reasons](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons), [How tool use works](https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works), [Handle tool calls](https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls).

### Prompt guidance vs. enforcement

| Need | Use | Why |
| --- | --- | --- |
| A preference Claude may judge case by case | System prompt, CLAUDE.md, a skill | The model reads it and decides |
| A rule with no exceptions (identity check before refund, no edits to `.env`) | `PreToolUse` hook that returns `deny`, a deny rule, or a managed setting | The harness applies it every time |
| Clean up or log a result after a tool ran | `PostToolUse` hook | Runs after the tool, so it cannot stop the call |

Source: [Claude Code best practices](https://code.claude.com/docs/en/best-practices), [Agent SDK hooks](https://code.claude.com/docs/en/agent-sdk/hooks).

### What a subagent gets

| Receives | Does not receive |
| --- | --- |
| Its own system prompt and the task prompt from the parent | The parent's conversation history and tool results |
| Project CLAUDE.md, when settings sources load it | The parent's system prompt |
| Its tool list, or every tool available to subagents if none is set | Preloaded skill content, unless listed in the agent's `skills` |

Only the subagent's final message returns to the parent. Source: [Subagents in the Agent SDK](https://code.claude.com/docs/en/agent-sdk/subagents).

### Tool choice

| `tool_choice` | Behavior |
| --- | --- |
| `auto` | Claude decides whether to call a tool. Default when tools are provided |
| `any` | Claude must call one of the tools |
| `tool` | Claude must call the named tool |
| `none` | Claude cannot call tools. Default when no tools are provided |

Not every model and setting supports `any` and `tool`; the docs list where forced tool use returns an error and what to use instead. Source: [Define tools](https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools).

### MCP basics

| Item | Fact |
| --- | --- |
| Tools | Controlled by the model, which discovers and calls them |
| Resources | Data the application exposes as context, such as schemas or document lists |
| Prompts | Templates the user invokes explicitly |
| Protocol error | Unknown tool or malformed request, returned as a JSON-RPC error |
| Tool execution error | API failure, bad input, or business rule, returned in the result with `isError: true` so the model can adjust |

Source: [MCP server concepts](https://modelcontextprotocol.io/docs/learn/server-concepts), [MCP tools specification](https://modelcontextprotocol.io/specification/latest/server/tools).

| Claude Code MCP scope | Loads in | Shared with team | Stored in |
| --- | --- | --- | --- |
| Local (default) | Current project only | No | `~/.claude.json` |
| Project | Current project only | Yes, through version control | `.mcp.json` at the project root |
| User | All your projects | No | `~/.claude.json` |

When one name is defined in several scopes, local wins over project, and project over user. `.mcp.json` supports `${VAR}` and `${VAR:-default}`. Source: [Claude Code MCP](https://code.claude.com/docs/en/mcp).

### CLAUDE.md and rules

| File | Who it applies to |
| --- | --- |
| Managed policy CLAUDE.md | Everyone in the organization |
| `~/.claude/CLAUDE.md` | You, in all projects |
| `./CLAUDE.md` or `./.claude/CLAUDE.md` | The team, through version control |
| `./CLAUDE.local.md` | You, in this project only (add it to `.gitignore`) |
| `CLAUDE.md` in a subdirectory | Loads when Claude works on files in that directory |
| `.claude/rules/*.md` | Loads at launch, or only for matching files when it has `paths` frontmatter |

Files are concatenated, not overridden. `@path` imports can nest up to four hops. `/memory` lists and opens the files; `/context` shows what loaded. Source: [How Claude remembers your project](https://code.claude.com/docs/en/memory).

### Claude Code commands and flags

| Item | Use |
| --- | --- |
| `claude -p "..."` | Print mode for scripts and CI: runs, prints, exits |
| `--output-format text`, `json`, `stream-json` | Output format in print mode |
| `--json-schema '{...}'` | Validated JSON matching your schema, print mode only |
| `--resume` / `-r`, `--continue` / `-c` | Resume a named or recent session |
| `--fork-session` | With resume, start a new session ID from the old history |
| `--permission-mode plan` or `Shift+Tab` | Plan mode: Claude researches and proposes, no edits until you approve |
| `/compact`, `/clear`, `/context` | Summarize the session, start clean, see context usage |

Source: [CLI reference](https://code.claude.com/docs/en/cli-reference), [Permission modes](https://code.claude.com/docs/en/permission-modes), [Explore the context window](https://code.claude.com/docs/en/context-window).

### Message Batches API

| Fact | Value |
| --- | --- |
| Cost | 50% less than standard calls |
| Typical time | Most batches finish in under 1 hour |
| Expiry | Requests not processed within 24 hours expire, and are not billed |
| Size limit | 100,000 requests or 256 MB per batch, whichever comes first |
| Results kept | 29 days after creation |
| Matching | Results can come back in any order; match on `custom_id` |

Source: [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing).

### Prompting and evaluation

| Technique | Fact |
| --- | --- |
| Examples | 3–5 relevant, varied examples, wrapped in `<example>` tags |
| Long inputs | Put long documents at the top and the question at the end; Anthropic reports up to 30% better answers on complex inputs |
| Several documents | Wrap each in `<document>` with `<source>` and `<document_content>` |
| Hallucinations | Let Claude say it does not know; ground answers in quotes |
| Grading evals | Grading in code is fastest; human grading is the most flexible and the slowest; grading by a model scales with a clear rubric once you have tested it |

Source: [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices), [Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations), [Define success and build evaluations](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests).

### Where the exam guide and today's docs differ

| Exam guide says | Docs on October 28, 2026 say |
| --- | --- |
| Subagents are spawned with the Task tool | The tool was renamed Agent in Claude Code v2.1.63; `Task` still works as an alias ([Subagents](https://code.claude.com/docs/en/sub-agents)) |
| Custom slash commands live in `.claude/commands/` | Commands were merged into skills; files in `.claude/commands/` still work ([Skills](https://code.claude.com/docs/en/skills)) |
| `tool_choice: "any"` guarantees a tool call | Some newer models reject forced tool use; the docs suggest `auto` with strict tool use or structured outputs ([Define tools](https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools)) |

The exam guide is the authority for what is scored. Know both names.

## Free official resources

- [Certification page](https://anthropic-partners.skilljar.com/claude-certified-architect-foundations-certification) on the Anthropic Partner Academy, with registration
- [Exam guide (PDF)](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F6nizmqk8tpzpfjvt6qmmav7rh%2Fpublic%2F1783542750%2FClaude+Certified+Architect+%E2%80%93+Foundations+Exam+Guide.pdf): domains, task statements, the six scenarios, four practice exercises, and 12 sample questions with explanations
- [Certification FAQ](https://anthropic-partners.skilljar.com/page/faq-certifications) and [eligibility rules](https://anthropic-partners.skilljar.com/page/certification-eligibility)
- [Exam prep courses](https://anthropic-partners.skilljar.com/page/claude-certified-architect-foundations-prep-courses) on the Partner Academy
- Free public courses on Anthropic Academy: [Building with the Claude API](https://anthropic.skilljar.com/claude-with-the-anthropic-api), [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action), and [Introduction to Model Context Protocol](https://anthropic.skilljar.com/introduction-to-model-context-protocol)
- Documentation: [Claude API](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview), [Claude Code](https://code.claude.com/docs/en/overview), [Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview), and [MCP](https://modelcontextprotocol.io/docs/learn/server-concepts)
- Anthropic articles: [Building effective agents](https://www.anthropic.com/research/building-effective-agents), [Writing tools for agents](https://www.anthropic.com/research/writing-tools-for-agents), [Effective context engineering for AI agents](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents), and [How we built our multi-agent research system](https://www.anthropic.com/research/multi-agent-research-system)
- [Claude Partner Network](https://claude.com/partners), which you need to belong to before you can register

Anthropic retired its earlier practice exam when delivery moved to Pearson. The 12 sample questions in the exam guide show the format.

## Practice questions

[practice-questions.md](practice-questions.md) has 15 original questions in the exam's scenario style, weighted by domain, each with an explanation and a link to the docs. For more, the [Examice CCA-F practice bank](https://examice.com/exams/anthropic/cca-f?utm_source=github&utm_medium=social&utm_campaign=cca-f-study-guide-and-practice-questions&utm_content=link_2) has 130+ questions with a written explanation and a discussion under each, and the first 5 questions are free, explanations included.

## Exam day tips

- You have 120 minutes for 60 questions, so about 2 minutes each. Read each scenario's setup once and note its constraints (a resolution target, a latency limit, money at stake) before the first question.
- Each question tells you how many answers to choose. Check that number before you pick.
- When a question asks for the best first step, look for the change that fixes the root cause with the least new machinery. The sample answers in the exam guide prefer a better tool description or prompt over a new classifier or routing layer.
- When an error has financial or compliance consequences, prefer enforcement in code (a hook or a prerequisite check) over a stronger prompt.
- The exam is closed book: no notes, documentation, translation tools, or AI assistants.
- For online delivery, run Pearson's system test ahead of time and clear your desk. The name on your ID must match your registration exactly.
- The score report shows percent correct by domain, which tells you what to restudy if you need a retake.

## FAQ

### How hard is CCAR-F?

The exam guide aims it at solution architects with 6 or more months of practical work on the Claude API, Agent SDK, Claude Code, and MCP. Questions ask for the best design choice in a production scenario, not a definition, and the cut score is 720 on a scale of 100–1,000. Anthropic does not publish a pass rate. Source: [exam guide](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F6nizmqk8tpzpfjvt6qmmav7rh%2Fpublic%2F1783542750%2FClaude+Certified+Architect+%E2%80%93+Foundations+Exam+Guide.pdf).

### How long should I study?

If you already build with these tools, the six week plan above, about 40 hours, covers the blueprint. If you have not built with one of the four technologies, add time to build with it, because the questions assume you have seen how it behaves.

### Who can take the exam?

People who work at a Claude Partner Network organization and register with a company email on the partner's domain; personal emails are not accepted, and you must be 18 or older. Anthropic says [membership in the network is free](https://claude.com/resources/articles/four-role-based-claude-certifications). Source: [eligibility page](https://anthropic-partners.skilljar.com/page/certification-eligibility).

### Does the certification expire?

Yes. It is valid for 12 months. You renew on time with a free, unproctored assessment on the Partner Academy; if it lapses, you retake the full exam at the full fee. Source: [exam guide](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F6nizmqk8tpzpfjvt6qmmav7rh%2Fpublic%2F1783542750%2FClaude+Certified+Architect+%E2%80%93+Foundations+Exam+Guide.pdf), [certification FAQ](https://anthropic-partners.skilljar.com/page/faq-certifications).

### What is the retake policy?

After a fail you wait 14 days, then 30 days after a second fail and 90 days after a third. You can attempt the exam up to 4 times in a rolling 12 months, and each attempt costs the full fee (partner discounts apply). Source: [certification FAQ](https://anthropic-partners.skilljar.com/page/faq-certifications).

### Is CCA-F the same exam as CCAR-F?

Yes. Anthropic's exam guide gives the code as CCAR-F for Claude Certified Architect – Foundations; CCA-F is a common shorthand on study sites.

## Contributing

Found an error or an outdated fact? Open an issue or a pull request with a link to the official source.

## License

Text and questions © Examice, licensed under [CC BY 4.0](LICENSE).

Anthropic, Claude, and Claude Code are trademarks of Anthropic, PBC. This guide is not affiliated with or endorsed by Anthropic.
