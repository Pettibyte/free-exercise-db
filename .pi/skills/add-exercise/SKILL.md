---
name: add-exercise
description: |
  Add one or more new exercises to this free-exercise-db repository as valid JSON.
  Use when the user asks to add, include, register, or create an exercise, mobility
  drill, stretch, lift, or movement in this repo, including when they supply a
  reference URL, a common name, or a description of the movement. Resolves
  ambiguity with ask_user, derives the exercise id, fills every required schema
  field, writes exercises/<Id>.json with an empty images array, regenerates
  dist/exercises.json, and validates with the repo Makefile targets.
license: Unlicense
metadata:
  scope: repo-local
  schema: schema.json at the repository root
---

# Add Exercise

Add exercises to this repository without breaking the schema, the ids, or CI.

The repo stores one JSON document per exercise in `exercises/`. `dist/exercises.json`
is generated. GitHub Actions runs `make lint` and fails if `dist/exercises.json` is
not byte-identical to a fresh build. Follow the steps in order.

Run every command in this skill from the repository root. That is the directory that
holds `Makefile` and `schema.json`. Do not assume an absolute path; the checkout may
move.

## Rules That Are Not Negotiable

1. **Never hand-edit `dist/exercises.json`.** Regenerate it with `make dist/exercises.json`.
2. **Never copy text from a reference URL.** The user's link shows the movement only.
   Write the instructions yourself, from your own knowledge of the movement.
3. **No images unless the user supplies them.** Set `"images": []`.
4. **Every required field must be present.** See the field table.
5. **Do not commit or push** unless the user asks.

## Step 1: Pick The id

The `id` equals the filename without `.json`. The pattern is `^[0-9a-zA-Z_-]+$`.

Follow the existing repo convention:

| Movement name | id / filename |
| --- | --- |
| Wall Sit | `Wall_Sit` |
| 90/90 Hip Stretch | `90_90_Hip_Stretch` |
| Reverse Tabletop | `Reverse_Tabletop` |
| Dead Hang | `Dead_Hang` |
| 3/4 Sit-Up | `3_4_Sit-Up` |

- Capitalize each word. Join words with `_`.
- Replace `/`, spaces, and commas with `_`. Keep existing hyphens.
- Do not use apostrophes, periods, parentheses, or slashes in an id.
- Check the id is free before you write: `ls exercises/ | grep -i <name>`.

## Step 2: Resolve Ambiguity With ask_user

Some choices need a human. Ask, but ask only what the repo cannot answer.

Batch every open question into **one** `ask_user` call. Do not ask one question per
turn. Ask **before** you write the file, so you write it once.

Rules for the questions:

- Use the schema's exact strings as the option values. If you ask about `category`,
  the options are `stretching` and `strength`, not prose. The answer must be usable
  with no further translation.
- Put your recommended option first and say why in the `detail` field.
- Skip any question the user already answered in their request.
- Skip anything you can settle by reading `schema.json` or a nearby file in
  `exercises/`. A question the repo can answer is a question you wasted.
- If the user leaves a question unanswered, take the default below and state that
  you did so in your final report. Do not re-ask.

Ask when one of these is genuinely unclear:

| Ambiguity | Question | Default if unanswered |
| --- | --- | --- |
| A hybrid movement could be a stretch or a strength hold. A dead hang lengthens the shoulder or trains grip. | Which is this for: mobility (`stretching`) or strength (`strength`)? | `stretching` when the user named mobility, flexibility, or recovery. `strength` when they named reps, load, or training. |
| The primary muscle is a close call. A reverse tabletop is glutes or shoulders depending on cueing. | Which muscle should be primary? | The muscle that moves the most weight or holds the most load. |
| The level is beginner for some people and intermediate for others. | What level should this be? | `beginner`. |
| The name collides with an existing exercise. "Hip stretch" already has `90_90_Hip_Stretch`. | Add a new file, or edit the existing one? | New file with a distinct name. Never silently overwrite. |
| The user may have images to add. | Do you have image files, or should `images` stay empty? | `"images": []`. |
| Whether to commit. | Commit this, or leave it for you? | Leave it uncommitted. |

## Step 3: Fill Every Required Field

Allowed values come from `schema.json`. Do not invent values.

| Field | Type | Allowed values |
| --- | --- | --- |
| `id` | string | `^[0-9a-zA-Z_-]+$`, equals filename |
| `name` | string | Human-readable display name, e.g. `"Wall Sit"` |
| `force` | string or null | `static`, `pull`, `push`, `null` |
| `level` | string | `beginner`, `intermediate`, `expert` |
| `mechanic` | string or null | `isolation`, `compound`, `null` |
| `equipment` | string or null | `null`, `medicine ball`, `dumbbell`, `body only`, `bands`, `kettlebells`, `foam roll`, `cable`, `machine`, `barbell`, `exercise ball`, `e-z curl bar`, `other` |
| `primaryMuscles` | array | one or more from the muscle list below |
| `secondaryMuscles` | array | zero or more from the muscle list below |
| `instructions` | array | 3-6 step strings, each a complete instruction |
| `category` | string | `powerlifting`, `strength`, `stretching`, `cardio`, `olympic weightlifting`, `strongman`, `plyometrics` |
| `images` | array | `[]` when there are no images |

Muscle list, exact strings only:
`abdominals`, `abductors`, `adductors`, `biceps`, `calves`, `chest`, `forearms`,
`glutes`, `hamstrings`, `lats`, `lower back`, `middle back`, `neck`, `quadriceps`,
`shoulders`, `traps`, `triceps`.

