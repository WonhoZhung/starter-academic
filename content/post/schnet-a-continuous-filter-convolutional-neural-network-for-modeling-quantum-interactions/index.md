---
title: "SchNet: A Continuous-filter convolutional neural network for modeling
  quantum interactions"
date: 2021-09-07T01:53:07.741Z
draft: false
featured: false
tags:
  - Paper Summary
  - Invariance
image:
  filename: featured
  focal_point: Smart
  preview_only: false
---
This post is based on a paper [SchNet: A Continuous-filter convolutional neural network for modeling quantum interactions](https://papers.nips.cc/paper/2017/file/303ed4c69846ab36c2904d3ba8573050-Paper.pdf) by [Kristof T. Schütt](https://scholar.google.de/citations?user=0e49RfgAAAAJ) at [NIPS 2017](https://papers.nips.cc/paper/2017).

> We propose _SchNet_: a neural network specifically designed to respect essential quantum chemical constraints. _SchNet_ delivers both rotationally invariant energy prediction and rotationally equivariant force predictions.

### OUTLINE:
1. Backgrounds
2. Objectives
3. Proposed architecture of _SchNet_
4. Remarks from the experiments

<br>
<br>

---

<br>  
<br>  

#### 1. Backgrounds

#### 2. Objectives

#### 3. Proposed architecture of _SchNet_
##### (a) Continuous-filter convolutions
- The discrete filter is unable to capture the continuous changes in positions of the atoms, resulting in the discontinuous energy predictions.
- On the other hand, the continuous filter can capture the arbitrary positions of the atoms, and yield smooth energy predictions.

#### 4. Remarks from the experiments
