# Auto Browser: WebVoyager-mini results

Results, per-task evidence and grading setup for Auto Browser, a browser agent that runs as a Chrome extension, on wv-mini: 105 tasks from the WebVoyager benchmark, seven from each of its 15 sites. The write-up, with the cost analysis and limitations, is on the Auto Browser blog: [82.8% on WebVoyager-mini at 2.6 cents a task](https://autobrowser.dev/blog/webvoyager-mini-results).

| System | Passed / graded | Pass rate | Model cost, 105 tasks | Cost per task |
|---|---|---|---|---|
| Auto Browser + DeepSeek V4.1 Flash | 77 / 93 | 82.8% | $2.75 | $0.026 |
| Auto Browser + Claude Sonnet 5.5 | 73 / 92 | 79.3% | $91.55 | $0.87 |

Tasks stopped by a site's bot check are not graded: 12 in the DeepSeek run, 13 in the Sonnet run. Cost covers the agent's model calls and leaves out the judge. All results are self-reported.

## Method

- **Tasks.** The first seven tasks from each WebVoyager site, in WebVoyager's order, with the task fixes [Alumnium published](https://github.com/alumnium-hq/WebVoyager/tree/alumnium/data). Four tasks were left out and replaced by the next task from the same site, and five were reworded. `tasks/wv-mini-patches.json` gives the reason for each change.
- **Grading.** GPT-5.5 (`gpt-5.5-2026-04-23`, medium reasoning effort) with our own prompt, in `judge/`. The judge reads the task, the run date, the final answer, the agent's actions with the page text it read, the final page, and a final screenshot when one was captured. A task passes only when every condition is met and the answer comes from the website.
- **Bot checks.** A task that hits a bot check is stopped and not graded. The agent is never allowed past one.
- **Attempts.** One attempt per task, with a ten-minute limit. Runs were made from Kuala Lumpur, Malaysia, so sites served their Malaysian versions.
- **Cost.** What OpenRouter recorded for the agent's model calls. The judge is not included.

## Contents

| Path | What it holds |
|---|---|
| `results/<model>.json` | Run summary and one row per task |
| `results/per-task.csv` | Each task's status and cost, both models side by side |
| `evidence/<model>/<task>.json` | One record per task: the task, the final answer, the judge's verdict and reason, the final URL and page title, cost, time, and every action the agent took |
| `evidence/<model>/screenshots/<task>.jpg` | The final-page screenshot the judge saw, when one was captured |
| `tasks/wv-mini/` | The 105 task files |
| `tasks/wv-mini-patches.json` | Every change to the upstream tasks, with its reason |
| `judge/prompt-wv-2.2.txt` | The judge's prompt template |
| `judge/pin.json` | The judge's model, reasoning effort and input limits |

`<model>` is `deepseek-v4.1-flash` or `claude-sonnet-5.5`.

## Reading a task record

`status` is `pass` or `fail` for a graded task, and `blocked` for a task stopped by a bot check. A blocked task has no verdict, and `not_graded_because` names the check.

`actions` lists the agent's tool calls in order, each as `{ tool, args, ok, ms }`. Tool results and page text are left out. The judge saw them, so a verdict can rest on text that is not in the record. Session parameters were removed from URLs.

## Runs

| | DeepSeek V4.1 Flash | Claude Sonnet 5.5 |
|---|---|---|
| OpenRouter model | `deepseek/deepseek-v4.1-flash-1017` | `anthropic/claude-sonnet-5.5`, served by Anthropic |
| Run date (Kuala Lumpur time) | 1 and 2 November 2026 | 31 October 2026 |
| Agent commit | `568b32b` | `9e3fbc0` |
| Judge | `gpt-5.5-2026-04-23`, medium reasoning effort, prompt `wv-2.2` | the same |
| Attempts | one per task, ten-minute limit | the same |

Notes on the runs:

- Auto Browser's source is private. The two commits have the same agent code and differ only in the test harness, in ways that do not change how a finished task is graded.
- A task's cost is the change in the OpenRouter account's spend from its start to the next task's start. OpenRouter records a call's cost about 30 seconds after the call, so a few cents can land on the neighbouring task. The run totals are exact.
- In the DeepSeek run, the judge's API account ran out of credit near the end of the batch. The six Wolfram Alpha tasks were then run one at a time on the same code, model and judge.

## Licence

The task files derive from [WebVoyager](https://github.com/MinorJerry/WebVoyager) and from [Alumnium's patched copy](https://github.com/alumnium-hq/WebVoyager/tree/alumnium/data), both under the Apache License 2.0. This repository is released under the same licence; see `LICENSE` and `NOTICE`.

WebVoyager: Hongliang He et al., "WebVoyager: Building an End-to-End Web Agent with Large Multimodal Models", ACL 2024, [arXiv:2401.13919](https://arxiv.org/abs/2401.13919).
