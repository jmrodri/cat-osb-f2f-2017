# Catalog Highlights
**Attendees**
  * Red Hat
    * Paul Morie
    * Erik Nelson
    * Jesus Rodriguez
    * Jessica Forrester
    * Matthew Staebler
    * Jay Boyd
    * Jeff Peeler
  * Pivotal
    * Alex Ley
    * Matt McNeeney
  * Google
    * Ville Aikas
    * Scott Nichols
    * Michael Kibbe
    * Martin Gannholm
  * IBM
    * Doug Davis
    * Morgan Bauer
  * Microsoft
    * Kent Rancourt
  * Manifold
    * James Bowes
    * Gary Poster
  * Dell EMC
    * Mike Rhodes
  * SAP
    * Florian Muller

**Async Bind**

Alpha implementation has already been merged to master. It can be enabled via
`--feature-gates AsyncBindingOperations=true` as an argument to the service-catalog
binary. It's also available in the helm chart as `asyncBindingOperationsEnabled`.

More on this during the OSB meetings;
[OSB proposal is in "validation through implementation phase"](https://github.com/openservicebrokerapi/servicebroker/pull/334)

**Better kubectl UX**

Not helpful getting back opaque ServiceClass and ServicePlan IDs. Custom cols
can be specified, but that's not a great experience. Two options moving forwards:

- OPENAPI support, annotations on types will expose via an openapi doc default cols.
There is a PR open for this.
- k8s machinery, can implement interfaces in storage layer that tell kubectl the default
columns. Possibly a better solution, but in likely won't be available until 1.10.

Once the OPENAPI PR gets merged, kubectl requires a flag to look at it. It's
an inconveniently long flag, so there is a desire to be able to configure this
once via ENV_VAR or config option. PR for config forthcoming.

**Bind credential remapping**

If binding gives back keys foo bar baz, bunch of services give back different
versions of the same thing, want to translate how they end up in the secret.

Template, or chart that expects DB_HOST, mysql gives MYSQL_HOST. Need to map
MYSQL_HOST -> DB_HOST.

**Code health and Docs**

Bunch of conversation around code health, and potentially of more interest
to Ansible Broker folks, better documentation in the project. Doc wishlist:

```
Wanted docs:
* k8s concepts (links)
* design docs
* why servicecatalog?
* API docs
* API concepts
* More details in/available from walkthrough
* Userguide/tasks? (relist), stuff gotten from types.go
* Integratino with particular brokers?
* What is UPS broker
```

**RBAC controls**

Controller needs various permutations of checks to confirm a user can
perform a given action. Desire is to allow a cluster admin to say: sally can
use this set of SCs and plans, Bill can do everything, Jane can only do X or Y.

Involves executing SAR (Subject Access Reviews) to confirm users can `$VERB`
some catalog resource. Possible to introduce additional verbs beyond the normal
CRUD, so something like `provision` would actually be valid.

Of interesting note, it is quite difficult to implement something called
`AclFilteredList` within k8s, which is basically querying for "what is the list
of things that I have access to?".
Long conversation about why this is true, and the various ways role based sets
of privileges could be achieved.

Hoping to offer something like a kubectl plugin that can list objects that
a user has access to, although there is some hesitation around implementing
plugins at all, since this may be reworked in the future. AI to investigate
what the story is with plugins upstream.

**Namespaced Resources**

Use-case is to find a 3rd party broker and register that broker into your own
namespace. From there, you can see the cluster available class/plans, along
with your own namespace scoped ones.

Would require namespaced scoped broker.

Agreed:
* There is a need for namespace scoped brokers, which implies namespace scoped classes
and plans as well.
* The objects that are available must be able to be filtered in some way, possibly
based on role access.

# OSB Highlights

**Instance Updates**

What happens when an update fails? Spec says nothing about failures.
Agreed that if an update fails, instance should be in the state that it was
before the upgrade request was made. Going to need error codes to more
granularly describe exactly what occurred so systems can respond intelligently.

There is a desire for v3 to have some kind of rollback if an update fails.

Separately, right now, spec only allows for specifying whether or not
an instance can have its plan changed. There is no visibility into what
transitions are available. AI to propose describing valid transition matrix
for plans (APB spec can currently do this, broker has the data as well).

**Metrics, Health, Logging - Problem Discovery**

```
* Service Instance Health
  - Aggregated view of all my service instances
  - Red/green is a good start, not the end goal, need more
  - There is a difference between metrics and health
* Broker health
  - Should the platform ping the broker?
  - Is ping sufficient?
  - Proactive health vs error rate detection
  - Ping doesn't tell the whole story
```

Started to discuss the idea of broker actions (extension mechanism to the spec),
that allows a broker to implement enhanced behaviors (like metrics, health, logging).
Leads into...

**Generic Actions**

There is a requirement for a formal extension mechanism to allow brokers to
arbitrarily extend the spec. The platform needs to be able to discover these
extensions to understand what a given broker is capable of.

[Proposal](https://github.com/openservicebrokerapi/servicebroker/issues/114)

A lot of conversation working through how this would work. Sounding like those
interested will break off into smaller working group and distill the details
back into the proposal. Examples include things like backup/restore, logging/metrics.

**Orphan Mitigation**

Some complaints around the language of the spec being ambiguous. AI to
remove "orphan mitigation" section and the term "orphan", and just
talk about when it's expected to "clean up" for each failed op.

**Swagger Doc & JSON Schema**

Issue: Two docs, the spec and a swagger doc vs a single doc.

Conclusion:
Moving to 2 docs, machine readable swagger and human readable spec. Spec takes
precedence. Any PR that changes the spec must also update swagger.

Also a desire for endpoint that returns the entire JSON schema instead of
embedding it in the catalog. would allow for reference to common bits rather
than a lot of duplication. Likely to help things like UIs w/ payload sizes.
Proposal incoming. No separate endpoint, revisit in v3.

**OSB Validation Toolchain**

Microsoft has OSB validation tool interested in donating.

https://github.com/Haishi2016/osb-checker

Conversation about where this will live, and how it can be made better.
Opportunity to help here.

It was stated that the desire was to transfer ownership to the
openservicebrokerapi github organization. Whether this goes into the mainline
spec repo was still up for debate.

**Auth workflows**

Mostly considering how to add MTLS (mutual TLS, not multiplexed TLS) language
to the spec.  Possibly can help with feedback re: opaque bearer auth, which
is already in use within OpenShift.

**2.14 Release Planning**

GET and Async Bindings goals for v2.14, targeting end of Jan 2018.
Opaque Bearer Token auth stretch for v2.14

AI: we need to update the async binding, get and token auth PRs with comments
about how well the implement went based on the API perspective.

**v3 Goals**

* Introducing idea of Spec / Status where brokers can indicate supported version.
* Push based watch streams! Removes need for last_operation.
* Need to think about how v2 and v3 brokers will live side by side.
* Proposal to remove synchronous calls.
* Version negotiation

**Service Meshes**

Ex: App running in CloudFoundry, service (DB) running in kube. Want to bind
to that service. What's involved in making sure we can reach/talk to each other,
and authenticate the parties? How is istio involved? Issues with separate
istio control planes.
