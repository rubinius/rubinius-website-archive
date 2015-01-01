---
layout: site
s3_base_url: "https://s3.amazonaws.com/releases.rubini.us/rubinius-"
---

Rubinius 2.x targets Ruby 2.x, where Rubinius 1.x targets Ruby 1.8

<table class="stack_effect">
  <thead>
    <tr>
      <th>Version</th>
      <th>Date</th>
      <th>Link</th>
    </tr>
  </thead>
  <tbody>
{% for release in site.data.releases %}
  <tr class="{% cycle 'odd', 'even' %}">
    <td>{{ release.version }}</td>
    <td>{{ release.date }}</td>
    <td><a href="{{ page.s3_base_url }}{{ release.version }}.tar.bz2">[bzip2]</a></td>
  </tr>
{% endfor %}
  </tbody>
<table>

