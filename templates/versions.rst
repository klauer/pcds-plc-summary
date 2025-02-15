{%- macro chapter(title) %}
{{ title | title_fill("=") }}
{{ title }}
{{ title | title_fill("=") }}{% endmacro -%}

{%- macro section(title) %}
{{ title }}
{{ title | title_fill("-") }}{% endmacro -%}

{%- macro subsection(title) %}
{{ title }}
{{ title | title_fill("^") }}{% endmacro -%}

{%- macro subsubsection(title) %}
{{ title }}
{{ title | title_fill('"') }}{% endmacro -%}

{% set all_references = [] %}
{% for project in projects.values() | sort(attribute="git_info.repo_slug") %}
    {% for name in get_library_versions(project) %}
        {% if name not in all_references %}
            {% set _ = all_references.append(name) %}
        {% endif %}
    {% endfor %}
{% endfor %}

{% set all_plcs = [] %}
{% set plc_to_project = {} %}
{% for project in projects.values() | sort(attribute="git_info.repo_slug") %}
    {% for plc in project.plcs %}
        {% set _ = all_plcs.append(plc) %}
        {% set _ = plc_to_project.update({plc: project}) %}
    {% endfor %}
{% endfor %}

TwinCAT Versions
================

This summary was generated |today|.

.. list-table::
    :header-rows: 1

    * - Project
      - TwinCAT Version
      - Pinned

{% for project in projects.values() | sort(attribute="git_info.repo_slug") %}
    * - {{ project.git_info.repo_slug }}
      - {{ project.attributes.TcVersion }}
      - {{ project.attributes.TcVersionFixed }}

{% endfor %}


Dependencies
============

This summary was generated |today|.

By PLC Project
--------------

{% for project in projects.values() | sort(attribute="git_info.repo_slug") %}
{% for plc in project.plcs | sort(attribute="name") %}

{{ subsection(project.git_info.repo_slug + ": " + plc.name) }}

Version `{{ project.git_info.describe }} <{{ project.git_info.tree_urls[0] }}>`_ - `Documentation <{{ project.git_info.doc_urls[0] }}>`_

{% set plc_versions = get_library_versions(plc) %}
.. list-table:: {{ project.git_info.repo_slug }}: {{plc.name}}
    :header-rows: 1

    * - Dependency
      - Library
      - Placeholder
      - Resolution

{% for ref_name, ref_info in plc_versions.items()|sort %}
    * - {{ref_name}}
{% for category in ["LibraryReference", "PlaceholderReference", "PlaceholderResolution"] %}
    {% if category in ref_info %}
      - ``{{ ref_info[category].version }}`` {{ ref_info[category].vendor_short }}

    {% else %}
      -

    {% endif %}
{% endfor %}
{% endfor %}
{% endfor %}
{% endfor %}

By Library
----------

{% for ref_name in all_references|sort %}

{{ subsection(ref_name) }}

.. list-table::
    :header-rows: 1

    * - Project
      - Library
      - Placeholder
      - Resolution

{% for plc in all_plcs | sort(attribute='name') %}
{% set plc_versions = get_library_versions(plc) %}
{% if ref_name in plc_versions %}
    * - {{ plc_to_project[plc].git_info.repo_slug }}:{{ plc.name }}
{% for category in ["LibraryReference", "PlaceholderReference", "PlaceholderResolution"] %}
    {% if category in plc_versions[ref_name] %}
      - ``{{ plc_versions[ref_name][category].version }}``, {{ plc_versions[ref_name][category].vendor_short }}

    {% else %}
      -

    {% endif %}
{% endfor %}
{% endif %}

{% endfor %}
{% endfor %}
