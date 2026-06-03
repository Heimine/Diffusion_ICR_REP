---
title: "Evaluating the Representation Space of Diffusion Models via Self-Supervised Principles"
permalink: /
layout: single
classes: wide
---

<p class="home-heading"><a href="https://heimine.github.io/" aria-label="Back to homepage"><span aria-hidden="true">&larr;</span> Back to homepage</a></p>

<p class="button-row">
<!-- <a class="btn btn--success" href="{{ site.github.repository_url }}"><i class="fab fa-github" aria-hidden="true"></i> Code</a> -->
<a class="btn btn--arxiv" href="#"><i class="fas fa-file-alt" aria-hidden="true"></i> arXiv</a>
<a class="btn btn--openreview" href="https://openreview.net/forum?id=2yMTCCHEjU"><i class="fas fa-book-open" aria-hidden="true"></i> OpenReview</a>
<a class="btn btn--warning" href="#"><i class="fas fa-chalkboard" aria-hidden="true"></i> Slides</a>
</p>

<p class="author-row">
<a class="author-link" href="https://heimine.github.io/"><strong>Xiao Li</strong></a><sup>1,*</sup>, <a class="author-link" href="https://umjiayx.github.io/">Yixuan Jia</a><sup>1,*</sup>, <a class="author-link" href="https://la0ka1.github.io/">Zekai Zhang</a><sup>1</sup>, <a class="author-link" href="https://scholar.google.com/citations?user=RO2ZlG8AAAAJ&hl=en/">Xiang Li</a><sup>1</sup>, <a class="author-link" href="https://shilianghe007.github.io/">Lianghe Shi</a><sup>1</sup>, <a class="author-link" href="https://jinxinzhou.github.io/">Jinxin Zhou</a><sup>2</sup>, <a class="author-link" href="https://zhihuizhu.github.io/">Zhihui Zhu</a><sup>2</sup>, <a class="author-link" href="https://liyueshen.engin.umich.edu/">Liyue Shen</a><sup>1</sup>, and <a class="author-link" href="https://qingqu.engin.umich.edu/"><strong>Qing Qu</strong></a><sup>1</sup>
</p>
<p class="affiliation-row">
<sup>1</sup>University of Michigan &nbsp;&middot;&nbsp; <sup>2</sup>Ohio State University
&nbsp;&middot;&nbsp; <sup>*</sup>Equal contribution
</p>

<p class="tldr-box"><strong>TL;DR.</strong> Diffusion models can be evaluated and monitored <em>from within</em>, using the geometry of their own learned representations — no labels, no sampling, no external networks needed.</p>

---

<p class="lead-italic"><em>Diffusion models are also representation learners</em></p>

<p>Diffusion models have achieved remarkable generative results, but their internal representations are also powerful: features extracted from a frozen diffusion backbone at the right noise level perform competitively with dedicated self-supervised learning (SSL) methods on downstream classification, segmentation, and correspondence tasks.</p>

<p>This raises a natural question: if diffusion models are strong representation learners, can we use the <em>quality of their representations</em> as a window into their generative behavior?</p>

<img class="feature-figure figure--narrow" src="{{ '/assets/figures/teaser_safe_final.png' | relative_url }}" alt="Tight clustering of invariant representations and semantic nearest neighbors." width="80%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>Invariant vs. residual components.</strong> Nearest neighbors via the invariant component <span class="math-inline">\(\bm{s}\)</span> are semantically related to the query; neighbors via the residual <span class="math-inline">\(\bm{\xi}\)</span> are not.</p>

---

<p class="lead-italic"><em>What makes a good representation?</em></p>

<p>Modern SSL methods are built on two complementary principles. First, <em>representation invariance</em>: features of different augmented views of the same image should remain stable. Second, <em>representation expansion</em>: features should spread out across the embedding space, avoiding collapse and preserving distinct image identities.</p>

<p>For diffusion models, these properties are <em>not explicitly optimized</em> — only a denoising objective is. Yet we find they emerge implicitly, and measuring them offers a natural diagnostic for diffusion models.</p>

