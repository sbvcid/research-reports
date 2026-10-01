---
layout: default
title: "IBKR Research"
show_title: false
---

# IBKR Research

Buy-side research reports, published in full.

<div class="intro">
Each report is a complete internal research memo, reproduced without abridgement:
the derivation chain, the labelling of every figure by its epistemic status, the
record of figures that were wrong and how they were corrected, and an explicit
list of the questions the author declined to answer. Reports contain no price
target, no rating and no recommendation.
</div>

## Reports

<ul class="report-list">
{% assign reports = site.pages | where: "published", true | sort: "date" | reverse %}
{% for r in reports %}
  <li>
    <a class="r-title" href="{{ r.url | relative_url }}">{{ r.title }}</a>
    <p class="r-meta">
      <time datetime="{{ r.date | date_to_xmlschema }}">{{ r.date | date: "%-d %B %Y" }}</time>
      {% if r.report_id %}<span class="sep">&middot;</span> {{ r.report_id }}{% endif %}
    </p>
    <p class="r-desc">{{ r.description }}</p>
  </li>
{% endfor %}
</ul>

<p><a href="{{ '/feed.xml' | relative_url }}">RSS feed</a></p>