# Attention Sink and the Ceiling on Attention-Based KV Cache Selection

A single-experiment probe testing whether the attention-sink phenomenon
explains why query-attention signals underperform for KV cache reuse selection.

**Model:** Mistral-7B-Instruct-v0.3 | **Data:** MuSiQue (n=150) | **Hardware:** 2x T4
## Method

Ground truth: per-token L2 change in layer-31 key vectors between "all six
passages in one context" and "each passage encoded alone." The top 20% of
document tokens by this measure is the target set.

Selection signal: query-attention (Sparse-Q style) read at layer 12, summed
over query rows, averaged over heads.

Two interventions:
1. Sink masking. Zero the attention paid to the first 4 positions, renormalize
   each query row, rescore. Does removing sink mass improve ranking?
2. Tail-coverage diagnostic. Measure what fraction of attention mass the top
   20% of document tokens actually capture, per sample.

   ## Three findings:

1. **The sink replicates strongly.** Roughly 52% of query-attention mass lands
   on the first four tokens, confirming MagicPIG's observation on a different
   model and task.

2. **The sink does not explain the ceiling.** Masking it slightly *reduces*
   recall. Removing mass shared uniformly across query rows renormalizes without
   reordering document tokens. The sink explains the appearance of sparsity,
   not the failure of selection.

3. **Tail coverage predicts selection quality per sample.** Inputs where
   attention is peaky are where attention-based selection works; long-tailed
   inputs are where it fails. This links MagicPIG's geometry to the selection
   ceiling at the per-input level.
   ## Limitations

Single model, single dataset, n=150, fixed seed. Tail coverage and recall are
both derived from the same attention matrix, so finding 3 is a relationship
between a signal's shape and its accuracy, not a claim about independent
quantities. Sink masking uses the first 4 positions, following MagicPIG's own
convention; other definitions of the sink were not swept.
## Reference

Chen et al., MagicPIG: LSH Sampling for Efficient LLM Generation. ICLR 2025.
arXiv:2410.16179