<p>We decompose each representation into two components. For a training image <span class="math-inline">\(\bm{x}_0\)</span> and a random perturbation <span class="math-inline">\(a\)</span> (data augmentation + diffusion noise):</p>

$$
\bm{s}(\bm{x}_0) := \mathbb{E}_a\!\left[\bm{h}(a(\bm{x}_0))\mid \bm{x}_0\right], \qquad \bm{\xi}(a,\bm{x}_0) := \bm{h}(a(\bm{x}_0)) - \bm{s}(\bm{x}_0).
$$

<p>Here <span class="math-inline">\(\bm{s}\)</span> is the <em>invariant component</em> — stable across noisy and augmented views — and <span class="math-inline">\(\bm{\xi}\)</span> is the <em>residual component</em> capturing view-specific variation. The total feature covariance then decomposes cleanly as <span class="math-inline">\(\bm{\Sigma}_h = \bm{\Sigma}_s + \bm{\Sigma}_\xi\)</span>.</p>

---

<p class="lead-italic"><em>The Invariant Contamination Ratio (ICR)</em></p>

<p>To summarize the health of the representation space in a single trackable scalar, we introduce the <strong>Invariant Contamination Ratio (ICR)</strong>. We solve the generalized eigenproblem <span class="math-inline">\(\bm{\Sigma}_s \bm{v}_i = \lambda_i \bm{\Sigma}_\xi \bm{v}_i\)</span>, where each eigenvalue <span class="math-inline">\(\lambda_i\)</span> measures the invariant signal-to-noise ratio along a <button class="inline-note-trigger" type="button" aria-expanded="false" aria-controls="note-fisher" data-note-target="note-fisher">Fisher direction</button>:</p>

<div id="note-fisher" class="inline-note-body" hidden>
  <p>This follows the same generalized eigenstructure as classical Fisher Linear Discriminant Analysis, where <span class="math-inline">\(\bm{\Sigma}_s\)</span> and <span class="math-inline">\(\bm{\Sigma}_\xi\)</span> play roles analogous to between-class and within-class covariances — with each individual image acting as its own "class."</p>
</div>

$$
\mathrm{ICR} := \frac{1}{1 + \frac{1}{d}\sum_{i=1}^d \lambda_i}.
$$

<p>ICR approaches 0 when the invariant component dominates across all Fisher directions (a clean, structured representation), and approaches 1 when residual variation floods the space (a contaminated one). Crucially, ICR is <em>entirely label-free</em> and computed from training features alone — no generation, external encoders, or held-out data.</p>

<img class="feature-figure" src="{{ '/assets/figures/icr_noise.png' | relative_url }}" alt="ICR and classification accuracy plotted against noise level for CIFAR10, CIFAR100, and ImageNet, showing aligned minima." width="85%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>ICR predicts downstream performance across noise levels.</strong> The noise levels that minimize ICR coincide with those that maximize linear classification accuracy — a <em>semantic window</em> where representations are strongest.</p>

---

<p class="lead-italic"><em>A semantic window in the noise schedule</em></p>

<p>The diffusion noise schedule creates a family of representations indexed by noise level <span class="math-inline">\(\sigma_t\)</span>. At very low noise, representations are entangled with fine-grained, augmentation-specific details. At very high noise, they collapse toward uninformative Gaussian structure. In between, an intermediate range yields the richest semantic structure.</p>

<p>ICR identifies this range automatically. Across CIFAR10, CIFAR100, and ImageNet, the ICR curve is U-shaped and attains its minimum around the noise levels where linear probe accuracy peaks. This gives a simple, label-free rule for selecting the best noise scale for diffusion features (no downstream classifier required during selection).</p>

---

<p class="lead-italic"><em>Tracking generalization and memorization during training</em></p>

<p>We fix an intermediate noise scale and track ICR as training progresses. The behavior splits cleanly by data regime.</p>

<p>In the <strong>data-rich regime</strong>, ICR and FID decrease simultaneously throughout training. As the model better approximates the underlying distribution, its internal features shift toward stable, low-dimensional structure. ICR captures this improvement <em>from the inside</em>, without the need to generate new samples.</p>

