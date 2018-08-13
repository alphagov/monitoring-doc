# 5. Give users access to Prometheus and Alertmanager

Date: 2018-08-08

## Status

Pending.

## Context

Our users need an easier way to write alerts, they currently have no
easy way to test queries before writing alerts. Our Prometheus
service requires authentication which locks our user out of it.

## Decision

We will give our users access to Prometheus so they can use its
expression browser to test queries when writing alerts. We will
do this by using IP Whitelisting instead of authentication and
only allowing our office IP addresses.

We identified that the user needs to be authentication to platform
in order to learn behaviors of different team's and know who is
performing what action. There was a number of different routes that
we could have taken to achieve this. One possible route that we
considered was using Oauth2 authentication. This would enable users
to authenticate themselves to the platform with their Google account.

We did not choose to go with this option this time for expediency.
The idea behind this was to try to deliver the fastest value as
possible to the user. This method enables us to learn more about the user's
usage pattern. We do intend to add authentication but this will be done at
a later date.

## Consequences

Anyone in the office can access Prometheus and Alertmanager. This means that
they can use expression browser in order to dynamically test and create queries
with fast feedback loop. We are not able to track our user which has some minor
negative consequences.

