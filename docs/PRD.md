# Bootc Operator PRD

## Motivation

[bootc] is a CNCF project that allows one to build and deploy image-based Linux
systems using the same ecosystem of tools as application containers. This makes
it a great fit for K8s nodes. Additionally, bootc is distro-agnostic, which
means that it has potential for broad usage in the K8s ecosystem as a host OS.

The missing piece is an operator which bridges bootc and K8s. This is the Bootc
Operator. Its goal is to tightly integrate with bootc and to surface its state
and capabilities to the K8s control plane to make it easier to monitor and
manage bootc nodes in the cluster.

## User Stories

### Installing

- A user wants to install the bootc operator.
    - It should be a single `kubectl create -f` to get the operator installed.
    - Once installed, the operator should do nothing by default.

### Onboarding

- A user wants to have their bootc nodes managed by the bootc operator.
    - They should be able to register their nodes with the operator, e.g. by
      creating one or multiple CRDs which describes the desired state of their
      nodes.
    - The bootc operator should start reconciling desired vs actual based on the
      CRDs and bootc status.
    - It should be possible for users to only ask the operator to manage a
      subset of their nodes.
    - It should be possible to have different pools of nodes with different
      desired states.
- A user wants their bootc nodes to use an image protected by pull secrets.
    - They should be able to provide to the operator a pull secret to use.
    - The operator should take care of propagating this down to the nodes.

### Managing

- A user wants their nodes to be updated by digest (manual).
    - They specify the digest they want to pin on.
    - The operator rolls out the update.
- A user wants their nodes to be updated by tag (manual).
    - They specify the tag they want to follow.
    - The operator resolves the tag to a digest _once_ and then rolls out the update as usual.
    - The operator monitors updates to the tag. If one exists, it updates a
      status in a CRD to surface this.
    - The user can easily tell the operator to update to the resolved digest.
- A user wants their nodes to automatically remain up to date (auto).
    - They specify the tag they want to follow.
    - They specify they want automatic updates. Optionally, they specify
      maintenance windows.
    - The operator monitors updates to the tag.
    - The operator automatically rolls out updates.
- A user wants to monitor the rollout of an update
    - They should be able to easily follow the state of the update as it rolls
      out through the cluster.
    - They should be able to tell if something has gone wrong, and the error
      message should be clearly surfaced.
    - The operator should stage updates on target nodes before initiating
      a rollout.
    - The operator should not continue with the rollout if a node reports an
      error.
- A user wants to limit the number of nodes to update in parallel.
    - They can do this by editing a CRD.
- A user wants to stop a rollout 
    - They can do this by editing a CRD.
    - They should be able to later resume the rollout.
    - They should be able to cancel the rollout. Updated nodes should roll back
      to the previous version.
- A user wants to make use of bootc-specific features, for example soft-reboot
  or any future relevant features (such as dynamic config overlays).
    - The operator should have first class support for bootc features.

### Developing

- A developer wants to test their changes to the operator.
    - It should be possible to run e2e tests locally using virtualization.

## Relationship to the Machine Config Operator

The [MCO] is an operator capable of managing bootc hosts (such as RHEL CoreOS)
but it is tied to [OpenShift]. It is overall quite powerful but complex. It also
implements features rendered less valuable by the bootc model (such as day-2
support for [Ignition] configs, and package layering via [rpm-ostree]).

The Bootc Operator tries to re-imagine what a "bootc-native" operator not tied
to any particular K8s distribution looks like. That said, a primary goal is to
support being leveraged by existing distributions where it may be driven by a
more powerful operator (such as the MCO on OpenShift).

## Relationship to Cluster API

The two are complementary. [Cluster API] provides APIs for provisioning
and deleting nodes on various infrastructures, but otherwise treats the OS
running on those nodes as a black box. The bootc operator knows nothing about
provisioning nodes, but knows how to manage the OS once they've joined the
cluster.

## Relationship to kured

[Kured] is a Kubernetes daemon which handles update rollout and reboot
coordination. It is minimal and well-scoped. However, for the Bootc Operator, we
want to ensure we take full advantage of the features afforded by an image-based
system like bootc as part of rollout strategies. Update staging and OS rollbacks
are notable examples. While there is potential for collaboration and code
sharing, we are focused for now on exploring the problem space when tightening
the scope to bootc hosts.

[bootc]: https://github.com/bootc-dev/bootc
[Cluster API]: https://github.com/kubernetes-sigs/cluster-api
[Ignition]: https://coreos.github.io/ignition/
[Kured]: https://kured.dev/
[MCO]: https://github.com/openshift/machine-config-operator
[OpenShift]: https://www.redhat.com/en/technologies/cloud-computing/openshift
[rpm-ostree]: https://coreos.github.io/rpm-ostree/
