####################
K8s Production-Grade 
####################

Creating a Kubernetes cluster to test building clusters and hosting
cloud native applications using mostly open source on prem environments.

=========
Objective
=========

Build a K8s cluster using technologies available at work as a proof of concept
and reusable template to build from.

The idea is to ***keep the cluster as simple as possible while meeting production
grade requirements*** and scale to run sizable, but not extreme, machine learning
and big data compute applications.

Production means:

* Tne installation is secure
* The deployment is managed with a repeatable and recorded process
* Performance is predictable and consistent
* Updates and configuration changes can be safely applied
* Logging and monitoring is in place to detect and diagnose failures and
  resource shortages
* Service is “highly available enough” considering available resources,
  including constraints on money, physical space, power, etc...
* A recovery process is available, documented, and tested for use in the event
  of failures

What do these considerations mean then for Kubernetes components.

**Kubernetes components from a high availability perspective**

+--------------------------+------------------------------+------------------------------------------+-------------+
| Component                | Role                         | Effect of Loss                           | Recommended |
|                          | Role                         |                                          | Instances   |
+==========================+==============================+==========================================+=============+
| etcd                     | Maintains state of all       | Loss of storage is catastrophic. Loss of | odd #, 3+   |
|                          | Kubernetes objects           | majority means Kubernetes loses control  |             |
|                          |                              | plane, API Server depends on etcd,       |             |
|                          |                              | read-only API calls not requiring a      |             |
|                          |                              | quorum may work, and existing workloads  |             |
|                          |                              | may continue to run.                     |             |
+--------------------------+------------------------------+------------------------------------------+-------------+
| API Server               | provides API used internally | Unable to stop, start, update new pods,  | 2+          |
|                          | and externally               | services, replication controller.        |             |
|                          |                              | Scheduler and Controller Manager depend  |             |
|                          |                              | on API Server. Workloads continue if     |             |   
|                          |                              | they are not dependent on API calls      |             |
|                          |                              | (ie operators, custom controllers, ERDs) |             |
+--------------------------+------------------------------+------------------------------------------+-------------+
| kube-scheduler           | Places pods on nodes         | No pod placements, priority and          | 2+          |
|                          |                              | preemption                               |             |
+--------------------------+------------------------------+------------------------------------------+-------------+
| kube-controller-manager  | Runs many controllers        | Core control loops that regulate state   | 2+          |
|                          |                              | cease, in-tree cloud provider            |             |
|                          |                              | integration breaks.                      |             |
+--------------------------+------------------------------+------------------------------------------+-------------+
| cloud-controller-manager | Integration for out-of-tree  | Cloud provider integration breaks        | 1           |
| (CCM)                    | cloud providers              |                                          |             |
+--------------------------+------------------------------+------------------------------------------+-------------+
| Add-ons (e.g., DNS)      | Varies                       | Varies                                   | Depends on  |
|                          |                              |                                          | add-on, 2+  |
|                          |                              |                                          | for DNS     |
+--------------------------+------------------------------+------------------------------------------+-------------+

-----------------
Node Architecture
-----------------

* **etcd** - 3 or 5 nodes 8GB RAM and a 20GB. A five-node etcd cluster is a
  best practice if you can afford it. Why? Because you could engage in
  maintenance on one and still tolerate a failure
* **3+ hosts** - Run etcd, API server, scheduler, controller manager in a VM on
  three hosts. Application workload runs in a VM on each host. 


**Node Layout and Resources**

+--------------------+---------------+-----------------+-----+--------+--------+
| VM Name            | Role          | IP Address      | CPU | RAM    | DISK   |
+--------------------+---------------+-----------------+-----+-----------------+
| saw-k8s-haproxy-01 | Load Balancer | 192.168.100.150 | 2   | 4 GB   | 20 GB  |
+--------------------+---------------+-----------------+-----+-----------------+
| saw-k8s-m-01       | Master        | 192.168.100.151 | 2   | 8 GB   | 20  GB |
+--------------------+---------------+-----------------+-----+-----------------+
| saw-k8s-m-02       | Master        | 192.168.100.152 | 2   | 8 GB   | 20  GB |
+--------------------+---------------+-----------------+-----+-----------------+
| saw-k8s-m-03       | Master        | 192.168.100.153 | 2   | 8 GB   | 20  GB |
+--------------------+---------------+-----------------+-----+-----------------+
| saw-k8s-w-01       | Worker        | 192.168.100.154 | 2   | 8 GB   | 20  GB |
+--------------------+---------------+-----------------+-----+-----------------+
| saw-k8s-w-02       | Worker        | 192.168.100.155 | 2   | 8 GB   | 20  GB |
+--------------------+---------------+-----------------+-----+-----------------+
| saw-k8s-w-03       | Worker        | 192.168.100.156 | 2   | 8 GB   | 20  GB |
+--------------------+---------------+-----------------+-----+-----------------+
| saw-k8s-w-04       | Worker        | 192.168.100.157 | 2   | 8 GB   | 20  GB |
+--------------------+---------------+-----------------+-----+-----------------+
|                                                      | 16  | 60 GB  | 160 GB |
+------------------------------------------------------+-----+-----------------+

===================
Target Technologies
===================

* **VM hypervisor** - `Proxmox VE`_ for VM hosting this doesn't matter much
* **Configuration management** - Salt_ for IaaC to setup and maintain Kubernetes nodes programattically
* **VM OS** - CentOS_ for node operating systems to replicate using RHEL 8
* **K8s ingress controller** - HAProxy_ for high availability load balancing
* **K8s cluster creation**- kubeadm_ for bootstrapping K8s nodes 

.. _`Proxmox VE`: https://www.proxmox.com/en/ 
.. _Salt: https://saltproject.io
.. _CentOS: https://www.centos.org/download/
.. _HAProxy: http://www.haproxy.org

**Assets list**

1. CentOS boot ISO
2. HAProxy package


References:

* `How to make Kubernetes production grade anywhere <https://kubernetes.io/blog/2018/08/03/out-of-the-clouds-onto-the-ground-how-to-make-kubernetes-production-grade-anywhere/>`_

