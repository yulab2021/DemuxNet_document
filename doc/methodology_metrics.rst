Methodology & Metrics
=====================

To mitigate stochasticity across deep learning runs and scale variance across converged models,
DemuxNet introduces two core metrics: **WIS** and **DCS**.

1. Weighted Importance Score (WIS)
----------------------------------

Definition
~~~~~~~~~~

WIS measures the comprehensive ranking score of a gene's contribution across multiple runs,
and assigns higher weights to high-accuracy models.

Formulas
~~~~~~~~

Model weight:

.. math::

   W_k = \left( \max\left(0, 2 \times (Acc_k - 0.5)\right) \right)^2

Single-run rank score:

.. math::

   S_{i,k} = 1 - \frac{Rank_{i,k}}{M}

Final WIS:

.. math::

   WIS_i = \frac{\sum_{k=1}^{N} (S_{i,k} \times W_k)}{\sum_{k=1}^{N} W_k}

2. Directional Consistency Score (DCS)
--------------------------------------

Definition
~~~~~~~~~~

DCS measures the stability of a gene's regulation direction across runs.

Formula
~~~~~~~

.. math::

   DCS_i = \frac{\sum_{k=1}^{N} (\mathrm{sign}(IG_{i,k}) \times W_k)}{\sum_{k=1}^{N} W_k}

Interpretation
~~~~~~~~~~~~~~

- :math:`DCS \approx 1`: Stable positive driver
- :math:`DCS \approx -1`: Stable negative driver
- :math:`|DCS| \le 0.5`: Ambiguous / noise

Screening Strategy
------------------

We recommend using a quadrant plot:

- X axis: DCS
- Y axis: WIS

Filter: keep genes with :math:`|DCS| > 0.5` and high WIS rank as robust drivers.
