# F3K Draw Master

A single-page web app for generating randomized competition draws for F3K (free-flight glider) events. Assigns pilots to groups, pairs them with timers, and selects tasks per round.

## How the Draw Works

The draw is generated in three stages: task selection, pilot-to-group assignment, and timer assignment. Each stage has its own constraints.

### 1. Task Selection

- A number of tasks are drawn randomly from the available task list.
- No duplicate tasks — each task can only appear once.
- The drawn tasks are flown in the order they were selected.

### 2. Pilot-to-Group Assignment

For each round, pilots are distributed across groups with the following constraints:

- **Equal group sizes**: Pilots are spread as evenly as possible across groups.
- **Minimize repeat encounters**: The algorithm tracks how often each pair of pilots has been in the same group across all previous rounds (an "encounter matrix"). It generates many random distributions and picks the one where pilots who have already flown together are least likely to be grouped again.
- **Even group rotation**: As a consequence of the randomized optimization, pilots tend to visit all groups roughly equally across rounds. This is measured by the "Group visits (min–max)" statistic.

### 3. Timer Assignment

After pilots are assigned to groups, each pilot is paired with a timer. Constraints:

- **Cross-group only**: A timer must be from a different group than the pilot they are timing. This ensures timers have no conflict of interest.
- **Mutual pairing**: Timers are assigned in pairs — if Pilot A times Pilot B, then Pilot B also times Pilot A.
- **Minimize repeated timer pairs**: The algorithm tracks all previous timer pairings and prefers partners that haven't been paired before. Multiple attempts are made to minimize the number of repeated pairs across the entire draw.
- **Avoid-tag groups (soft constraint)**: Pilots can be tagged with one or more group numbers using a semicolon after their name (e.g., `Pilot B; 1` or `Pilot C; 1, 3`). Pilots sharing any tag are preferably **not** paired as each other's timer. This is useful for separating top-level pilots, beginners, or pilots who shouldn't work together. The constraint is soft — if no better option exists, same-tag pairings are still allowed.
- **Odd pilot count**: If there is an odd number of pilots, one pilot per round will have no timer assigned.

## Draw Quality Statistics

After generating a draw, the following quality metrics are displayed:

| Statistic | Meaning |
|---|---|
| **Pairs (possible / needed / repeated)** | Total possible timer pairs, how many are needed across all rounds, and how many pairs had to be reused. |
| **Max times same opponents** | The highest number of times any two pilots were placed in the same group. Lower is better. |
| **Group visits (min–max)** | Across all pilots, the minimum and maximum number of times any pilot visited a single group. Closer values mean more even rotation. |
| **Min unique timers per pilot** | The fewest distinct timer partners any single pilot had across all rounds. Higher means better timer variety. |
| **Same-tag timer pairings** | Number of timer pairs where both pilots share an avoid-tag. Only shown when tags are used. Lower is better (0 = constraint fully satisfied). |

## Usage

Open `index.html` in a browser. No build step, dependencies, or server required.

### Pilot Input Format

Enter one pilot per line. Optionally append avoid-tags after a semicolon:

```
Pilot A
Pilot B; 1
Pilot C; 1, 3
Pilot D; 2, 3
Pilot E; 2
```

- `; 1` — assigns tag 1 (e.g., top pilots who shouldn't time each other)
- `; 1, 3` — assigns multiple tags (pilot avoids pairing with anyone sharing tag 1 **or** tag 3)
- No semicolon — no avoid-tag constraint

### Buttons

- **Generate All** — draw tasks, assign pilots to groups, and assign timers.
- **Generate Tasks** — re-draw only the tasks, keeping existing pilot/timer assignments if the round count matches.
- **Generate Pilots & Timers** — re-assign pilots and timers while keeping the current tasks.
- **Generate Timers** — re-assign only timers, keeping current tasks and group assignments.
- **Print / Save PDF** — print the draw; input controls are hidden automatically.
- **Download CSV** — export the draw as a CSV file.
