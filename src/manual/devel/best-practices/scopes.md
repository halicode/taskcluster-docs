---
title: Scopes and Roles
order: 10
---

## When To Require A Role

When verifying the scopes for an API call or some other operation, the default behavior should be to check for appropriately-parameterized scopes.
These answer the question, "Can the caller do X".

However, when the operation modifies an object which will later perform actions using a role, then it is appropriate to require that the caller possess that role.
For example, hooks trigger tasks using role `hook-id:<hookGroupId>/<hookId>`, so the API calls to create and modify the hook require `assume:hook-id:<hookGroupId>/<hookId>` directly.
It is not enough simply to possess the role's extended scopes -- the caller must possess the assumed role itself.

## Encoding Information In Roles

Scopes should only ever be used to evaluate scope satisfaction; never pattern match scopes to try to extract information from them.

A common example of this error is in trying to determine a user's identity based on their credentials.
Since the `taskcluster-login` service helpfully adds scopes like `assume:mozilla-user:jrainer@ranierfamily.net`, it is tempting to look for a scope matching that pattern and extract the email address.

This has a few awkward failure modes, though.
Administrative users may have multiple matching scopes, or even `assume:mozilla-user:*`.
Even if those administrative users should avoid using your service with such powerful credentials, it's easy to do accidentally and incautious code may assume the user is named `*`.
Other credentials may have no matching scope, but still posess the scopes to authorize the bearer to perform an operation.
Basically, scopes are not a great way to communicate this information.

The appropriate way to determine a user's identity (as described in [Third Party Integration](/manual/integrations/apis/3rdparty)) is to find an email from some less trustworthy source such as the clientId, and then *verify* that email against the scopes, by asking "is `assume:mozilla-user:<email>` satisfied?"