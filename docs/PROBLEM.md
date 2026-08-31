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

This is a small proof of concept, not a production system and not a
replacement for a real force decoder:

- Uses public surface EMG data with repeated/sustained contractions
  (fatigue-inducing), not proprietary hardware or data.
- Trains a simple baseline EMG-to-force regressor to make the drift visible
  and measurable.
- Builds a signal-quality feature set (spectral, amplitude, cross-channel)
  and checks whether it tracks the regressor's growing error over a session.
- Stops there. A real deployment would need per-subject calibration
  baselines, thresholding, and integration into a live collection pipeline —
  none of which this repo attempts.

The goal is to demonstrate the failure mode is real and measurable, and that
a first-pass detector is feasible from signal properties alone.