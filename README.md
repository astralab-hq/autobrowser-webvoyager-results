# Auto Browser: WebVoyager-mini results

Results, per-task evidence and grading setup for Auto Browser, a browser agent that runs as a Chrome extension, on wv-mini: 103 tasks from the WebVoyager benchmark, seven from each of its 15 sites less two retired tasks. The write-up, with the cost analysis and limitations, is on the Auto Browser blog: [Auto Browser on WebVoyager-mini, final runs: 92.7% at 1.3 cents a task](https://autobrowser.dev/blog/webvoyager-mini-final-results).

| Run | Passed / graded | Pass rate | Model cost, 103 tasks | Cost per task |
|---|---|---|---|---|
| Auto Browser + DeepSeek V4.1 Flash, run 1 | 82 / 89 | 92.1% | $1.33 | $0.013 |
| Auto Browser + DeepSeek V4.1 Flash, run 2 | 84 / 90 | 93.3% | $1.40 | $0.014 |
| Auto Browser + Claude Sonnet 5.5 | 79 / 91 | 86.8% | $97.19 | $0.94 |

DeepSeek's mean over its two runs is 92.7%. Tasks stopped by a site's bot check are not graded: 14 and 13 in the DeepSeek runs, and 12 in the Sonnet run. Cost covers the agent's model calls and leaves out the judge. All results are self-reported.

The runs from the first blog post, 82.8% with DeepSeek on 105 tasks, are kept at the tag [`first-post-2026-11-02`](https://github.com/astralab-hq/autobrowser-webvoyager-results/tree/first-post-2026-11-02).

## Method

- **Tasks.** The first seven tasks from each WebVoyager site, in WebVoyager's order, with the task fixes [Alumnium published](https://github.com/alumnium-hq/WebVoyager/tree/alumnium/data). Four tasks were left out and replaced by the next task from the same site, and five were reworded. Two more were retired with no replacement, because the live site can no longer satisfy them. `tasks/wv-mini-patches.json` gives the reason for each change.
- **Grading.** GPT-5.5 (`gpt-5.5-2026-04-23`, medium reasoning effort) with our own prompt, in `judge/`. The judge reads the task, the run date, the final answer, the agent's actions with the page text it read, the final page, and a final screenshot when one was captured. A task passes only when every condition is met and the answer comes from the website.
- **Bot checks.** A task that hits a bot check is stopped and not graded. The agent is never allowed past one.
- **Attempts.** One attempt per task, with a ten-minute limit. All three runs were made on 7 November 2026 from Kuala Lumpur, Malaysia, between 16:07 and 22:14 local time, so the date was the same as in the contiguous United States. Sites served their Malaysian versions.
- **Reasoning.** Every model call asked the model to reason, except a last answer near the step or time limit and the retry after a reply that came back empty at the length limit.
- **Cost.** What OpenRouter recorded for the agent's model calls. The judge is not included.

## Contents

| Path | What it holds |
|---|---|
| `results/<run>.json` | Run summary and one row per task |
| `results/per-task.csv` | Each task's status and cost, all three runs side by side |
| `evidence/<run>/<task>.json` | One record per task: the task, the final answer, the judge's verdict and reason, the final URL and page title, cost, time, and every action the agent took |
| `evidence/<run>/screenshots/<task>.jpg` | The final-page screenshot the judge saw |
| `tasks/wv-mini/` | The 103 task files |
| `tasks/wv-mini-patches.json` | Every change to the upstream tasks, with its reason |
| `judge/prompt-wv-2.2.txt` | The judge's prompt template |
| `judge/pin.json` | The judge's model, reasoning effort and input limits |

`<run>` is `deepseek-v4.1-flash-run-1`, `deepseek-v4.1-flash-run-2` or `claude-sonnet-5.5`.

## Reading a task record

`status` is `pass` or `fail` for a graded task, and `blocked` for a task stopped by a bot check. A blocked task has no verdict, and `not_graded_because` names the check.

`actions` lists the agent's tool calls in order, each as `{ tool, args, ok, ms }`. Tool results and page text are left out. The judge saw them, so a verdict can rest on text that is not in the record. Session and tracking parameters were removed from URLs. Where the run's own record cut a long argument short, `args` holds that text as it was recorded.

## Runs

| | DeepSeek V4.1 Flash, runs 1 and 2 | Claude Sonnet 5.5 |
|---|---|---|
| OpenRouter model | `deepseek/deepseek-v4.1-flash-1017` | `anthropic/claude-sonnet-5.5`, served by Anthropic |
| Started (Kuala Lumpur time) | 7 November 2026, 16:07 and 19:58 | 7 November 2026, 18:14 |
| Agent commit | `29e9d03` | the same |
| Judge | `gpt-5.5-2026-04-23`, medium reasoning effort, prompt `wv-2.2` | the same |
| Attempts | one per task, ten-minute limit | the same |

Notes on the runs:

- Auto Browser's source is private. Commit `29e9d03` is part of release 1.6.0, which adds three small fixes made after these runs.
- A task's cost is the change in the OpenRouter account's spend from its start to the next task's start. OpenRouter records a call's cost about 30 seconds after the call, so a few cents can land on the neighbouring task. The run totals are exact.
- The judge was not consistent on `allrecipes-5`: it passed DeepSeek's second run and failed Sonnet's run with the same recipe. Both verdicts are as graded.

## Licence

The task files derive from [WebVoyager](https://github.com/MinorJerry/WebVoyager) and from [Alumnium's patched copy](https://github.com/alumnium-hq/WebVoyager/tree/alumnium/data), both under the Apache License 2.0. This repository is released under the same licence; see `LICENSE` and `NOTICE`.

WebVoyager: Hongliang He et al., "WebVoyager: Building an End-to-End Web Agent with Large Multimodal Models", ACL 2024, [arXiv:2401.13919](https://arxiv.org/abs/2401.13919).
