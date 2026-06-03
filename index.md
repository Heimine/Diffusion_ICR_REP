---
title: "Evaluating the Representation Space of Diffusion Models via Self Supervised Principles"
permalink: /
layout: single
classes: wide
---

<p class="home-heading"><a href="https://la0ka1.github.io/" aria-label="Back to homepage"><span aria-hidden="true">&larr;</span> Back to homepage</a></p>

<p class="button-row">
<a class="btn btn--success" href="{{ site.github.repository_url }}"><i class="fab fa-github" aria-hidden="true"></i> Code</a>
<a class="btn btn--arxiv" href="#"><i class="fas fa-file-alt" aria-hidden="true"></i> arXiv</a>
<a class="btn btn--openreview" href="#"><i class="fas fa-book-open" aria-hidden="true"></i> OpenReview</a>
<a class="btn btn--warning" href="{{ '/assets/Diffusion_Model_Knows_When_It_Overfits_camready.pdf' | relative_url }}"><i class="fas fa-file-pdf" aria-hidden="true"></i> Paper</a>
</p>

<p class="author-row">
<strong>Xiao Li</strong><sup>1,*</sup>, Yixuan Jia<sup>1,*</sup>, Zekai Zhang<sup>1</sup>, Xiang Li<sup>1</sup>, Lianghe Shi<sup>1</sup>, Jinxin Zhou<sup>2</sup>, Zhihui Zhu<sup>2</sup>, Liyue Shen<sup>1</sup>, and Qing Qu<sup>1</sup>
</p>
<p class="affiliation-row">
<sup>1</sup>University of Michigan &nbsp;&middot;&nbsp; <sup>2</sup>Ohio State University &nbsp;&middot;&nbsp; <sup>*</sup>Equal contribution
</p>

<p class="tldr-box"><strong>TL;DR.</strong> We introduce ICR, a label free representation metric that tells when diffusion models learn robust semantic structure and when they begin to overfit.</p>

<img class="feature-figure" src="{{ '/assets/figures/fig_memorization.png' | relative_url }}" alt="ICR anticipates memorization during data limited diffusion training." width="92%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>Key message.</strong> In data limited training, ICR first improves and then degrades before strong memorization appears.</p>

---

<p class="lead-italic"><em>Diffusion models are not only generators. They are also representation learners.</em></p>

<p>Diffusion models achieve strong image generation performance, but their intermediate features are also useful for downstream tasks such as classification and segmentation. This raises a natural question: can we evaluate a diffusion model directly from its representation space, and can this reveal both representation quality and generation quality?</p>

<p>Our answer is yes. We view diffusion representations through two principles from self supervised learning: good features should be stable under semantics preserving perturbations, while still spreading out across different images.</p>

---

<p class="lead-italic"><em>Decomposing diffusion features into invariant and residual components.</em></p>

<p>For each image, we apply multiple perturbations, including standard data augmentation and diffusion noise. Given the representation <span class="math-inline">\(h(a(x))\)</span>, we decompose it as</p>

$$
h(a(x)) = s(x) + \xi(a,x),
$$

<p>where <span class="math-inline">\(s(x)\)</span> is the invariant component shared across views, and <span class="math-inline">\(\xi(a,x)\)</span> is the residual component that captures view specific variation.</p>

<img class="feature-figure" src="{{ '/assets/figures/fig_decomposition.png' | relative_url }}" alt="Nearest neighbors of invariant and residual components." width="90%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>Invariant versus residual features.</strong> Nearest neighbors based on <span class="math-inline">\(s\)</span> are semantically meaningful, while neighbors based on <span class="math-inline">\(\xi\)</span> are much less structured.</p>

---

<p class="lead-italic"><em>A scalar metric: Invariant Contamination Ratio.</em></p>

<p>The decomposition induces two covariance matrices: <span class="math-inline">\(\Sigma_s\)</span> for invariant signal and <span class="math-inline">\(\Sigma_\xi\)</span> for residual variation. We compare them through a generalized eigenvalue problem, where each eigenvalue measures the invariant signal to residual noise ratio along one Fisher direction.</p>

