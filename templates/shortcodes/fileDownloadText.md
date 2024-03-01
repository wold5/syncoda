{# V1 Generate downloadlink from linktext + filename. Uses page.version #}

{% set filepath = ["/files", page.extra.application_name, page.extra.application_version, filename] | join(sep="/") %}

- [ {{ linktext }} ]({{ filepath }}) \
<small>SHA256 {{ get_hash(path=filepath, sha_type=256, base64=false) }}</small>