<img class="feature-figure" src="{{ '/assets/figures/icr_fid_rich.png' | relative_url }}" alt="ICR and FID both decrease monotonically during data-rich training on CIFAR10 and ImageNet." width="80%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>ICR tracks FID in the data-rich regime.</strong> Both decrease monotonically, reflecting improving representation invariance and generation quality in parallel.</p>

<p>In the <strong>data-limited regime</strong>, the picture changes sharply. ICR follows a clear U-shaped trajectory: it first decreases as the model learns genuine structure, then rises again as the model begins fitting sample-specific idiosyncrasies. Recent work has documented this <button class="inline-note-trigger" type="button" aria-expanded="false" aria-controls="note-early" data-note-target="note-early">early learning phenomenon</button> — but prior detection methods either require large numbers of generated samples or rely on FID, which is known to be unreliable for memorization detection.</p>

<div id="note-early" class="inline-note-body" hidden>
  <p>The early learning phenomenon in diffusion models — where image quality initially improves before the model begins to memorize — has been documented across several recent works. See for exmaple <a href="https://arxiv.org/abs/2410.24060">Li et al. (2024)</a> and <a href="https://arxiv.org/abs/2505.17638">Bonnaire et al. (2026)</a>.</p>
</div>

<img class="feature-figure" src="{{ '/assets/figures/c10_4096_icml_memratio.png' | relative_url }}" alt="ICR follows a U-shaped curve in data-limited training; the memorization ratio begins rising after the ICR minimum." width="85%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>ICR anticipates memorization.</strong> The memorization ratio remains near zero around the ICR minimum and rises only afterward — making ICR a practical, label-free early stopping signal.</p>

<p>The ICR minimum marks the transition point: beyond it, the model gradually starts to memorize individual training samples rather than learning shared semantic structure. The nearest neighbors of the invariant component track this transition qualitatively — they are semantically meaningful near the ICR minimum and degrade on both sides.</p>

<img class="feature-figure" src="{{ '/assets/figures/nearest_neighbors_training.png' | relative_url }}"
  alt="Nearest neighbors of the invariant component at three training stages: early, ICR minimum, and severe overfitting."
  width="80%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>Nearest neighbors track the training transition.</strong> At the ICR minimum (middle row), retrieved neighbors are more semantically close than the early and late stages, where ICR is larger.</p>

---

<p class="lead-italic"><em>What drives feature expansion?</em></p>

<p>ICR is a relative measure; it does not directly reveal how total representation energy evolves. Examining the traces <span class="math-inline">\(\mathrm{Tr}(\bm{\Sigma}_s)\)</span> and <span class="math-inline">\(\mathrm{Tr}(\bm{\Sigma}_\xi)\)</span> separately also shows a difference between training regimes. In the data-rich setting, growing feature capacity is predominantly devoted to the invariant component <span class="math-inline">\(\bm{\Sigma}_s\)</span>. In the data-limited setting, once the limited semantic structure has been largely extracted, further feature expansion is dominated by the residual component <span class="math-inline">\(\bm{\Sigma}_\xi\)</span> — the model starts "filling up" with noise rather than signal.</p>

<img class="feature-figure" src="{{ '/assets/figures/trace_comparison.png' | relative_url }}" alt="Bar charts showing that data-rich training expands the invariant covariance trace, while data-limited training expands the residual trace." width="80%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>Feature expansion is driven by invariant structure when data are abundant, and by residual variation when data are scarce and invariant structure signal is saturated.</strong></p>

---

<p class="lead-italic"><em>A unified perspective</em></p>

<p>Our results suggest that diffusion models can be monitored and understood alternatively through the geometry of their own learned representations. ICR provides an intrinsic training-time signal that is:</p>

<ul>
  <li><strong>Label-free</strong> — no class annotations or external encoders needed.</li>
  <li><strong>Sample-free</strong> — computed from training features, not generated images.</li>
  <li><strong>Sensitive</strong> — detects the onset of memorization.</li>
  <li><strong>Interpretable</strong> — rooted in a clean invariant/residual decomposition with a direct signal-to-noise interpretation.</li>
</ul>

<p>This work is part of a growing effort to bridge self-supervised representation learning and generative diffusion modeling, showing that insights from one enrich our understanding of the other.</p>