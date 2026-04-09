# Fylla Periocular Workflow

This document summarizes the **publishable** workflow behind the second vision module I designed for Fylla, focused on the under-eye area.

It intentionally stays at workflow level. The production notebook, training code, weights, and deployment glue remain private.

## Goal

The goal of this extension was to move Fylla beyond acne-only analysis and add a dedicated periocular branch able to reason about under-eye signs such as:

- eye bags
- dark circles
- age-related / wrinkle-adjacent under-eye cues

## High-level pipeline

```mermaid
flowchart LR
    A["Face image"] --> B["Detect face and landmarks"]
    B --> C["Crop periocular region"]
    C --> D["Extract features with pretrained facial backbone"]
    D --> E["Train multi-task head for bags and age-related cues"]
    E --> F["Run label-quality checks and clean noisy samples"]
    F --> G["Extend with dark-circle task"]
    G --> H["Use replay to preserve previous behavior"]
    H --> I["Tune thresholds and export inference checkpoint"]
    I --> J["Integrate into Fylla website inference flow"]
```

## Workflow design

### 1. Region-focused preprocessing

Instead of feeding the whole face to the second model, I isolated the periocular region. The workflow uses face detection plus facial landmarks to crop a stable under-eye area before inference and training.

This choice keeps the model focused on the visual zone that actually matters for bags, dark circles, and nearby texture cues.

### 2. Transfer learning instead of training from scratch

The model starts from a pretrained facial backbone and then specializes it for the new task family. This made the workflow practical for product integration and reduced the amount of task-specific data needed compared with a full training-from-scratch pipeline.

### 3. Multi-task formulation

The first training stage was organized as a multi-task problem, so the model could learn several correlated eye-area signals together instead of treating each cue as a separate isolated project.

At a public level, the important idea is that the workflow was designed to share visual features across related periocular outcomes rather than duplicate independent pipelines.

### 4. Label-quality pass

One important issue in aesthetic datasets is label noise. Before relying on the training split as-is, I added a cleaning pass to identify likely mislabeled samples and reduce the impact of noisy supervision.

This helped make the final workflow more reliable without changing the product-facing behavior of the website.

### 5. Dark-circle extension without throwing away previous tasks

The later extension added a dedicated dark-circle output on top of the earlier eye-area workflow.

Instead of retraining only on the new task and risking a collapse of the previous behavior, I used a replay-oriented setup:

- keep a subset of earlier periocular data in the loop
- mix it with the new dark-circle data
- optimize the model so it learns the new target while retaining earlier useful behavior

This is the central workflow choice I most wanted to preserve in the public narrative, because it reflects the real product problem: extending a deployed analysis flow without losing what already works.

### 6. Thresholding and product integration

The final stage was not just "train a model and stop". I also evaluated decision thresholds and packaged the resulting checkpoint so the website could call the periocular module as part of the Fylla analysis experience.

That integration layer is private, but the important point is that the model work was done with deployment in mind from the start.

## Why this repository stays high-level

The public version stops at the workflow for a reason. The following pieces stay private:

- training notebooks and code
- checkpoints and model weights
- exact data organization
- production inference code
- internal website logic around how outputs are combined and surfaced

That is why this repository should be read as a **case study** and not as a reproducibility package.
