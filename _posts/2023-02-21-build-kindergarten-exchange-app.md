---
layout: post
---

As a practice task in our [first intership]({% link _posts/2023-01-04-internship.md %})
we are rebuilding a webapp that will help parents organize themselves and find
free places in kindergartens that are closer to them. When there are no
vacancies the only way to move the kids to another kindergarten is to find
someone else who also wants to move. Existing app source is
[https://github.com/trkin/premesti.se](https://github.com/trkin/premesti.se) and
we are rebuilding to use latest tools like [Turbo](https://turbo.hotwired.dev/)

Main tools that we will use:
* [Ruby on rails]({% link _posts/2023-01-24-learning-materials-for-ruby-and-rails.md %})
  as a server side engine
* Github projects for organizing tasks
* Neo4j graph database to find matches in rotations
* Figma for designs
* Tailwind for styles
* Jekyll for blog
* Turbo and Stimulus for registration and chat

All practicioners should choose one area in which to become a master so they can
teach other teammates.

Source code is on github https://github.com/trkin/kindergarten-exchange

Main features should be:
* user registration using various ways: phone number, email, social buttons
* user select `move` ie kindergarten in which they are interested in
* user can share their moves on social networks
* graph engine that calculates matches for each new `move`
* for each match a new `chat` is created and notifications are sent
* admin pages to invite users to register, to send promotional emails and users
  can unsubscribe to them

Advanced features:
* enable payment so moves that includes at least one premium move can chat
  immediatelly, and moves that are all free are waiting one week for
  notification and chat (send email to buy premium or to wait).
  Premium can be obtained by sharing on social networks. Use other features for
  premium like: free sms notification
* create mobile apps for Android and iOS
* create Viber app for registration

App should be using I18n transations and covered with tests.
