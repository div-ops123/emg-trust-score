# Problem: Silent Drift in EMG-to-Force Decoding

## Context

Robot manipulation policies are usually trained on video and hand-pose data.
Neither of those tells you how hard a hand is gripping — a hand squeezing an
egg and a hand resting can look identical in pixel space and joint angles.
Force is the missing signal, and surface EMG (the electrical activity of the
forearm muscles that drive the hand) is one of the few non-invasive ways to
recover it directly, instead of guessing it from motion.

The value of this kind of data depends entirely on one thing: the inferred
force has to be trustworthy. If it quietly stops being accurate, a robot
policy still gets trained on it — just on the wrong lesson, with full
confidence.

## The failure mode

An EMG-to-force decoder is calibrated under certain conditions: a specific
electrode placement, a rested muscle, a specific wearer. Those conditions
don't hold for the whole length of a data-collection session. Two things
break them in different ways:

- **Electrode shift.** The band moves slightly on the wrist during use. The
  same true force now produces a different raw EMG reading than the one the
  decoder was calibrated on. The *signal* has shifted, not the person's
  actual force.

- **Muscle fatigue.** As a muscle tires, the body recruits more motor units
  to produce the same force, and the EMG frequency spectrum shifts. So for
  the *same true force*, the signal itself changes over time. This isn't a
  sensor problem — the underlying EMG-to-force relationship is no longer
  fixed.

Neither failure is visible from the outside. The sensor doesn't disconnect
and the pipeline doesn't error out — it just keeps producing force numbers,
increasingly wrong, indistinguishable from good data unless someone is
specifically watching for it.

## The question this project explores

**Can you tell, from the raw EMG signal alone, whether the current
EMG-to-force mapping is still trustworthy — without ever seeing the true
force?**

If drift shows up in signal properties (frequency content, amplitude trend,
cross-channel behavior) before or alongside where decoding error grows, that
signal could be used as a lightweight trust score: a flag on a session or
time window saying "this stretch of data is probably degraded," so it can be
down-weighted or reviewed before it reaches a training set.

## Scope of this repo

This project focuses on one failure mode only: **muscle fatigue drift within a
single continuous session, for a single wearer.**

Two other real failure modes exist in this space and are deliberately left
out of this v1:

- **Electrode shift** — no public dataset captures ground-truth electrode
  displacement, so this would require simulating it. Simulated drift is a
  weaker demonstration than a real, measured one, so it's left for a future
  version.
- **Cross-subject variance** — this is a generalization gap present from the
  start of a session, not something that drifts over time. It's a different
  problem (calibration transfer) from the one this repo is testing, so it's
  intentionally out of scope here rather than blended in.

What this repo does:

- Uses a public dataset of a single continuous, sustained isometric
  contraction, with EMG and true force recorded together throughout — one
  subject, one session, no proprietary hardware or data.
- Trains a simple baseline EMG-to-force regressor to make fatigue-driven
  decoding error visible and measurable over the session.
- Builds a signal-quality feature set (spectral median frequency, RMS
  amplitude trend) and checks whether it tracks the regressor's growing
  error as fatigue sets in — without ever using the true force to do it.
- Stops there. A real deployment would need multi-subject validation,
  electrode-shift handling, calibration baselines, and thresholding — none
  of which this repo attempts.

The goal is to show the fatigue failure mode is real, measurable, and
detectable from signal properties alone — not to solve it end-to-end.