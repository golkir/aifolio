---
layout: page
title: Titled Tuesday Analysis
description: Statistical analysis of chess games played in Titled Tuesday events on chess.com
img: assets/img/project2/back.jpg
importance: 1
category: work
related_publications: false
---

## Overview
As part of this project, I created a dataset of chess games played in "Titled Tuesday" events on `chess.com` between July 2022 and December 2023 by scraping the website. Then, I executd various statistical analyses on it. The goal was to find interesting patterns and regularities in online chess tournaments.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/project2/chess.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/project2/data.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Chess is chase
</div>

### Statistical tools applied were:
- Descriptive statistics (mean, number of wins, losses, rank distribution).
- Game accuracy distribution analysis between tournaments rounds to find differences in accuracy between initial and final rounds
- K-means clustering and k-nearest neighbors analysis.
- Linear regression to predict game results based on player accuracy.
- Statistical tests (Kolmogorov-Smirnov test, Shapiro-Wilk normality test).
- Kernel Density Estimation (KDE) and visualization
- Maximum Likelihood Estimation of mean and standard deviation assuming chess data normality
- Correlation analysis of score and accuracy (Spearman's rank corellation coefficient)

## Repos
[Github](https://github.com/golkir/titled-tuesday-chess-statistical-analysis)

[Dataset](https://huggingface.co/datasets/kirillgoltsman/titled-tuesday-chess-games)

