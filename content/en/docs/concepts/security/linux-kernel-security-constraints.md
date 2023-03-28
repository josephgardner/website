---
title: Linux kernel security constraints for Pods and containers
description: >
  Overview of Linux kernel security modules and constraints that you can use to
  harden your Pods and containers.
content_type: concept
weight: 100
---

<!-- overview -->

This page describes some of the security features that are built into the Linux
kernel that you can use in your Kubernetes workloads. To learn how to apply
these features to your Pods and containers, refer to
[Configure a Security Context for a Pod or Container](/docs/tasks/configure-pod-container/security-context/).
You should already be familiar with Linux and with the basics of Kubernetes
workloads.

<!-- body -->

## Run workloads without root privileges {#run-without-root}

When you deploy a workload in Kubernetes, use the Pod specification to restrict
that workload from running as the root user on the node. You can use the Pod
securityContext to define the specific Linux user and group for the processes in
the Pod, and explicitly restrict containers from running as root users. Setting
these values in the Pod manifest takes precedence over similar values in the
container image, which is especially useful if you're running images that you
don't own.

Configuring the kernel security features on this page provides fine-grained
control over the actions that processes in your cluster can take, but managing
these configurations can be challenging at scale. Running containers as
non-root, or in user namespaces if you need root privileges, helps to reduce the
chance that you'll need to enforce your configured kernel security capabilities.

## Security features in the Linux kernel {#linux-security-features}

Kubernetes lets you configure and use Linux kernel features to improve isolation
and harden your containerized workloads. Common features include the following:

* **Secure computing mode (seccomp)**: Filter a process' system calls
* **AppArmor**: Restrict the access privileges of individual programs
* **Security Enhanced Linux (SELinux)**: Assign security labels to objects for
  more manageable security policy enforcement

You use the `securityContext` field in your Pod specification to define the
constraints that apply to those processes. The security context also supports
other security settings, such as specific Linux capabilities or file access
permissions using UIDs and GIDs. To learn more, refer to
[Configure a Security Context for a Pod or Container](/docs/tasks/configure-pod-container/security-context/).

### seccomp

Some of your workloads might need privileges to perform specific actions as the
root user on your node's host machine. Linux uses *capabilities* to divide the
available privileges into categories, so that processes can get the privileges
required to perform specific actions without being granted all privileges. Each
capability has a set of system calls (syscalls) that a process can make. seccomp
lets you restrict these individual syscalls. <!--Copied from seccomp tutorial-->
It can be used to sandbox the privileges of a process, restricting the calls it
is able to make from userspace into the kernel.<!--End copy-->

In Kubernetes, you use a *container runtime* on each node to run your
containers. Example runtimes include CRI-O, Docker, or containerd. Each runtime
allows only a subset of Linux capabilities by default. You can further limit the
allowed syscalls individually by using a seccomp profile. Container runtimes
usually include a default seccomp profile. <!--Copied from seccomp tutorial-->
Kubernetes lets you automatically
apply seccomp profiles loaded onto a node to your Pods and containers.<!--End copy-->

{{<note>}}
Kubernetes also has the `allowPrivilegeEscalation` setting for Pods and
containers. When set to `false`, this prevents processes from gaining new
capabilities and restricts unprivileged users from changing the applied seccomp
profile to a more permissive profile.
{{</note>}}

To learn how to implement seccomp in Kubernetes, refer to
[Restrict a Container's Syscalls with seccomp](/docs/tutorials/security/seccomp/).

#### Considerations for seccomp {#seccomp-considerations}

seccomp is a low-level security configuration that you should only configure
yourself if you require fine-grained control over Linux syscalls. Using
seccomp, especially at scale, has the following risks:

* Configurations might break during application updates
* Attackers can still use allowed syscalls to exploit vulnerabilities
* Profile management for individual applications becomes challenging at scale

**Recommendation**: Use the default seccomp profile that's bundled with your
container runtime. If you need a more isolated environment, consider using a
sandbox, such as gVisor. Sandboxes solve the preceding risks with custom
seccomp profiles, but require more compute resources on your nodes and might
have compatibility issues with GPUs and other specialized hardware.

### Policy-based mandatory access control {#policy-based-mac}

You can use Linux policy-based mandatory access control (MAC) mechanisms, such
as AppArmor and SELinux, to harden your Kubernetes workloads.

#### AppArmor

#### SELinux

SELinux is a Linux kernel security module that lets you restrict the access
that a specific *subject*, such as a process, has to the files on your system.
You define security policies that apply to subjects that have specific SELinux
labels. When a process that has an SELinux label attempts to access a file, the
SELinux server checks whether that process' security policy allows the access
and makes an authorization decision.

In Kubernetes, you can set an SELinux label in your manifest securityContext
The specified labels are assigned to those processes. If you have configured
security policies that affect those labels, the host OS handles authorization.

To learn how to use SELinux in Kubernetes, refer to
[Assign SELinux labels to a container](/docs/tasks/configure-pod-container/security-context/#assign-selinux-labels-to-a-container).

#### Differences between AppArmor and SELinux {#apparmor-selinux-diff}

The operating system on your nodes usually includes one of either AppArmor or
SELinux. Both mechanisms provide similar types of protection, but have
differences such as the following:

* **Configuration**: AppArmor uses profiles to define access to resources.
  SELinux uses policies that apply to specific labels.
* **Policy application**: In AppArmor, you define resources using file paths.
  SELinux uses the index node (inode) of a resource to identify the resource.

### Summary of features {#summary}

The following table describes the use cases and scope of each security control.
You can use all of these controls together to build a more hardened system.

{{<table caption="Summary of Linux kernel security features">}}
| Security feature | Scope | Use case | How to use | Example |
| --- | --- | --- | --- |---|
| seccomp | Pod or container | Restrict individual kernel calls in the userspace.
{{</table>}}

## {{% heading "whatsnext" %}}