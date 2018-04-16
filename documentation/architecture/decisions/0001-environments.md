# 1. environments

Date: 2018-04-16

## Status

Pending

## Context

We want to have separate environments for running our software at different stages of release.
This will be used to provide a location where changes can be tested without impacting
production work and our users.

## Decision

We have decided to have two separate environments, staging and production. The staging
environment will be where the tools team and other people working on monitoring
and metrics itself will test their changes. The production environment will run
all of our users monitoring and metrics and poll each of their environments.

## Consequences

Keeping the two environment separate reduces the chance of a core change impacting
our users and will allow us to test aspects of our system such as handling load,
fail over testing and new, possibly behaviour changing, version upgrades. Users will
test their prometheus related changes in our production environment and will not have access
to the staging environment.
