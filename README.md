# Task Scheduler

Give it your tasks with durations, priorities and deadlines, plus the hours you
are not available, and it lays out a day — with breaks it insists on.

**[Live → task-scheduler-lilac.vercel.app](https://task-scheduler-lilac.vercel.app/)**

---

## There is no AI in this

The app used to be called "AI Task Scheduler". It is not one, and the name has
been removed.

`src/lib/timetable-scheduler.ts` is 626 lines of deterministic, rule-based
scheduling: sort by priority and deadline, place each task in the first slot
that satisfies the constraints, then insert breaks. No model, no learning, no
inference, no network call. The same tasks on the same day always produce the
same timetable.

Calling it AI would have been the easier description and a false one. What it
actually is — a constraint scheduler that enforces a 90-minute attention limit
— is a more interesting thing to have built.

---

## What it does

- **06:00 to 23:00**, with the last task allowed to start at 22:45.
- **A 90-minute cap on continuous work.** Past that the scheduler will not
  place another task until a break has happened.
- **Two break lengths.** 15 minutes after a sustained stretch, 30 after two
  hours or more of accumulated work.
- **Unavailable blocks** — school, meals, anything — are placed first and
  scheduled around.
- **Priority and deadline ordering**, so the thing due tomorrow outranks the
  thing due next week at equal priority.
- **Never schedules into the past.** A timetable generated at 14:00 starts at
  14:00, and a past date returns nothing at all rather than a day you cannot
  act on.

Everything is stored in `localStorage`. No account, no server, no sync.

---

## How a day gets built

```
tasks + unavailable blocks + a date
   ↓  reject past dates outright
place unavailable blocks for that day
   ↓
carry over tasks already scheduled and still valid
   ↓
prioritizeTasks()   priority, then deadline
   ↓
placeTasks()        first slot that fits, respecting the 90-minute cap:
                    if continuous work ≥ 90 and the gap since the last task
                    is shorter than the break it has earned, skip the slot
   ↓
insertIntelligentBreaks()
                    short break after a task of 45 min or more
                    long break once continuous work reaches 90 min
   ↓
a timetable
```

The cap is enforced in two places on purpose: `placeTasks` refuses to put work
in a slot that would violate it, and `insertIntelligentBreaks` then makes the
break visible as a block on the timetable rather than leaving a mysterious gap.

---

## Stack

React 18, TypeScript, Vite, Tailwind, shadcn/ui (39 components vendored),
deployed on Vercel.

**The shell was scaffolded with Lovable.** The layout, the component library
and the initial wiring came from a generator. `src/lib/timetable-scheduler.ts`
is the part that was written rather than generated, and it is the part worth
reading.

---

## Getting started

```bash
git clone https://github.com/DDXPlayz/task-scheduler.git
cd task-scheduler
npm install
npm run dev
```

```bash
npm run build
npm run preview
```

No environment variables, no backend.

---

## Status & verification

**Status:** live, complete, not under active development. No tests.

| Claim | How it was checked |
|---|---|
| Live at `task-scheduler-lilac.vercel.app` | HTTP 200, checked 2026-11-21 |
| Builds from a bare clone | `npm install && npm run build` — **run, passes** |
| No AI anywhere | read the scheduler end to end: no model, no inference, no network call. The claim was removed from the title, the meta tags and the page heading |
| The constants are real | `dayStart = 6`, `dayEnd = 23`, `shortBreakDuration = 15`, `longBreakDuration = 30`, `maxContinuousWork = 90` in `timetable-scheduler.ts` |

**Not verified:** the scheduler's output has not been checked against a set of
worked example days; there are no tests. The `localStorage` keys still carry
their original `ai-scheduler-` prefix — deliberately, because the app is
deployed and renaming them would silently discard every existing user's saved
schedule.

---

## Credits

Written by **Dhruva Goyal**. Scaffolded with Lovable; the scheduling algorithm
is original. shadcn/ui components are MIT.

---

## Licence

[MIT](LICENSE).
