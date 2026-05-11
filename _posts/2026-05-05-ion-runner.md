---
title: "ION Runner"
date: 2026-05-05
permalink: /posts/2026/05/ion-runner/
tags:
  - games
  - fun
---

Whenever I am optimizing an instrument, particularly for a targeted analyte, I often 
think about the ion's journey from desorption to the detector. I had heard [quadrupole theory](https://www.nobelprize.org/prizes/physics/1989/paul/lecture/) described as balancing a marble on two saddles (alternating RF and DC voltages) 
spinning very quickly, trying to keep the marble from falling.

<div style="text-align: center;">
  <img src="/images/rotating_saddle_clean.gif" alt="Quadrupole saddle analogy" style="width: 50%; max-width: 300px;" />
</div>

I think about the [Mathieu stability diagram](https://www.process-insights.com/wp-content/uploads/2022/06/Process-Insights_Extrel_Practical-Quadrupole-Theory-Graphical-Theory.pdf), and whether my ion is being ejected from 
the quadrupole, crashing into one of the rods, or just dancing its way through.

So I created this little browser game. There are A LOT of artistic liberties taken here, 
but it's not too far from what I see in my mind when tuning the instrument parameters.

In **ION Runner**, you control the quadrupole field — not the ion itself. A charged ion 
drifts between two rod pairs, pulled toward whichever plate it drifts closest to. Your 
job is to balance the RF/DC field and maintain a stable trajectory.

Hold **Q** to increase the lower field and push the ion up. Hold **P** to increase the 
upper field and push the ion down. Watch out for other ions passing through — they will 
knock your ion off course. Maintain a stable trajectory long enough to reach **20ms** 
flight time and achieve detection.

Use **Q** and **P** on your keyboard, or tap the top and bottom halves of the game on mobile.

<iframe src="/assets/ion_runner.html" width="620" height="560" style="border:none; display:block; margin: 0 auto;"></iframe>
