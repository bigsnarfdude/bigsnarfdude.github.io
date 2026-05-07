---
permalink: /paper/
sitemap: false
---

<meta name="robots" content="noindex, nofollow">

<style>
.paper-container { max-width: 860px; margin: 0 auto; font-family: Georgia, serif; }
.paper-title { font-size: 1.5em; font-weight: bold; margin-bottom: 0.3em; line-height: 1.3; }
.paper-authors { color: #555; margin-bottom: 1em; }
.abstract { background: #f9f9f9; border-left: 3px solid #aaa; padding: 1em 1.2em; margin: 1.5em 0; font-size: 0.95em; line-height: 1.6; }
.abstract h3 { margin-top: 0; font-size: 1em; text-transform: uppercase; letter-spacing: 0.05em; color: #444; }
.figure-row { display: flex; gap: 1em; margin: 2em 0; flex-wrap: wrap; }
.figure-block { flex: 1 1 45%; min-width: 280px; }
.figure-block img { width: 100%; border: 1px solid #ddd; }
.figure-caption { font-size: 0.82em; color: #444; margin-top: 0.4em; line-height: 1.4; }
.figure-caption strong { color: #222; }
.download-btn { display: inline-block; background: #2c5f8a; color: white; padding: 0.6em 1.2em; text-decoration: none; border-radius: 3px; font-size: 0.95em; margin: 1em 0 1.5em; }
.download-btn:hover { background: #1e4263; color: white; }
.section-divider { border: none; border-top: 1px solid #ddd; margin: 2em 0; }
.pdf-embed-label { font-size: 0.85em; color: #666; margin-bottom: 0.5em; }
</style>

<div class="paper-container">

<div class="paper-title">Clean Fine-Tuning Rotates the Authority-Flip Response Along the Confidence Axis</div>
<div class="paper-authors">Anonymous Authors &mdash; Under Review</div>

<a class="download-btn" href="/assets/papers/iatrogenic_paper_fixed.pdf">Download PDF</a>

<div class="abstract">
<h3>Abstract</h3>
<p>When people consult language models for medical advice, reliability is the whole point. Yet safety-trained chat models suffer an <em>authority flip</em>: they abandon the correct multiple-choice answer when a fabricated "clinical-guideline update" is prepended to the prompt, even though the fake update contains zero evidence for the flip. Does clean supervised fine-tuning fix this?</p>

<p>Clean SFT does not cure the flip; it <strong>rotates</strong> it. The flip rate drops when the base model was already uncertain (Q1 &minus;20.3pp, paired McNemar <em>p</em> = 1.9&times;10<sup>&minus;3</sup>) and rises when the base model was already confident (Q4 +10.1pp, <em>p</em> = 4.2&times;10<sup>&minus;3</sup>), on the same 500-item MedMCQA pool. The full-pool average is &minus;4.6pp, <em>p</em> = 0.088 — not significant.</p>

<p>We rank attention heads by per-head OV projection onto the Q4&minus;Q1 confidence direction, then zero-ablate the top three at layers 25 and 31. Doing this <em>before</em> fine-tuning defends across every confidence band (+15.0pp full pool, McNemar <em>p</em>=5.97&times;10<sup>&minus;11</sup>, <em>n</em>=500). A matched orthogonal probe is mildly <em>anti</em>-defense (&minus;4.2pp). It is not "any six heads."</p>

<p><strong>Key finding:</strong> Clean fine-tuning that appears to average a vulnerability out may be <em>redistributing</em> it across the input distribution. Mechanistic localization, not aggregate deltas, distinguishes a real defense from a bookkeeping win.</p>
</div>

<hr class="section-divider">

<h2>Figures</h2>

<div class="figure-row">
  <div class="figure-block">
    <img src="/assets/images/fig1_rotation.png" alt="Figure 1: Confidence-stratified flip rates before and after clean SFT">
    <div class="figure-caption"><strong>Figure 1.</strong> Confidence-stratified flip rates before and after clean SFT. Q1 (low confidence) defends; Q4 (high confidence) worsens. The full-pool delta (&minus;4.6pp) masks the rotation entirely.</div>
  </div>
  <div class="figure-block">
    <img src="/assets/images/fig2_defense.png" alt="Figure 2: Head-ablation defense vs. orthogonal null across confidence strata">
    <div class="figure-caption"><strong>Figure 2.</strong> Head-ablation defense vs. orthogonal null across confidence strata. Compliance-direction ablation defends uniformly (+15pp); the matched orthogonal set mildly worsens every stratum.</div>
  </div>
</div>

<div class="figure-row">
  <div class="figure-block">
    <img src="/assets/images/fig3_steer.png" alt="Figure 3: Steering vector subtraction effect on baseline flip rate">
    <div class="figure-caption"><strong>Figure 3.</strong> Orthogonal residual probe subtracted from the untrained model halves the baseline flip rate (6.7%&rarr;3.4%) with accuracy pinned at 81.5%. The defense does not transfer post-SFT.</div>
  </div>
  <div class="figure-block">
    <img src="/assets/images/fig4_perhead.png" alt="Figure 4: Per-head OV norm contributions to the compliance direction">
    <div class="figure-caption"><strong>Figure 4.</strong> Per-head OV norm contributions to the compliance direction at layers 25 and 31. Top-3 heads account for a concentrated fraction of the total norm, validating the ablation target selection.</div>
  </div>
</div>

<hr class="section-divider">

<div class="pdf-embed-label">Full paper (PDF viewer):</div>
<iframe src="/assets/papers/iatrogenic_paper_fixed.pdf" width="100%" height="900px" style="border: 1px solid #ccc;"></iframe>

</div>
