---
layout: post
---

For Ruby we are following

* <https://github.com/thoughtbot/guides/tree/main/ruby>
* <https://github.com/Shopify/ruby-style-guide>
* <https://github.com/airbnb/ruby>

For automated way of controlling code styles we use
[Standard](https://github.com/testdouble/standard) which uses Rubocop underneath
and it is installed with `bundle add standard` and configured:
```
# .rubocop.yml
require: standard

inherit_gem:
  standard: config/base.yml
```

# Text syntax on buttons and messages

* labels on buttons, titles,...  should be upper case with no period: `This Job
Is Paused`
* sentences on error messages, flash messages...  should have period at the end:
`Job has been copied successfully.`