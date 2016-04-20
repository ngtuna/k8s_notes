#Labels and Selectors
Labels are key/value pairs that are attached to Kubernetes objects
##Motivation
Labels enable users to map their own organizational structures onto system objects in a loosely coupled fashion, without requiring clients to store these mappings.
##Label selectors
The label selector is the core grouping primitive in Kubernetes. Via a `label selector`, the client/user can identify a set of objects.
##Service and Replication Controller
The set of `pods` that a `service` targets is defined with a `label selector`. Similarly, the population of `pods` that a `replication controller` should manage is also defined with a `label selector`.

`Labels selectors` for both objects are defined in json or yaml files using maps, and only equality-based requirement selectors are supported:
