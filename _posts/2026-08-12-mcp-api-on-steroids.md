---
title: "MCP: an API on steroids (and why we built one just for our own operation)"
lang: en
last_modified_at: 2026-08-12T20:00:00-03:00
categories:
  - articles
tags:
  - mcp
header:
  image: /assets/images/nonsense.jpg
  overlay_image: /assets/images/nonsense.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
---

The buzzword right now is MCP (Model Context Protocol). The protocol was released by Anthropic in November 2024 as an open standard for connecting language models to external systems, and in no time at all it became the default way to give an LLM "hands." Claude, ChatGPT, Cursor, Gemini — all using MCP. There are now dedicated websites for registering and listing a huge variety of MCP servers.

In practice, an MCP server is nothing more than an API on steroids. A mix of REST and GraphQL, with one difference that changes everything: the consumer isn't a developer reading documentation — it's a language model deciding in real time what to call and in what order.

## What MCP inherited from REST

Not long ago, this comparison would have felt forced. MCP was born as a stateful, bidirectional protocol over JSON-RPC: the client opened a connection, performed an `initialize` handshake, received an `Mcp-Session-Id`, and carried that session through every subsequent call. That worked great for a process running on your laptop and very poorly for a remote server with more than one instance.

The 2026-07-28 specification is the largest revision to the protocol since its launch, and it brought MCP closer to REST on three fronts.

**Statelessness.** The `initialize` handshake and the `Mcp-Session-Id` header were removed. Each request carries the protocol version, client identity, and client capabilities directly in the `_meta` field, so any server instance can handle any request. A plain round-robin load balancer works fine — no sticky sessions, no shared session store. When the server needs to maintain state across calls, it returns an explicit handle that the model passes back as an argument, much like a resource ID in a URL.

**Plain HTTP infrastructure.** The HTTP transport now exposes the method being called in a header (`Mcp-Method`). This allows gateways to route requests, apply rate limits, and enforce authorization policies without opening the request body. Authorization follows OAuth, increasingly aligned with OpenID Connect.

**Caching.** Responses to `tools/list`, `resources/list`, and `prompts/list` can be cached by the client for however long the server indicates in the `ttlMs` field.

There is also an older inheritance: in MCP, *resources* are identified by URI, exactly like REST resources.

## What MCP inherited from GraphQL

From GraphQL, MCP inherited the idea of a self-documenting API. A GraphQL client runs introspection and discovers the schema. An MCP client calls `tools/list` and receives each tool with its name, a natural-language description, and a JSON Schema for its parameters (and, optionally, its output). There is no separate Swagger file that can fall out of sync with the code: the contract is served by the server itself.

<!-- FILL IN: replace with a real tool JSON from your server -->
```json
{
  "name": "search_items",
  "description": "Fulltext search over catalog Items (name, description, keywords, category) to resolve item_id. A numeric query looks up by id directly. Pass region_code to restrict results to a specific market.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Free-text search string or numeric item id"
      },
      "region_code": {
        "type": "integer",
        "description": "Region code to restrict results to a local market"
      },
      "limit": {
        "type": "integer",
        "description": "Maximum number of results to return (default: 20, max: 50)"
      }
    },
    "required": ["query"]
  }
}
```

Notice that the description isn't written for a human. It is, in practice, a prompt: it tells the model *when* to use the tool. That changes quite a bit about how you think about documentation.

Also from GraphQL came the single endpoint. Everything goes through the same address, and what changes is the JSON-RPC method being called (`tools/call`, `resources/read`, and so on), not the URL. Not that this is necessarily a good thing — anyone who has had to debug GraphQL knows how annoying it is to figure out which method an error is coming from inside a query with multiple entries.

## What MCP left behind

GraphQL hands the client a query language. The client can build whatever query it wants, at whatever depth it wants, and the server has to defend itself with depth limits, cost analysis, and persisted queries. MCP has none of that. The client can only invoke the operations the server decided to expose, with the parameters the server defined. The surface area is closed by design.

