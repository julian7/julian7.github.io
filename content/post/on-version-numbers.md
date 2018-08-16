---
title: "On Version Numbers"
date: 2018-08-16T12:37:47+02:00
draft: false
tags:
  - semver
  - software architecture
---
The other day I got a question from a principal software architect: can they open a single Pull Request for multiple git repositories?

He drew two scenarios:

1. When they have a fundamental change in their gradle configs, they have to make the change in every repository
2. If there is an API change, they have to modify the code at multiple places at the same time.

This is a very large code base coming from ClearCase, and it compiles for several hours. It provides several executables, which should be run on different machines. A real legacy system.

They have already done heroic efforts to separate components, but interdependency is still very high.

## The bad news

Obviously, it is not possible what they want. Or, to be more precise, it is not possible what they **think** they want.

The first problem is always a tough situation, and we have to swallow the bitter pill. However, for the second problem, I think what they need is proper versioning, and better interoperability. They won’t be able to save on the number of merge requests, for sure. However, leveraging good practices with a proper versioning system can turn tides.

## Semantic versioning

Semantic Versioning is easy: there are tree to five identifiers one can put together:

- MAJOR version: increment when there are incompatible API changes
- MINOR version: increment when adding functionality in a backwards-compatible manner
- PATCH version: increment when fixing bugs in a backwards-compatible manner
- Pre-release: optional, denoting a pre-release version
- Build metadata: optional, add metadata version number

Example: 2.0.0-1.3.9+20130313144700: it can be a pre-release of 2.0.0, based on 1.3.9, with bundled metadata released on March 13, 2013, at 2:47 PM UTC.

It is well documented at [semver.org](http://semver.org/).

However, by learning from Lean principles (this time, one specific principle: limit work in progress), instead of jumping into implementing incompatible changes, take a less invasive method.

## API versioning and deprecations

If there is a need for change in the API, one should add a new one with extended functionality, instead of braking compatibility. Then, the old API can be re-implemented to call the new one, and markig it deprecated:

```ruby
## send announcement to users
def send_colored_announcement_to_users(users, message, color)
  users.each do |user|
    user.send_announcement(message, color)
  end
end

## send announcement to users
### deprecated
def send_announcement_to_users(users, message)
  deprecated_call('send_color_announcement_to_users')
  send_color_announcement_to_users(users, message, 'black')
end
```

In the example, we use `deprecated_call` method to log the fact of using a deprecated API, and its documentation also flagged as `### deprecated`.

Versioning opens a lot of questions: whether add version numbers to the API call’s name, or use a (slightly) different name? However, one thing is common: when an old API endpoint is superseded, deprecate it.

Deprecations can be consumed at multiple levels: at static code analysis (which can feed upon inline documentation’s deprecation notices), or at testing, by logging deprecation warnings.

## Identifying and deleting dead code

If we do this faithfully, rolling out a new major version can be just the removal of deprecated code. Nothing else make the app run faster, become smaller, than removing code.

Also, this makes a new major release being a non-event. It might sound a bit flat, but in operations, this is the Holy Grail.

## "I know it’s a steam engine... but what makes it move?"

This all work nice in theory, but there are a lot of missing pieces you have to put in place:

- Tracking outdated versions: if a dependency upgrades, how can we say it’s safe (let alone possible) to upgrade?
- Enforcing upgrades: how can we get API consumers to upgrade?
- Is there a way to use multiple versions in a single build output? If there are multiple version requirements for different parts of the same executable, odds are only one can be built in.
- How to set up a development strategy for subcomponents? When there are multiple consumers, it’s often not enough to directly modify the code (let’s call it tactical changes), but it might require a higher goal, a strategy.
- What to collect in deprecation measurement? With multiple modules, it’s often a good idea to collect module names, deprecated and suggested call names, and at least the version name, from when the deprecation took place, to be collected.

Program management has to answer these questions, before sailing out.

Obviously, it doesn’t save us a lot of PRs being opened, but it makes each PR being independent of each other, and also uncovers a lot of yet invisible work of interdependence.
