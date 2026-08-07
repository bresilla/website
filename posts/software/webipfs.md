+++
type = "post"
status = "published"
date = "2020-01-23"
readingtime = 8

slug = "web-ipfs"
title = "Hosting a website on the InterPlanetary File System"
thumbnail = "thumbnail.png"
foot = "Between 'I want to be alone' and 'I want to be let alone' there is huge difference - Greta Garbo"
categories = ["LINUX"]
series = ["IPFS"]
part = "1"
tags = ["ipfs", "website", "dnslink", "p2p", "crypto", "gpg", "blockchain"]
description = "Publish a static site to IPFS, pin it, test it through a gateway, and point a human-readable DNSLink name at the current CID."
punchline = "IPFS gives content an address derived from the content itself. That makes a static build verifiable and easy to mirror, but somebody still has to keep the blocks online."
tldr = "Build with relative or subdomain-safe URLs, add the output directory to IPFS, pin the root CID, and publish dnslink=/ipfs/CID in a _dnslink TXT record."
credits = [
    "https://docs.ipfs.tech/",
    "https://docs.ipfs.tech/quickstart/pin/",
    "https://docs.ipfs.tech/how-to/websites-on-ipfs/custom-domains/",
    "https://docs.ipfs.tech/how-to/websites-on-ipfs/dnslink-gateway/",
    "https://docs.ipfs.tech/how-to/websites-on-ipfs/dnslink-action/",
]

[style]
    accent = "#6acad1"
    theme = "dark"
+++

# Why I wanted a second route to the site

I do not expect GitHub Pages to vanish tomorrow. That was never the interesting argument for me. The interesting bit is that a normal URL says *where* to ask for a file, while an IPFS CID says *which bytes* I expect to receive.

If two machines provide the same CID, either can serve the content. A local IPFS node can verify the blocks instead of trusting whichever HTTP server answered first. That is useful for a static website because a finished build is just a directory of immutable files.

There are limits. IPFS does not make a site immortal, private, or magically distributed. Publishing a CID does not guarantee that anybody else stores it. Public gateways can log requests and may remove cached data. Dynamic server code does not fit this deployment model. The practical goal is smaller: publish a verifiable static copy that can be pinned by more than one provider.

{{< image url="ipfs-logo.jpg" border="1" width="30" >}} IPFS logo {{< /image >}}

# The pieces

The terms are easy to mix up:

- A **CID** identifies content. Change one byte and the CID changes.
- A **node** stores and exchanges blocks with peers.
- A **pin** tells a node or pinning service to retain content instead of garbage-collecting it.
- A **gateway** exposes IPFS content over HTTP for browsers that do not speak IPFS directly.
- **DNSLink** maps a DNS name to an `/ipfs/...` or `/ipns/...` path through a TXT record.

IPFS is not a blockchain, and a site does not need one. The content graph uses cryptographic hashes and links between blocks. Consensus about a global ledger is a different problem.

# Prepare the static build

Start with a site that can be served from static files. Build it into a clean directory such as `dist/`:

```bash
faqe build ./content --output ./dist
```

The exact build command depends on the generator. Inspect the output before adding it to IPFS:

```bash
find dist -maxdepth 2 -type f | sort | sed -n '1,40p'
```

Routing is the common trap. A site viewed through a path gateway may live below `/ipfs/CID/`. Root-relative links such as `/about/` point at the gateway root rather than the CID path. There are three reasonable fixes:

1. Build with relative links.
2. Build for the `/ipfs/CID/` base path after the CID is known, which is awkward because changing the build changes the CID.
3. Use a subdomain gateway such as `https://CID.ipfs.dweb.link/`, where the CID gets its own origin and `/about/` remains inside that origin.

I prefer subdomain gateways for testing and a DNSLink-aware domain for humans. They also give each site an isolated browser origin; path gateways cannot provide the same isolation between arbitrary CIDs.

# Add the directory with Kubo