And the infamous N+1? It is worth being honest here. N+1 is a classic GraphQL problem (each resolver firing a query per item, usually solved with DataLoader), and MCP does not eliminate it by magic. The problem just moves to a different layer. If you expose overly granular tools — like `fetch_order` and `fetch_customer` — the model will call them one by one, in a loop, to answer something like "which of tomorrow's orders haven't been paid yet?" That's N+1 tool calls, each with network latency and token cost.

The defense is in design, not in the protocol: intent-oriented tools that return everything the model needs for a given task in a single call. One call to `pending_orders(delivery_date:)` instead of ten calls to `fetch_order`. On the server side, good old ActiveRecord `includes` keeps doing the heavy lifting.

Knowing what the user wants matters. Instead of exposing models and returning raw data, we should ask what the user is trying to accomplish and respond with refined, purpose-built information.

## Wouldn't a well-documented REST API do the job?

That debate is legitimate and has no single answer. Those who say yes argue that LLMs already read OpenAPI specs quite well, that an MCP server is yet another surface to maintain, version, and secure, and that many MCP servers out there are just thin wrappers around a REST API that already existed. There is also the security angle: prompt injection and malicious tool descriptions are real risks when a model acts on text it does not control.

Those who say no point out that MCP standardizes the client side: the same server works with any compatible client, without writing integration code for each one. Discovery and authorization are also standardized. And, most importantly, a REST API is designed around resources, while a good MCP server is designed around intents. Mirroring REST endpoints one-to-one into tools tends to produce exactly the N+1 problem described above. Finally, the protocol offers features designed for human interaction — like asking the user for confirmation mid-call, tracking long-running tasks, and even rendering server-driven UI.

In my experience, the answer depends on who is on the other side. And it was with that in mind that we made a somewhat different decision.

## Our case: an MCP server built for internal use

I work at Reventals, a TapGoods marketplace for renting equipment for parties and events. In practice it works like an e-commerce platform, with a few quirks.

When people talk about MCP for e-commerce, the image that comes to mind is an agent buying on behalf of a customer. Our approach was different: an internal MCP server, built for our own operators. The users are the operations team, who previously depended on the admin panel — or on me — to update marketing pages specific to each region, add partners, and look up items and categories. With the server connected to Claude Desktop or Claude Code (yes, our operations team has been successfully using Claude Code too), they query and manage the site in natural language.

A few reasons led us to start internally. The audience is known and authenticated, which significantly reduces risk. Every new question from operations that used to become a ticket or a new admin screen can now be answered by combining tools that already exist. And internal use is a safe laboratory for learning how to design tools before exposing anything to customers.

### Design decisions

**Tools by intent, not by table.** Instead of mirroring Rails models, each tool corresponds to something an operator actually asks or does. For example, we have regional wedding pages and style examples for each region — with Claude's help, the team can not only register new pages but also get assistance writing copy.

**Read separated from write.** Query tools and data-mutation tools are treated differently. We make write operations explicit with tool annotations like `destructiveHint` and a restricted list of write operations.

**Auditing.** Every `tools/call` is logged with the operator, arguments, and result. Changes are stored so we can recover if any information is lost (just in case Claude decides to hallucinate).

**Descriptions as prompts.** Tool descriptions were written and rewritten by watching model behavior, not for a developer to read. We had the operations team run assisted tests and tell us how they wanted to "talk" to the model.

## Conclusion

MCP is not magic. It is API design with a different consumer, and the latest specification, by going stateless, made the protocol look even more like the REST we already know. The truly new part lies in designing tools for a model that decides on its own what to call.

And perhaps the most interesting use case is not the most obvious one. Before putting an agent in front of your customers, it's worth asking who inside your company spends their day requesting reports, opening tickets, or navigating admin screens. Your first MCP server might just be for those people.