$$
\Sigma_s v_i = \lambda_i \Sigma_\xi v_i,
\qquad
\mathrm{ICR}=\frac{1}{1+\frac{1}{d}\sum_{i=1}^{d}\lambda_i}.
$$

<p>A lower ICR means the representation is less contaminated by augmentation and noise sensitive residual variation. Importantly, ICR is label free and can be computed from training features alone.</p>

---

<p class="lead-italic"><em>ICR finds the semantic window across diffusion noise levels.</em></p>

<p>Diffusion models expose features across many noise levels. We find that ICR is minimized at intermediate noise levels, exactly where downstream classification accuracy is maximized. This gives a simple way to select strong diffusion features without training a downstream classifier.</p>

<img class="feature-figure" src="{{ '/assets/figures/fig_noise_window.png' | relative_url }}" alt="ICR and classification accuracy across noise levels." width="90%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>Semantic window.</strong> Across CIFAR10, CIFAR100, and ImageNet, the lowest ICR aligns with the highest linear probe accuracy.</p>

---

<p class="lead-italic"><em>ICR tracks generation quality and anticipates memorization.</em></p>

<p>In data rich training, ICR decreases together with FID, showing that stronger generation quality is reflected in cleaner internal representation geometry.</p>

<img class="feature-figure" src="{{ '/assets/figures/fig_data_rich.png' | relative_url }}" alt="ICR and FID decrease together in data rich training." width="82%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>Data rich regime.</strong> As training progresses, both ICR and FID improve.</p>

<p>In data limited training, ICR follows a clear early learning pattern: it first decreases, then increases before the memorization ratio rises. Thus, ICR can act as an intrinsic early stopping signal without generated samples, external evaluators, or held out data.</p>

<p>If we look at nearest neighbors retrieved using the invariant component <span class="math-inline">\(s\)</span>, the strongest semantic similarity appears when ICR reaches its minimum, before memorization begins. This suggests that the minimum ICR marks the point where the model learns the most robust semantic representations.</p>

<img class="feature-figure" src="{{ '/assets/figures/fig_neighbors_training.png' | relative_url }}" alt="Nearest neighbors of invariant components throughout limited data training." width="90%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>Representation quality over training.</strong> Nearest neighbors are most semantic near the minimum of ICR.</p>

---

<p class="lead-italic"><em>Where does feature expansion go?</em></p>

<p>Finally, the decomposition reveals how representations evolve during training. With abundant data, most of the growth in representation energy is allocated to the invariant component, corresponding to increasingly robust semantic features. With limited data, however, the invariant energy eventually saturates, and further growth is dominated by the residual component. This suggests that the model shifts from learning semantic structure to capturing sample specific details, which is indicative of memorization.</p>

<img class="feature-figure" src="{{ '/assets/figures/fig_energy_allocation.png' | relative_url }}" alt="Invariant and residual energy allocation in data limited and data rich training." width="78%" style="display:block;margin:auto;" />
<p class="figure-caption"><strong>Energy allocation.</strong> Data rich training expands invariant structure, while data limited training eventually expands residual variation.</p>

---

<p class="lead-italic"><em>Summary.</em></p>

<p>Our work introduces a representation based framework for monitoring diffusion models. ICR connects three phenomena in one intrinsic metric: downstream representation quality, generation quality, and the transition from generalization to memorization.</p>

<p class="tldr-box"><strong>Takeaway.</strong> A diffusion model knows when it overfits through the geometry of its own learned representations.</p>

<script>
document.addEventListener('DOMContentLoaded', function () {
  document.querySelectorAll('.inline-note-trigger').forEach(function (button) {
    button.addEventListener('click', function () {
      const target = document.getElementById(button.dataset.noteTarget);
      const expanded = button.getAttribute('aria-expanded') === 'true';
      button.setAttribute('aria-expanded', String(!expanded));
      if (target) target.hidden = expanded;
    });
  });
});
</script>
