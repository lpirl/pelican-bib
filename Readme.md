Pelican BibTeX
==============

Organize your scientific publications with BibTeX in Pelican

Author          | Vlad Niculae
----------------|-----
Author Email    | vlad@vene.ro
Author Homepage | http://vene.ro
Github Account  | https://github.com/vene

*Note*: This code is unlicensed. It was not submitted to the `pelican-plugins`
official repository because of the license constraint imposed there.


Requirements
============

`pelican_bibtex` requires `pybtex`.

```bash
pip install pybtex
```

How to Use
==========

This plugin reads a user-specified BibTeX file and populates the context with
a list of publications, ready to be used in your Jinja2 template.

Configuration is simply:

```python
PUBLICATIONS_SRC = 'content/pubs.bib'
```

If the file is present and readable, you will be able to find the `publications`
variable in all templates.  It is a list of tuples with the following fields:
```
(key, year, text, bibtex, pdf, slides, poster)
```

1. `key` is the BibTeX key (identifier) of the entry.
2. `year` is the year when the entry was published.  Useful for grouping by year in templates using Jinja's `groupby`
3. `text` is the HTML formatted entry, generated by `pybtex`.
4. `bibtex` is a string containing BibTeX code for the entry, useful to make it
available to people who want to cite your work.
5. `pdf`, `slides`, `poster`: in your BibTeX file, you can add these special fields,
for example:
```
@article{
   foo13
   ...
   pdf = {/papers/foo13.pdf},
   slides = {/slides/foo13.html}
}
```
This plugin will take all defined fields and make them available in the template.
If a field is not defined, the tuple field will be `None`.  Furthermore, the
fields are stripped from the generated BibTeX (found in the `bibtex` field).

Split into lists of publications
--------------------------------

You can add an extra field to each bibtex entry. This field has a value as a comma seperated list.
These values are the keys of lists containing the associated bibtex entries.

For example, if you want to associate an entry with two different tags (foo-tag, bar-tag), 
you add the following field to the bib entry:

```
@article{
   foo13
   ...
   tags = {foo-tag, bar-tag}
}
```

You'll need to set `PUBLICATIONS_SPLIT_BY = tags` in your `pelicanconf.py`. In your template 
(see below), you can then access these lists with the variables `publications_lists['foo-tag']` 
and `publications_lists['bar-tag']`

With `PUBLICATIONS_UNTAGGED_TITLE = 'others'` you can assign all untagged entries 
to the variable `publications_lists['others']`.

Template Example
================

You probably want to define a 'publications.html' direct template.  Don't forget
to add it to the `DIRECT\_TEMPLATES` configuration key.  Note that we are escaping
the BibTeX string twice in order to properly display it.  This can be achieved
using `forceescape`.

```python
{% extends "base.html" %}
{% block title %}Publications{% endblock %}
{% block content %}

<script type="text/javascript">
    function disp(s) {
        var win;
        var doc;
        win = window.open("", "WINDOWID");
        doc = win.document;
        doc.open("text/plain");
        doc.write("<pre>" + s + "</pre>");
        doc.close();
    }
</script>
<section id="content" class="body">
    <h1 class="entry-title">Publications</h1>
    <ul>
    {% for key, year, text, bibtex, pdf, slides, poster in publications %}
    <li id="{{ key }}">{{ text }}
    [&nbsp;<a href="javascript:disp('{{ bibtex|replace('\n', '\\n')|escape|forceescape }}');">Bibtex</a>&nbsp;]
    {% for label, target in [('PDF', pdf), ('Slides', slides), ('Poster', poster)] %}
    {{ "[&nbsp;<a href=\"%s\">%s</a>&nbsp;]" % (target, label) if target }}
    {% endfor %}
    </li>
    {% endfor %}
    </ul>
</section>
{% endblock %}
```

Using lists of publications
---------------------------

The variable `publications_lists` is a map with the keys being the comma seperated entries of the field
defined in `PUBLICATIONS_SPLIT_BY`. You can replace `publications` from the previous example with
`publications_lists['foo-tag']` to only show the publications with the tag `foo-tag`. You can also iterate
over the map and present all bib entries of each list:

The section of the previous example changes to:

```python
...
<section id="content" class="body">
    <h1 class="entry-title">Publications</h1>

	{% for tag in publications_lists %}
	   {% if publications_lists|length > 1 %}
        		<h2>{{tag}}</h2>
	   {% endif %}
	   <ul>
	    {% for key, year, text, bibtex, pdf, slides, poster in  publications_lists[tag] %}
	    <li id="{{ key }}">{{ text }}
	    [&nbsp;<a href="javascript:disp('{{ bibtex|replace('\n', '\\n')|escape|forceescape }}');">Bibtex</a>&nbsp;]
	    {% for label, target in [('PDF', pdf), ('Slides', slides), ('Poster', poster)] %}
	    {{ "[&nbsp;<a href=\"%s\">%s</a>&nbsp;]" % (target, label) if target }}
	    {% endfor %}
	    </li>
	    {% endfor %}
	   </ul>
	{% endfor %}
</section>
...
```

Extending this plugin
=====================

A relatively simple but possibly useful extension is to make it possible to
write internal links in Pelican pages and blog posts that would point to the
corresponding paper in the Publications page.

A slightly more complicated idea is to support general referencing in articles
and pages, by having some BibTeX entries local to the page, and rendering the
bibliography at the end of the article, with anchor links pointing to the right
place.
