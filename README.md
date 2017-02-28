pyjq: Binding for jq JSON Processor
===================================

[![Build Status](https://travis-ci.org/doloopwhile/pyjq.svg?branch=travis-ci)](https://travis-ci.org/doloopwhile/pyjq)

pyjq is a Python bindings for jq (<http://stedolan.github.io/jq/>).

> jq is like sed for JSON data – you can use it to slice and filter and
> map and transform structured data with the same ease that sed, awk,
> grep and friends let you play with text.
>
> <http://stedolan.github.io/jq/>

You can seamlessly call jq script (like regular expression) and process
plain python data structure.

For your information, <https://pypi.python.org/pypi/jq> is a also jq
bindings but different and incompatible with pyjq.

Example
-------

    >>> data = dict(
    ...     parameters= [
    ...         dict(name="PKG_TAG_NAME", value="trunk"),
    ...         dict(name="GIT_COMMIT", value="master"),
    ...         dict(name="TRIGGERED_JOB", value="trunk-buildall")
    ...     ],
    ...     id="2013-12-27_00-09-37",
    ...     changeSet=dict(items=[], kind="git"),
    ... )
    >>> import pyjq
    >>> pyjq.first('.parameters[] | {"param_name": .name, "param_type":.type}', data)
    {'param_type': None, 'param_name': 'PKG_TAG_NAME'}

Install
-------

You can install from PyPI by usual way.

    pip install pyjq

API
---

For jq script, [see its manual](http://stedolan.github.io/jq/manual/).

Only four APIs are provided:

- `all`
- `first`
- `one`
- `compile`

`all` transforms a value by JSON script and returns all results as a list.

```
>>> value = {"user":"stedolan","titles":["JQ Primer", "More JQ"]}
>>> pyjq.all('{user, title: .titles[]}', value)
[{'user': 'stedolan', 'title': 'JQ Primer'}, {'user': 'stedolan', 'title': 'More JQ'}]
```

`all` takes an optional argument `vars`.
`vars` is a dictonary of predefined variables for `script`.
The values in `vars` are available in the `script` as a `$key`.
That is, `vars` works like `--arg` option and `--argjson` option of jq command.
```
>>> pyjq.all('{user, title: .titles[]} | select(.title == $title)', value, vars={"title": "More JQ"})
[{'user': 'stedolan', 'title': 'More JQ'}]
```

`all` takes an optional argument `url`.
If `url` is given, the subject of transformation is got from the `url`.

```
>> pyjq.all(".[] | .login", url="https://api.github.com/repos/stedolan/jq/contributors") # get all contributors of jq
['nicowilliams', 'stedolan', 'dtolnay', ...
```

Additionally, `all` takes an optional argument `opener`.
The default `opener` will simply download contents by `urllib.request.urlopen` and decode by `json.decode`.
However, you can customize this behavior using custom `opener`.

`first` is similar to `all` but it only returns the first result of transformation.

```
>>> value = {"user":"stedolan","titles":["JQ Primer", "More JQ"]}
>>> pyjq.first('{user, title: .titles[]}', value)
[{'user': 'stedolan', 'title': 'JQ Primer'}]
```

`first` returns `default` when there are no results.

```
>>> value = {"user":"stedolan","titles":["JQ Primer", "More JQ"]}
>>> pyjq.first('.titles[] | select(test("e"))', value) # The first title which is contains "e"
'JQ Primer'
```

`first` returns the first result of transformation. It returns `default` when there are no results.

```
>>> value = {"user":"stedolan","titles":["JQ Primer", "More JQ"]}
>>> pyjq.first('.titles[] | select(test("T"))', value, "Third JS") # The first title which is contains "T"
'Third JS'
```

`one` also returns the first result of transformation but raises an Exception if there are no results.

```
>>> value = {"user":"stedolan","titles":["JQ Primer", "More JQ"]}
>>> pyjq.one('.titles[] | select(test("T"))', value)
IndexError: Result of jq is empty
```

`compile` returns a pyjq Script object with the compiled jq filter.  It can be used to validate a filter.
...
>>>  pyjq.compile(filter)
<_pyjq.Script object at 0x7fce4cedb4d0>
...

`compile` will raise an Exception if the filter is not valid for jq:
...
>>> filter = '.titles{} | select(test("T"))'
>>> pyjq.compile(filter)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib64/python2.7/site-packages/pyjq.py", line 15, in compile
    return _pyjq.Script(script.encode('utf-8'), vars=vars)
  File "_pyjq.pyx", line 164, in _pyjq.Script.__init__ (_pyjq.c:2413)
ValueError: jq: error: syntax error, unexpected '{', expecting $end (Unix shell quoting issues?) at <top-level>, line 1:
.titles{} | select(test("T"))
jq: 1 compile error
...

Limitation
----------

jq is a JSON Processor. Therefore pyjq is able to process only
"JSON compatible" data (object made only from str, int, float, list, dict).

Q&A
---

### How can I process json string got from API by pyjq?

You should apply `json.loads` in the standard library before pass to pyjq.

License
-------

Copyright (c) 2014 OMOTO Kenji. Released under the MIT license. See
LICENSE for details.

Changes
-------

### 2.0.0

- Semantic versioning.
- Bundle source codes of jq and oniguruma.
- Supported Python 3.5.
- Dropped support for Python 3.2.
- Aeded `all` method.

### 1.0

- First release.
