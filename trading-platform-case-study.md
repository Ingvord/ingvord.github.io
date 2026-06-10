# From Idea to Live Bot in One Command

*Case study: an autonomous multi-agent pipeline that takes a trading strategy
from natural language to a deployed AWS Lambda bot, unattended.*

`status: live on AWS (eu-central-1) · testnet grace period, mainnet one env var away · ~$2.50/month total infra cost`

---

## The problem

The lifecycle of a new trading strategy used to look like this: write the idea
as Backtrader code, smoke-test it by hand, run an Optuna notebook, copy-paste
the results CSV, analyse parameters in a REPL, write a spec document manually,
translate the logic into the production bot's codebase, configure YAML, deploy,
and hope. Every handoff was manual, undocumented at the boundary, and dependent
on human memory between sessions. The gap between "idea" and "live bot" was
measured in days.

This is the same failure mode I've spent years removing from scientific
infrastructure: the system works, but every transition through it requires a
person to carry context that nothing else holds.

## What it does now

One Telegram command:

```
/pipeline "fast ema crosses slow ema" 5
```

produces a live deployed Lambda bot in a single unattended execution. Behind
that command, AWS Step Functions orchestrates four agents running on ECS
Fargate:

1. **Designer** (LLM) turns the natural-language idea into a strategy spec,
   a pure-Python implementation, and smoke tests.
2. **Selector** (deterministic) runs a quick single-pass backtest on every
   candidate instrument, about one minute per symbol, and ranks them by a
   composite score.
3. **Researcher** (deterministic) runs Optuna TPE optimisation, but only on
   the top three candidates, and selects the best parameter set with equity
   charts as evidence.
4. **Deployer** (LLM) generates the production Lambda from the spec, runs
   contract tests against it, executes `sam deploy`, registers the bot in the
   fleet config, and reports to Telegram.

Failures at any stage notify Telegram through SQS; fleet management afterwards
is a set of bot commands: `/fleet`, `/pause`, `/resume`, `/kill`, `/pnl`.

## The decisions worth writing down

**Strategy-as-spec.** The canonical artifact is `strategy_spec.json`: typed
parameters with ranges, a state-machine definition, signal types, and a
natural-language `logic` block. The Designer generates it, the Researcher
updates it with optimised values, the Deployer reads it to generate production
code. Research and production never share a framework; they share a contract.
This is the same instinct as the Tango REST specification: when two worlds
must cooperate, define the boundary as data, not as shared code.

**Screen cheap, optimise expensive.** The original design ran optimisation
before instrument selection. Implementation reality inverted it: full Optuna
(200 trials) across eight-plus symbols costs hours, while a single default-
parameter backtest costs a minute and eliminates instruments that have no
affinity with the strategy. The pipeline now spends its compute where the
candidates have already earned it.

**LLM agents only where judgment lives.** Two of the four agents use an LLM
(idea-to-spec, spec-to-code). The two in the middle, screening and
optimisation, are deterministic, reproducible, and cheap. Drawing that line
deliberately keeps the pipeline auditable: every number that reaches
production came from a backtest, not a model's opinion.

**Notify, don't block.** The first design had human approval gates at every
stage, built on Step Functions `waitForTaskToken`. Operating the pipeline
killed them: stage duration is unpredictable (Fargate capacity allocation can
delay a stage by an arbitrary amount), so a blocking gate becomes a race
between token expiry and a human noticing a Telegram message in time. Lose the
race and a multi-hour run dies waiting for a tap. The gates were deliberately
deactivated in favour of informed autonomy: all four agents run unattended,
and the Deployer sends a full summary report to Telegram before deploying.
The human sees everything and blocks nothing; `/pause` and `/kill` exist for
the cases where seeing is not enough. The gate infrastructure remains in the
codebase, switchable per agent, for stages that may one day need a hard stop.

**Notifications never hold credentials.** Agent containers post status to SQS;
a single notifier Lambda owns the Telegram token. ECS tasks that run
LLM-generated code never see a secret they don't need.

## What it caught along the way

The first manually built strategy (a mean-reversion RSI setup) surfaced bugs
that now live in the pipeline's institutional memory: Backtrader's Wilder RSI
versus pandas-ta's EMA-smoothed RSI silently disagree, so the production path
computes Wilder EWM by hand; percent-based position sizing floors to zero
contracts on high-priced instruments unless the backtest harness carries
enough cash; signal-ordering matters when the candle that resets an indicator
is also the candle that should trigger the entry. Each one is encoded as a
test or a documented constraint, so the next strategy author inherits the
lesson instead of the bug.

## Graduated by design

New bots deploy to Bybit testnet first. That isn't a limitation; it's the
promotion path. A freshly generated strategy trades through a grace period on
testnet, and once it proves profit, switching to mainnet is a single
environment variable. No redeploy, no code change: promotion is configuration,
and the human decision happens on the human's schedule, not inside a pipeline
timeout.

The same evidence-gated thinking applies to the runtime. The production Lambda
is Python today: cold starts are irrelevant at 5-minute tick intervals and a
bot costs about $0.01/month to run. The Java 21 / GraalVM native-image target,
already proven in a separate reference bot, is scheduled for when the fleet
holds at least three profitable bots, the point where the migration earns its
cost. Infrastructure investment follows demonstrated value, not the other way
around.

Surveillance is live: a scheduled Lambda polls trade data and computes fleet
equity, a read-only HTML dashboard shows fleet state, and the Telegram bots
answer `/status`, `/log`, `/pnl`, and `/positions` on demand. The next layer,
automatic drawdown-versus-backtest-baseline alerts and scheduled
reoptimisation, is designed and queued behind the same rule: it ships when the
fleet is large enough to need it.

## Stack

Python 3.12 agents (Claude via Bedrock) · AWS Step Functions · ECS Fargate ·
Lambda · DynamoDB · S3 · EventBridge · SAM/CloudFormation · Backtrader ·
Optuna · Telegram Bot API · Java 21 / GraalVM (reference production bot)

---

*What this demonstrates, in the framing of the rest of this site: build the
platform that makes the next thing possible (idea to live bot, one command),
generate evidence before committing (screen, then optimise, then promote:
testnet to mainnet, Python to Java, each gate opened by proof), and leave the
method behind (strategy-as-spec, the bug catalog, notify-don't-block as an
operational pattern). The system is small. The discipline is the product.*
