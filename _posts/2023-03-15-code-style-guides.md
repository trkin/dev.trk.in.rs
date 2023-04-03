---
layout: post
---

For Ruby we are following

* <https://github.com/thoughtbot/guides/tree/main/ruby>
* <https://github.com/Shopify/ruby-style-guide>
* <https://github.com/airbnb/ruby>

For automated way of controlling code styles we use
[Standard](https://github.com/testdouble/standard) which uses Rubocop underneath
and it is installed with those commands:
```
bundle add standard

cat > .rubocop.yml <<HERE_DOC
require: standard

inherit_gem:
  standard: config/base.yml
HERE_DOC

cat > .standard.yml <<HERE_DOC
ignore:
  - "**/*":
    # put comma after each line {a:1,}
    - Style/TrailingCommaInHashLiteral
HERE_DOC
```

## Erb lint

TODO: https://github.com/Shopify/erb-lint

# Text syntax on buttons and messages

* labels on buttons, titles,...  should be upper case with no period: `This Job
Is Paused`
* sentences on error messages, flash messages...  should have period at the end:
`Job has been copied successfully.`
