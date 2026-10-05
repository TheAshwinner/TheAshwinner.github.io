---
layout: page
title: writing
subtitle: Notes, book highlights, anything random on my mind
permalink: /writing/
links:
  - name: letterboxd
    url: https://letterboxd.com/theashwinner/
# Topic buttons above the post list, in display order. `tag` is matched
# against the `tags:` list in each post's front matter; `default: true`
# buttons start switched on. A new button is an entry here plus that tag on
# the posts it covers - nothing else needs to change.
filters:
  - tag: ai
    label: AI
    default: true
  - tag: misc
    label: Misc
---

{%- comment -%}
Each button toggles on its own, and a post is shown if any of its tags is
switched on - so with both on, a post tagged ai and misc appears once. Tags
are slugified on both sides so `AI` or `Machine Learning` in a post still
match. A post with no tag matching any button is only visible without JS.
{%- endcomment -%}
{%- if page.filters %}
<div class="post-filter filter-bar" role="group" aria-label="Filter writing by topic">
  {%- for f in page.filters %}
  <button type="button" class="post-filter__btn filter-bar__btn" data-tag="{{ f.tag | slugify }}" data-default="{{ f.default | default: false }}" aria-pressed="false">{{ f.label | default: f.tag }}</button>
  {%- endfor %}
</div>
{%- endif %}

{% assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
<ul class="post-list">
{%- for year in posts_by_year %}
  <li class="post-list__year">
    <h2 class="post-list__year-heading">{{ year.name }}</h2>
    <ul class="post-list__items">
      {%- for post in year.items %}
      {%- capture tags %}{% for t in post.tags %}{{ t | slugify }} {% endfor %}{% endcapture %}
      <li class="post-list__item" data-tags="{{ tags | strip }}">
        <time class="post-list__date" datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%b %-d" }}</time>
        <a class="post-list__link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
      </li>
      {%- endfor %}
    </ul>
  </li>
{%- endfor %}
</ul>
<p class="post-list__empty" hidden>Nothing to show - switch a topic on above.</p>

{%- comment -%}
Same approach as the research filter on the home page: the list ships
unfiltered for no-JS readers and crawlers, and this runs during parse so the
default selection is applied before first paint.
{%- endcomment -%}
<noscript>
  <style>.post-filter { display: none; }</style>
</noscript>
<script>
  (function () {
    var bar = document.querySelector('.post-filter');
    if (!bar) return;

    var buttons = bar.querySelectorAll('.post-filter__btn');
    var years = document.querySelectorAll('.post-list__year');
    var empty = document.querySelector('.post-list__empty');

    function setOn(btn, on) {
      btn.classList.toggle('is-active', on);
      btn.setAttribute('aria-pressed', String(on));
    }

    function apply() {
      var selected = [];
      for (var i = 0; i < buttons.length; i++) {
        if (buttons[i].classList.contains('is-active')) {
          selected.push(buttons[i].getAttribute('data-tag'));
        }
      }

      var anyShown = false;
      for (var y = 0; y < years.length; y++) {
        var items = years[y].querySelectorAll('.post-list__item');
        var yearShown = false;
        for (var j = 0; j < items.length; j++) {
          var tags = (items[j].getAttribute('data-tags') || '').split(' ');
          var show = tags.some(function (t) { return selected.indexOf(t) !== -1; });
          items[j].hidden = !show;
          if (show) yearShown = true;
        }
        // Drop the year heading too, rather than leave it over an empty list.
        years[y].hidden = !yearShown;
        if (yearShown) anyShown = true;
      }
      if (empty) empty.hidden = anyShown;
    }

    for (var k = 0; k < buttons.length; k++) {
      setOn(buttons[k], buttons[k].getAttribute('data-default') === 'true');
      buttons[k].addEventListener('click', function () {
        setOn(this, !this.classList.contains('is-active'));
        apply();
      });
    }

    apply();
  })();
</script>

<p class="post-list__feed"><a href="{{ '/feed.xml' | relative_url }}">subscribe via rss</a></p>
