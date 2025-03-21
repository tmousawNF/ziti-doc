---
sidebar_label: Session Diagram
---

# Sessions and Connections Sequence Diagram

OpenZiti has a number of different connection and session types.  It is important to understand the different scopes and uses of these connections in working with the project, developing, operating, and most importantly, troubleshooting.

[![](https://mermaid.ink/img/pako:eNqdVDtrwzAQ_iuHpgTSoR49BEKSoRRKaDoVLUK6pKLyyZXPLSXkv1dS4hLnUZJOxqf77nv4rI3Q3qAoRYMfLZLGmVXroCpJbNkhvFq2sMSmsZ4aUGRg6olQc3qXJKlWga22tSKGqbMYH5O6dlar1NI_X84e4b5fmps1wrNvGcPx0RPylw_viZCDdw7DZWhxSnRUOhAV_YTPNC13dPNh4RRhLHKWCXfj8RkJJUwWD10gklIgHaRIkKsRPeO3kfWMXwdFMp3djL7R85VG8-g9JwzqKK-JSVuNw1t93zqp-Nekg1RmitUuDRi8TBfDRHC6zpllF1UJse3gZ8i6ckJ5uc6iijOoCxqS7GH_w_SSK_OQv8LIhF3bnuU0_B4udtugW8tiJCoMlbIm3gwbSQBS8BtWKEVcBmFwpVrHUkjaxlbVsl9-kxYlhxZHoq2N4u4iEeVKuea3OjeWfdgXtz_UjZHr)](https://mermaid-js.github.io/mermaid-live-editor/edit#pako:eNqdVDtrwzAQ_iuHpgTSoR49BEKSoRRKaDoVLUK6pKLyyZXPLSXkv1dS4hLnUZJOxqf77nv4rI3Q3qAoRYMfLZLGmVXroCpJbNkhvFq2sMSmsZ4aUGRg6olQc3qXJKlWga22tSKGqbMYH5O6dlar1NI_X84e4b5fmps1wrNvGcPx0RPylw_viZCDdw7DZWhxSnRUOhAV_YTPNC13dPNh4RRhLHKWCXfj8RkJJUwWD10gklIgHaRIkKsRPeO3kfWMXwdFMp3djL7R85VG8-g9JwzqKK-JSVuNw1t93zqp-Nekg1RmitUuDRi8TBfDRHC6zpllF1UJse3gZ8i6ckJ5uc6iijOoCxqS7GH_w_SSK_OQv8LIhF3bnuU0_B4udtugW8tiJCoMlbIm3gwbSQBS8BtWKEVcBmFwpVrHUkjaxlbVsl9-kxYlhxZHoq2N4u4iEeVKuea3OjeWfdgXtz_UjZHr)

## Control Plane
 1. The API Session is the first and primary session between and endpoint and the OpenZiti network instance.  This session is created during attachment, after validating the certificates in both directions, and the endpoint name.  This makes the endpoint present on the network, and all endpoints and routers have API sessions to the Controller(s)
2. The Edge Session is created with the API session authorization, and is specific to each service configured for the endpoint.  The edge session object holds information such as the service policies, parent API session, service ID, and other information the endpoint and network require to properly service each given service.
3. Channels are formed between the endpoint and each Edge Router available and within the policies.  These channels are monitored for latency to select best path, and are the control connections for incoming connections for hosted services.
4. Links connect Edge Routers logically.  Edge Routers can advertise a listener socket, which is distributed during client initialization to other Edge Routers.  All Edge Routers will attach to all others in a mesh, provided the policy dictates/allows it.  Each pair of routers will have one link per link type (TLS, WSS, etc.)  Links are a split connection, having both control plane and data plane messaging.

## Data Plane

  1. The TCP connections at either end of an OpenZiti connection are dependent on the implementation model.  If Tunnelers, or Edge Router with embedded Tunnelers are used, and the end device makes a TCP connection to gain entry to the OpenZiti network.  If the endpoints, both dialing and binding, or either one, is fully embedded via the SDK, these connections will not exist.
  2. The Connection is the flow specific connection between the endpoint and the initial Edge Router.  Each service invocation will create an independent Connection, and data will flow over this to the Edge Router
  3. The Fabric Circuit is the path in the OpenZiti Network from initial to terminating Edge Router, comprised of one more more Edge Routers, and zero or more Links. (An initiating Edge Router may have a local terminator for the service) 

These terms in their full and abbreviated forms appear in logs, metrics, and software, and are therefore critical terminology to understand OpenZiti Networks.