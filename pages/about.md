---
layout: page
title: About
permalink: /about/
weight: 4
---

# **About**

**CORPORA** (COntext-free model checking for Recursive PrObabilistic pRogrAms) is a [MSCA Postdoctoral Fellowship](https://marie-sklodowska-curie-actions.ec.europa.eu/actions/postdoctoral-fellowships) project funded by the European Commission under the Horizon Europe Programme, grant ID [101107303](https://cordis.europa.eu/project/id/101107303).

## Description

IoT and embedded systems are powered by increasingly sophisticated software components, employing machine learning to create devices that perform activities once exclusively carried out by humans. Since these activities may involve significant risks and responsibilities, ensuring the correctness and safety of involved software components is crucial.

Probabilistic Programs (PPs) are often employed in AI-powered software, particularly to exploit Bayesian inference, and to model randomized algorithms. Thus, studying verification of PPs can enable verification techniques for ensuring safety and correctness of AI-powered programs. PPs are computer programs that, besides ordinary programming constructs, may contain random choices, and variable assignments according to a random distribution.

The CORPORA (COntext-free model checking for Recursive PrObabilistic pRogrAms) project aims at developing new techniques for the verification of Recursive Probabilistic Programs, one of the most expressive classes of PPs. Recursive PPs may contain recursive procedures. Since procedures are governed by a stack, they exhibit a context-free behavior in execution traces: verification of procedural programs has motivated the study of model checking pushdown formalisms. The need for specifying properties concerning the stack’s contents has led to the introduction of ad-hoc specification formalisms, including temporal logics featuring context-free-aware modalities, such as CaRet, NWTL and POTL, the latter being introduced by the applicant during his Ph.D.

The CORPORA project will extend the context-free model checking framework to recursive PPs, and its research objectives will encompass the theoretical and practical study of model checking thereof. The project’s expected results include a model checker for context-free properties of recursive PPs, which will enable the practical evaluation of the techniques developed in the project.

# **People**

{% include people/index.html %}


{% comment %}
<div class="row">
{% include about/skills.html title="Programming Skills" source=site.data.programming-skills %}
{% include about/skills.html title="Other Skills" source=site.data.other-skills %}
</div>
{% endcomment %}

# **Timeline**

<div class="row">
{% include about/timeline.html %}
</div>
