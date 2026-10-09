---
layout: default
title: "Open Inquiry"
show_title: false
---

# Open Inquiry

Evidence-led research across markets, language, technology, and systems.

<div class="intro">
Each report is a complete research memo, reproduced without abridgement:
the derivation chain, the evidence and epistemic status of its findings, the
record of corrections, and the questions left unresolved. Reports preserve
the research process rather than presenting only the final result.
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