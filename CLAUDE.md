# How to work with me

I'm working in a timed session and need to understand and explain every change.
Favor small, reviewable steps over speed.

## Session start
- If NOTES.md and DECISIONS.md exist, read them first. They have the task,
  progress so far, and decisions already made.

## Planning
- Don't write or edit project files until I explicitly ask. If I ask for a plan,
  only plan. Read-only exploration (reading files, inspecting data) is fine anytime.
- Plans should be numbered steps, each small enough to test on its own.
  For each step, list the files it touches and how we'll verify it works.
- If a request is ambiguous, ask me clarifying questions until you are at least 95%
  confident about what I want before proceeding, instead of guessing.

## Implementing
- Implement one step at a time, then stop. Never continue to the next step unprompted.
- After each step, summarize exactly what changed and in which files.
- Stay within the scope I give you. No unrelated refactors, renames, or cleanups.
- Follow the existing patterns and conventions in this codebase.
- In new code, use long, descriptive variable names over short or abbreviated ones
  (e.g. trades_with_prices, not df2; daily_returns, not ret). Name output columns
  with their units (market_value_usd, return_pct). Don't rename existing variables
  unless I ask.

## Code structure
- Write data work as plain functions with type hints, in a script with a main()
  that reads as a summary of the pipeline and runs end to end with one command
  (if __name__ == "__main__").
- Keep file loading and saving separate from logic: transformation functions take
  and return DataFrames, so they can be tested with small hand-built inputs.
- Put thresholds and other tunable numbers in named constants at the top of the
  file, never as bare numbers inside functions.
- Keep it simple: no classes, frameworks, or elaborate error handling unless I ask.
- At the end of a run, print a short data quality summary: rows loaded per source,
  rows quarantined and why, unmatched keys, and final output size.
- Don't add new dependencies without asking me first.
- Don't make git commits. I'll commit myself.

## Explaining decisions
- When you make a non-obvious design choice, state the reason in one sentence
  and name the main alternative you didn't take.
- Point out trade-offs, assumptions, and known limitations instead of hiding them.

## Testing and debugging
- Never edit or delete tests to make them pass. Fix the code, or tell me why
  you think the test itself is wrong.
- When something fails, explain the likely cause and how to confirm it
  before changing any code.
- If two fix attempts don't work, stop and reassess assumptions with me.
- When writing tests, use tiny hand-built inputs whose expected values can be
  checked by reading them. Never derive expected values by running the code under test.
- Focus tests on logic that's easy to get subtly wrong (parsing, normalization,
  merges, grouped calculations), not on behavior pandas already guarantees.

## Comments
- Give every function we create a docstring: what it does, its inputs and outputs,
  and the assumptions it relies on.
- Add comments explaining any logic that isn't obvious at a glance: why it's done
  this way, edge cases handled, and decisions made.
- Always state assumptions in the code where they apply
  (e.g. "# Assumes one closing price per ticker per date").
- Don't add comments that merely restate a single line of code, and keep comments
  accurate as code changes.

## Inspecting data
- Before writing any transformation, inspect each source and report: shape, dtypes,
  null counts, duplicates, value_counts() for every text column, and a few sample rows.
  Never assume column names or formats.
- Look at every column, including ones the task doesn't mention. Flag status columns
  (cancelled, rejected, amended), fee columns, and currency columns, and ask me how
  to treat them.
- Flag numbers stored as text ("$1,234", "(50)" for negatives, "5%"), and units or
  currency differences between sources.
- Parse dates explicitly (format=...) and flag ambiguity: DD/MM vs MM/DD, timezones,
  trade date vs settlement date.
- Keep printed output compact: shapes, counts, head(), summaries. Never print whole
  DataFrames.

## Cleaning data
- Keep raw data untouched. Clean through functions that return new DataFrames,
  so the pipeline reruns from scratch.
- Normalize every source to one common schema (same column names, dtypes, and
  conventions) before combining any of them.
- Clean in this order: standardize column names, parse types (load ID columns as
  strings to keep leading zeros), normalize values (tickers, sides, signs, units,
  dates), deduplicate, handle missing values, validate.
- Before applying sign conventions, check whether quantities are already signed,
  so sells are never negated twice.
- When the same ID appears with different data (e.g. an amended trade), show me the
  rows and ask. Don't just keep the first.
- Never silently drop or fill values. Move invalid rows into a quarantine DataFrame,
  report missing-value counts, and propose options before filling.
- Never use information from after an event: no backfilling, no forward-looking
  merge_asof. If an exact date match fails, use the most recent earlier value and
  flag it as stale.
- Treat outliers (e.g. price moves over 50%) as things to investigate, not delete.
  Check for splits, unit mismatches, or bad data first.

## Combining and computing
- Before every merge: state the keys and join type, confirm the keys have the same
  dtype on both sides, check key uniqueness, and report row counts before and after
  plus unmatched rows. Use validate= and indicator=True, then drop the _merge column.
- Report row counts after every filter, cleaning step, and groupby. Check groupby
  keys for NaNs first (groupby drops them by default).
- Before implementing any metric, state its formula and the interpretation choices
  it makes (net vs gross, realized vs unrealized, weighted vs simple average,
  inclusive vs exclusive date ranges). If the task doesn't settle a choice, ask me.
- Sort by date before shift / pct_change / rolling, and group by ticker so
  calculations never cross from one ticker into another.
- Prefer vectorized operations over loops, iterrows, or row-wise apply.
- Use .loc for assignment (no chained indexing).
- Compare floats with a tolerance (np.isclose), never ==. Round only for final output.
- After key steps, add short assert checks (row counts, no unexpected nulls, totals
  reconcile across sources), including checks for assumptions stated in comments.
  Give every assert a message explaining what was expected.

## Decision log
- Keep a decision log in DECISIONS.md (create it if missing). Whenever we agree on a
  non-obvious decision or trade-off, append one entry: the decision, why, the main
  alternative rejected, and any downside. Only log decisions I've agreed to, not
  ones you're proposing. This file is an exception to waiting for my go-ahead.
