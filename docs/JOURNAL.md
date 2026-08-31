# Journal

## Why I picked this problem

Hemlock sells force labels, not raw EMG. That label is only worth something
if it's trustworthy. Nobody's watching for when it quietly stops being
trustworthy mid-session. That's the gap I wanted to poke at — small enough
for me to actually build, real enough to matter to their business.

## First mistake — I almost solved three problems at once

My first instinct was to lump electrode shift, muscle fatigue, and
cross-subject variance together as "drift." Wrong. Only fatigue is actually
drift — a relationship changing over time, same session, same person.
Electrode shift corrupts the input, not the relationship. Cross-subject
variance isn't drift at all — there was never one relationship to drift
from. Naming this correctly mattered more than I expected. If I'd built a
detector without separating these, I'd have been solving a fuzzy problem
and it would show.

## Second mistake — wrong dataset shape

I was ready to use Ninapro DB2 because it's the standard sEMG benchmark and
has 40 subjects with force data. But its trials are 5 seconds each. Fatigue
doesn't show up in 5 seconds. I needed the opposite of what Ninapro is good
at — not more subjects, one subject held for long enough to actually tire
out. Switched to a sustained isometric contraction dataset instead. Lesson:
more data isn't the fix when it's the wrong shape of data.

## Trade-off I'm making on purpose

I'm not touching electrode shift in v1. No public dataset has real
ground-truth shift events, and I don't want my first result to rest on
simulated data I made up myself. Better to nail one real, measurable
failure mode than to gesture at three shallow ones. I'll say this plainly
in the write-up instead of hiding it.

## What "done" looks like for v1

One subject, one session. A chart showing decoder error rise as the session
goes on. A chart showing my signal-quality features moving with it — before
I ever look at the true force. If those two charts line up, the core claim
holds: you can flag drift without ground truth. That's the whole v1.