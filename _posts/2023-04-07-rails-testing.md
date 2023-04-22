---
layout: post
---

# Rails testing

We usually use minitest since they are default Rails tool.
You can start learnig from guide <https://guides.rubyonrails.org/testing.html>

We use slow system test only for a few tests for success path flow. Instead of
system test, we use faster controller tests.
We use faster controller `ActionDispatch::IntegrationTest` test

You can find basic minitest assertions
<https://guides.rubyonrails.org/testing.html#available-assertions>
but for each [class](https://guides.rubyonrails.org/testing.html#a-brief-note-about-test-cases)
you can use other helpers and assertions like
[follow_redirect!](https://api.rubyonrails.org/v7.0.4.2/classes/ActionDispatch/Integration/RequestHelpers.html)

## Fixtures

<https://guides.rubyonrails.org/testing.html#the-low-down-on-fixtures>

In each fixture yml file we define `DEFAULT` that will contain defaults needed
to create valid item (for example presence validation on name and null false in
datable).
We create one fixture with the same name as model, for example
<https://github.com/trkin/rails_minitest/blob/main/test/fixtures/users.yml>
```
# test/fixtures/users.yml
# https://github.com/trkin/rails_minitest/blob/main/test/fixtures/users.yml
DEFAULTS: &DEFAULTS
  email: $LABEL@example.com
  encrypted_password: <%= User.new.send(:password_digest, 'password') %>
  confirmed_at: <%= Time.zone.now %>

user:
  <<: *DEFAULTS

unconfirmed:
  <<: *DEFAULTS
  confirmed_at:
```

The reason is that we do not need to think about the name when we a record. For
example when we need a user than `user = users(:user)`, when we need a comment
than we have it as `comment = comment(:comment)`
Those default records should represent common usage, eg `comment(:comment)`
belons to `posts(:post)` which belongs to `users(:user)`.
