# diy-ralph

This project is currently a collection of notes, scripts and prompts that I used
to stand up a couple of "Ralph development loops". Geoffrey Huntley's [original](https://ghuntley.com/ralph/)
post is a little vague about what is _needed_ and what are _suggestions_. So here
is my list of "ingredients" and questions that you need to answer or implement
when you're standing up your own loop:

1. Where are you going to run Ralph? On an old laptop? In the cloud?
2. How do you like running long-running tasks?
    * I have a habit of using `screen` but the industry currently adores `tmux`
    * You may want to graduate to running it in `systemd` or `launchd`
3. How do you like tracking your "root" spec?
    * Geoffrey's [example](https://github.com/ghuntley/how-to-ralph-wiggum) uses
      `@IMPLEMENTATION_PLAN.md`
    * Anthropic's [note](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
      on long running agents recommends using JSON. I like `TASKS.jsonl` with a
      corresponding spec/plan/prd.