### How To Choose The Fields

- **Isometric hold, stretch, or mobility drill:** `force` = `static`. Use `static`
  for a wall sit, a deep squat hold, a hip stretch, or a yoga pose.
- **Hanging or pulling a load toward you:** `force` = `pull`.
- **Pushing a load or your body away:** `force` = `push`. A reverse tabletop pushes
  the floor away, so it is `push`.
- **Bodyweight work with no gear:** `equipment` = `body only`. Use `null` only when
  the repo pattern for that drill has no equipment value at all.
- **`mechanic`:** use `compound` when two or more joints move, `isolation` when one
  joint moves. Use `null` for stretches and partner drills, matching nearby files.
- **`category`:** mobility, flexibility, and decompression work goes in
  `stretching`. Holds that build strength go in `strength`. A dead hang for
  shoulder lengthening is `stretching`; a wall sit is `strength`.
- **`level`:** `beginner` for a movement most people can attempt with cues.
  `intermediate` when it needs baseline strength or coordination, such as a reverse
  tabletop. `expert` for partner-loaded or high-mobility drills.
- **Muscles:** name the muscle the movement targets as primary. List stabilizers as
  secondary. Keep both lists short and honest. Do not list a muscle in both arrays.

## Step 4: Write The Instructions

Write 3-6 steps. Each step is one action the reader performs.

- Start with the setup. State body position, hand position, and foot position.
- Put the condition before the command: "If your heels lift, reduce the depth."
  Never put the command first and the warning after.
- Give a hold time or rep range, for example "Hold for 20-60 seconds."
- Use plain, direct sentences. Active voice. No marketing tone, no "unlock your
  potential", no emoji, no citation of the source article inside the data.
- Include one safety limit where the movement needs one: stop on sharp pain, keep the
  knees above the ankles, do not force the joint.
- Do not state medical claims. Describe the position and the feeling, not a
  diagnosis or a therapeutic promise. "You may feel a stretch across the hip" is
  fine. "This decompresses your discs" is not.

Start from `assets/exercise-template.json` in this skill directory. Copy it, fill
the values, and remove nothing.

## Step 5: Validate

The repo Makefile is the authority. Do not maintain a second validator.

`make lint` needs `check-jsonschema`. Install it once:

```bash
make install
```

If the system Python blocks the install, as PEP 668 does on some images, install
into a throwaway virtual environment and run the same schema from there:

```bash
python3 -m venv /tmp/cjs && /tmp/cjs/bin/pip install -q check-jsonschema
/tmp/cjs/bin/check-jsonschema --schemafile ./schema.json exercises/*.json
```

Run the repo checks:

```bash
make lint          # every field, every enum, the id pattern
make check_dupes   # must print no duplicate ids
```

Then run the two checks the schema cannot express. `schema.json` pattern-matches the
`id` but cannot see the filename, and it allows the same muscle in both muscle
arrays. Run these on the files you added, not across the whole directory. Sixteen
legacy files already carry muscle overlaps.

`id` must equal the filename. Prints nothing when every file matches:

```bash
for f in exercises/Wall_Sit.json; do
  [ "$(jq -r '.id' "$f")" = "$(basename "$f" .json)" ] || echo "id mismatch: $f (id=$(jq -r '.id' "$f"))"
done
```

No muscle may appear in both arrays. Prints one line per offender:

```bash
jq -r '. as $d
  | [ ($d.primaryMuscles // [])[] as $m
      | select((($d.secondaryMuscles // []) | index($m)) != null) | $m ]
  | select(length > 0)
  | input_filename + ": " + join(", ")' exercises/Wall_Sit.json
```

## Step 6: Rebuild The Distribution

```bash
make dist/exercises.json
jq 'map(select(.id == "Wall_Sit")) | length' dist/exercises.json
```

The second command must print `1`. Then confirm the diff only adds your exercises:

```bash
git diff --stat
git diff dist/exercises.json | head -60
```

## Step 7: Report

Tell the user:
- the file you created, by full path
- the field choices you made and why, in one line each, for `force`, `category`,
  and `level`
- any question you asked, and the answer
- any question the user left unanswered, and the default you applied
- which checks passed, and which did not run

Do not claim the schema passed unless `make lint`, or the virtual-environment
equivalent, actually ran.

## Adding Several Exercises

Add one file per exercise. Ask all clarification questions for the whole batch in
one `ask_user` call. Run the checks once, over all new files, then rebuild `dist`
once. Do not rebuild `dist` between files.

## Files In This Skill

- `assets/exercise-template.json` — copy this to start a new exercise file.

This skill ships no validator on purpose. `make lint` and `schema.json` are the
single source of truth. A second validator drifts from the schema and can report a
pass while CI fails.

## Anti-Patterns

- Writing instructions copied or lightly reworded from a blog post.
- Setting `images` to a path that has no file behind it.
- Adding a field the schema does not list, such as `tips`, `duration`, or `source`.
- Editing `dist/exercises.json` by hand, then rebuilding and losing the edit.
- Choosing a muscle name that is close but not in the enum, such as "hip flexors",
  "core", or "obliques". None of those exist in the schema.
- Putting a stretch in `strength` or a strength hold in `stretching` without a
  reason you can state.
- Asking the user a question the repo can answer, or asking one question per turn
  instead of one batched `ask_user` call.
- Hardcoding an absolute repository path in a command or note.
