+++
title = "ML theory papers"
date = "2023-06-21"
+++

[Mert Pilanci](https://scholar.google.com/citations?hl=en&user=aSAS-aAAAAAJ&view_op=list_works&sortby=pubdate)

[Neural Networks are Convex Regularizers](https://arxiv.org/abs/2002.10553)

- Mert Pilanci.
- Any finite two-layer neural network with ReLU has an equivalent convex problem.
- [Talk](https://www.youtube.com/watch?v=1DWP44wKBhg) at Stanford.
    An audience member said that you want SDG to converge to a local minumum (for generalization?). That doesn't sound right?
- A previous result showed that an infinite-width network is convex.
    This paper on the other hand gives a convex problem that can be implemented and solved.

[All Local Minima are Global for Two-Layer ReLU Neural Networks](https://deepai.org/publication/all-local-minima-are-global-for-two-layer-relu-neural-networks-the-hidden-convex-optimization-landscape)

- Mert Pilanci.
- This [paper from arxiv](https://arxiv.org/abs/2006.05900) looks similar (same authors).
- The paper characterizes all local minima of the said NN.
- There appear to be precedent of "No spurious local minima"-type results.
- This [article](https://direct.mit.edu/neco/article/31/12/2293/95610/Every-Local-Minimum-Value-Is-the-Global-Minimum) by different authors also discuss "all local minima are global"-type results.

[Topology of deep neural networks](https://dl.acm.org/doi/abs/10.5555/3455716.3455900) (Lek-Heng Lim)

- The Betti number reduces as the data passes through the layers.
    An explanation for the success of many-layer networks.
- Compares ReLU with sigmoid. Topology changes more quickly with ReLU.
    Non-smoothness assists in changing the topology. Maybe this is why quantization has helped.


