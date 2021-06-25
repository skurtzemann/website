---
reviewers:
- chenopis
title: (WIP) L'API Kubernetes
content_type: concept
weight: 30
description: >
  The Kubernetes API lets you query and manipulate the state of objects in Kubernetes.
  The core of Kubernetes' control plane is the API server and the HTTP API that it exposes. Users, the different parts of your cluster, and external components all communicate with one another through the API server.
card:
  name: concepts
  weight: 30
---

<!-- overview -->

Le coeur du {{< glossary_tooltip text="plan de contrôle" term_id="control-plane" >}} 
de Kubernetes est le {{< glossary_tooltip text=" serveur d'API" term_id="kube-apiserver" >}}. 
Le serveur d'API expose une API HTTP qui permet aux utilisateurs finaux, à différents éléments de votre cluster, 
et à des composants externes de communiquer ensemble.

L'API Kubernetes vous permet d'interroger et de manipuler l'état des objets dans Kubernetes
(par exemple: les Pods, les Namespaces, les ConfigMaps, et les Events).

La plupart des opérations sont réalisées à l'aide de l'interface en ligne de commande 
[kubectl](/fr/docs/reference/kubectl/overview/) 
ou tous autres outils en ligne de commande, tel que 
[kubeadm](/fr/docs/reference/setup-tools/kubeadm/), qui utilise également l'API.
Cependant, il est également possible d'accéder directement à l'API via des appels (de type) REST.

<!-- Most operations can be performed through the
[kubectl](/docs/reference/kubectl/overview/) command-line interface or other
command-line tools, such as
[kubeadm](/docs/reference/setup-tools/kubeadm/), which in turn use the
API. However, you can also access the API directly using REST calls. -->

Si vous souhaitez écrire une application utilisant l'API Kubernetes, privilégiez alors l'utilisation de l'une des [librairies clientes](/docs/reference/using-api/client-libraries/).

<!-- body -->

## Spécification OpenAPI {#api-specification}

Tous les détails de l'API sont documentés selon la norme [OpenAPI](https://www.openapis.org/). 

L'API Kubernetes fournie une spécification OpenAPI sur l'URL `/openapi/v2`.
Vous pouvez interroger le format de réponse à l'aide des en-têtes de requêtes comme ...

<!-- The Kubernetes API server serves an OpenAPI spec via the `/openapi/v2` endpoint.
You can request the response format using request headers as follows: -->

<table>
  <caption style="display:none">Valeurs possibles des en-têtes de requêtes des requêtes OpenAPI v2</caption>
  <thead>
     <tr>
        <th>En-têtes</th>
        <th style="min-width: 50%;">Valeurs possibles</th>
        <th>Notes</th>
     </tr>
  </thead>
  <tbody>
     <tr>
        <td><code>Accept-Encoding</code></td>
        <td><code>gzip</code></td>
        <td><em>not supplying this header is also acceptable</em></td>
     </tr>
     <tr>
        <td rowspan="3"><code>Accept</code></td>
        <td><code>application/com.github.proto-openapi.spec.v2@v1.0+protobuf</code></td>
        <td><em>pour une utilisation intra-cluster</em></td>
     </tr>
     <tr>
        <td><code>application/json</code></td>
        <td><em>par défaut</em></td>
     </tr>
     <tr>
        <td><code>*</code></td>
        <td><em>serves </em><code>application/json</code></td>
     </tr>
  </tbody>
</table>

Kubernetes implements an alternative Protobuf based serialization format that
is primarily intended for intra-cluster communication. For more information
about this format, see the [Kubernetes Protobuf serialization](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/protobuf.md) design proposal and the
Interface Definition Language (IDL) files for each schema located in the Go
packages that define the API objects.

## Persistance

Kubernetes stocke l'état sérialisé des objets en écrivant ces dernier dans
{{< glossary_tooltip term_id="etcd" >}}.

## API groups and versioning

To make it easier to eliminate fields or restructure resource representations,
Kubernetes supports multiple API versions, each at a different API path, such
as `/api/v1` or `/apis/rbac.authorization.k8s.io/v1alpha1`.

Versioning is done at the API level rather than at the resource or field level
to ensure that the API presents a clear, consistent view of system resources
and behavior, and to enable controlling access to end-of-life and/or
experimental APIs.

To make it easier to evolve and to extend its API, Kubernetes implements
[API groups](/docs/reference/using-api/#api-groups) that can be
[enabled or disabled](/docs/reference/using-api/#enabling-or-disabling).

API resources are distinguished by their API group, resource type, namespace
(for namespaced resources), and name. The API server handles the conversion between
API versions transparently: all the different versions are actually representations
of the same persisted data. The API server may serve the same underlying data
through multiple API versions.

For example, suppose there are two API versions, `v1` and `v1beta1`, for the same
resource. If you originally created an object using the `v1beta1` version of its
API, you can later read, update, or delete that object
using either the `v1beta1` or the `v1` API version.

### API changes

Any system that is successful needs to grow and change as new use cases emerge or existing ones change.
Therefore, Kubernetes has designed the Kubernetes API to continuously change and grow.
The Kubernetes project aims to _not_ break compatibility with existing clients, and to maintain that
compatibility for a length of time so that other projects have an opportunity to adapt.

In general, new API resources and new resource fields can be added often and frequently.
Elimination of resources or fields requires following the
[API deprecation policy](/docs/reference/using-api/deprecation-policy/).

Kubernetes makes a strong commitment to maintain compatibility for official Kubernetes APIs
once they reach general availability (GA), typically at API version `v1`. Additionally,
Kubernetes keeps compatibility even for _beta_ API versions wherever feasible:
if you adopt a beta API you can continue to interact with your cluster using that API,
even after the feature goes stable.

{{< note >}}
Although Kubernetes also aims to maintain compatibility for _alpha_ APIs versions, in some
circumstances this is not possible. If you use any alpha API versions, check the release notes
for Kubernetes when upgrading your cluster, in case the API did change.
{{< /note >}}

Refer to [API versions reference](/docs/reference/using-api/#api-versioning)
for more details on the API version level definitions.



## API Extension

The Kubernetes API can be extended in one of two ways:

1. [Custom resources](/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
   let you declaratively define how the API server should provide your chosen resource API.
1. You can also extend the Kubernetes API by implementing an
   [aggregation layer](/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/).

## {{% heading "whatsnext" %}}

- Learn how to extend the Kubernetes API by adding your own
  [CustomResourceDefinition](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/).
- [Controlling Access To The Kubernetes API](/docs/concepts/security/controlling-access/) describes
  how the cluster manages authentication and authorization for API access.
- Learn about API endpoints, resource types and samples by reading
  [API Reference](/docs/reference/kubernetes-api/).
- Learn about what constitutes a compatible change, and how to change the API, from
  [API changes](https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#readme).