Install [Kubo](https://docs.ipfs.tech/install/command-line/), initialize it once, and start the daemon:

```bash
ipfs init
ipfs daemon
```

In another terminal, add the build directory:

```bash
CID="$(ipfs add --recursive --quieter --cid-version=1 dist | tail -n 1)"
printf '%s\n' "$CID"
```

CIDv1 is useful for subdomain gateways because its lowercase base32 representation is DNS-safe. Confirm the root is pinned locally:

```bash
ipfs pin ls "$CID"
```

Open it through the local gateway:

```text
http://127.0.0.1:8080/ipfs/YOUR_CID/
```

Also try a public subdomain gateway:

```text
https://YOUR_CID.ipfs.dweb.link/
```

Do not declare victory after seeing only the home page. Open a nested post directly, reload it, inspect images, and check the browser console. Static-site routing bugs love direct URLs.

# Pin it somewhere that stays online

My laptop is not infrastructure. If it sleeps, goes offline, or garbage-collects the blocks, availability depends on some other node already having them.

A pinning service runs an IPFS node and retains the CID for an account. Upload the `dist/` directory through the provider's interface or use its current CLI/API instructions, then verify that the provider reports the same root CID. Pinning services differ in authentication and upload tools, but the invariant is simple: the published root CID must match the build tested locally.

For content I care about, I pin it in at least two independent places: my own Kubo node and a remote pinning service. Two pins are not decentralization theatre if they are genuinely independent machines. One provider with two product names does not count.

Test availability without relying on the local node. Stop the daemon temporarily or use another network, then request the CID through a gateway.

# Add a human-readable name with DNSLink

A CID is an excellent identifier and a terrible thing to say aloud. DNSLink puts the current CID behind a DNS name.

For `example.com`, publish this TXT record:

```text
Name:  _dnslink.example.com
Type:  TXT
Value: dnslink=/ipfs/YOUR_CID
```

DNS control panels disagree about whether the name should be `_dnslink`, `_dnslink.example.com`, or `_dnslink.example.com.`. Check the result instead of trusting the form:

```bash
dig +short TXT _dnslink.example.com
```

The output should contain:

```text
"dnslink=/ipfs/YOUR_CID"
```

You can then resolve it through a DNSLink-capable gateway:

```text
https://ipfs.io/ipns/example.com/
```

Serving the bare custom domain directly over HTTPS still requires an HTTP endpoint that knows how to resolve DNSLink. The current IPFS documentation describes running Kubo behind Caddy for that purpose. Another sensible setup is to keep the ordinary site on GitHub Pages or Cloudflare Pages and publish DNSLink alongside it. Normal browsers get the conventional site; IPFS-aware clients and gateways can discover the content-addressed copy.

# Updating the site

Every build produces a new CID when the output changes. The update procedure is therefore:

```bash
faqe build ./content --output ./dist
NEW_CID="$(ipfs add -Qr --cid-version=1 dist)"
printf '%s\n' "$NEW_CID"
```

Then:

1. Pin `NEW_CID` locally and remotely.
2. Test important routes through that CID.
3. Update the `_dnslink` TXT record.
4. Wait for DNS caches according to the record TTL.
5. Unpin old builds only after deciding they are no longer useful.

Keeping a few old CIDs gives the site cheap, immutable release history. A DNSLink update changes what the friendly name resolves to; it does not mutate the old build.

This sequence is easy to automate in CI, but DNS credentials deserve care. Use a token that can edit only the required DNS zone or record. Never print it in build logs, and pin the action versions used by the workflow.

# What IPFS proves, and what it does not

Fetching a CID through a verifying client proves that the returned blocks match that CID. It does not prove who created the site. Publish the CID through a domain you control, a signed release, or another authenticated channel if authorship matters.

An ordinary public gateway is also a trusted HTTP service from the browser's point of view. It may return the correct bytes, but the browser itself is not necessarily verifying the block graph. A local Kubo node or a verifying gateway changes that trust model.

That distinction is why I keep both versions. The conventional site is convenient. The IPFS copy is addressable by content, easy to mirror, and much harder to silently alter after I publish its CID. Neither one makes the other obsolete.
