# Phase 6: Feedback (Optional)

Builds an anonymized usage trace and directs the user to the Pulse survey form.

**Execute ALL steps in order. Do not skip or deviate.**

## Prerequisites

Read `$MIGRATION_DIR/.phase-status.json`. Verify `phases.discover == "completed"`. If not: **STOP**. Output: "Feedback requires at least the Discover phase to be completed."

## Step 0: Detect IDE and Version

Detect the IDE type and power version for the survey URL. These are passed as hidden fields — the user never sees or enters them.

### IDE Detection

Determine which IDE is running:

- **Kiro**: Check if the environment indicates Kiro (e.g., the `KIRO` environment variable is set, or the power was invoked via Kiro CLI/UI). Set `ide` to `kiro`.
- **Fallback**: If detection fails, set `ide` to `unknown`.

### Power Version Detection

Read the power version from `POWER.md` frontmatter (`version` field). If the frontmatter cannot be read or the field is missing, set `version` to `unknown`.

### Sanitization

Values must use only URL-safe characters: letters, numbers, dots (`.`), hyphens (`-`). Strip or replace any other characters.

Store the detected values as `$IDE_TYPE` and `$POWER_VERSION` for use in Step 2.

## Step 1: Build Trace

Load `steering/feedback-trace.md` and execute it. This produces `$MIGRATION_DIR/trace.json`.

If trace building fails: log the error, set `trace_included` to `false`, and skip to Step 3.

## Step 2: Show Trace and Provide Instructions

Read `$MIGRATION_DIR/trace.json` and display it pretty-printed so the user can see exactly what data is included:

```
--- Anonymized Trace (what will be shared) ---

<pretty-printed trace.json>

--- End Trace ---

This trace contains only aggregate counts, enum values, and timing data.
No resource names, file paths, account IDs, or secrets are included.
```

Then output the single-line minified version for copy-paste:

```
--- Copy the line below and paste it into the "Migration trace (optional)" field ---

<trace.json as single-line minified JSON — no newlines, no extra whitespace>

--- End ---
```

Then provide the survey link with IDE and version as hidden field query parameters:

```
Open the feedback form in your browser:
https://pulse.amazon/survey/MY0ZY7UA?ide=$IDE_TYPE&version=$POWER_VERSION

Answer the 5 quick questions in the form, then paste the trace line above
into the "Migration trace (optional)" field and submit.
```

Replace `$IDE_TYPE` and `$POWER_VERSION` with the actual values detected in Step 0. Example: `https://pulse.amazon/survey/MY0ZY7UA?ide=kiro&version=1.0.0`

## Step 3: Write feedback.json

Write `$MIGRATION_DIR/feedback.json`:

```json
{
  "timestamp": "<ISO 8601>",
  "survey_url": "https://pulse.amazon/survey/MY0ZY7UA?ide=$IDE_TYPE&version=$POWER_VERSION",
  "phases_completed_at_feedback": ["<list of completed phases>"],
  "trace_included": true
}
```

If trace building failed: set `"trace_included": false`.

## Step 4: Update Phase Status

Before status update, enforce output gate:

- `feedback.json` must exist.
- If `trace_included` is true, `trace.json` must exist.

If output gate fails: STOP and output: "Feedback outputs are incomplete. Fix feedback artifacts before completion."

Use the Phase Status Update Protocol (read-merge-write) to update `.phase-status.json` with `phases.feedback` set to `"completed"` and `current_phase` set to `"feedback"` — **in the same turn** as the output message below.

Output to user: "Thank you for helping improve this tool."

After feedback completes, return control to the workflow execution in POWER.md. The calling checkpoint determines whether to advance to the next phase or end the migration.
