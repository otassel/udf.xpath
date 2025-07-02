# udf.xpath
XPath 1.0 BigQuery Python UDF

```sql
CREATE OR REPLACE FUNCTION `udf.xpath`(doc STRING, xpath STRING)
RETURNS ARRAY<STRING>
LANGUAGE python
OPTIONS(
    runtime_version="python-3.11",
    entry_point="evaluate",
    packages=['lxml==6.0.0'],
    container_memory='4Gi',
    container_cpu=1
)
AS r"""
from lxml import html, etree
from typing import List

def evaluate(doc: str, xpath: str) -> List[str]:
    items: List[str] = []

    try:
        if not doc.strip():
            return items  # Return empty array for empty document

        tree = html.fromstring(doc)
        elements = tree.xpath(xpath)

        for element in elements:
            if isinstance(element, str):
                items.append(element)
            else:
                items.append(html.tostring(element, encoding='unicode', method='html'))

    except (etree.ParserError, etree.XPathError):
        # Gracefully handle parsing or XPath errors by returning an empty array
        pass

    return items
""";
```
