# valheim server host: real specs you need, VPS vs managed hosting, and how to set up a server that stays online

When people search for a Valheim server host, they're usually after the same few things: a world that stays up when nobody's playing, a connection that doesn't lag when five friends are all online, and a monthly bill that doesn't feel like a second rent. What they find instead is a wall of game-hosting brands charging per slot, plus forum threads full of people arguing about RAM.

This article covers both realistic options. You can pay a managed game host to click the buttons for you, or rent a small VPS and run the dedicated server yourself — which is often cheaper once you want more than 4 GB of RAM, and gives you full control over mods, backups, and restarts. We'll go through the actual hardware requirements, the setup process step by step, the plan pricing for one VPS provider that fits this use case well (Sharktech, a DDoS-focused host), and the problems that tend to bite people in the first week.

## What a Valheim dedicated server actually needs

Let's start with the numbers, because hosts love to oversell them.

The community wiki lists the dedicated server's minimum requirements as a quad-core CPU at 2.8 GHz, 2 GB of RAM, and 2 GB of storage, with a hexa-core at 3.4 GHz and 4 GB of RAM recommended. Those minimums are technically true — the server will boot with less than a gaming PC needs, since it doesn't render anything.

In practice, players running dedicated servers report that 8 GB of RAM is a comfortable target if you're running mods, hosting a large explored world, or expecting 5–10 concurrent players. A small world with 2–4 players runs fine on 4 GB. The game and world data together take roughly 10–20 GB of disk, so an NVMe drive of 40 GB or more gives you plenty of headroom for the game, logs, and automatic backups.

Two things matter more than raw numbers:

- **Single-thread CPU performance.** Valheim's server logic doesn't spread evenly across many cores. A fast modern core beats a slow one with eight threads. This is why old "16-thread" bargain servers sometimes run Valheim worse than a 4-core VPS with newer Xeon silicon.
- **Uptime and network quality.** The whole point of a dedicated server is that the world persists while you're asleep. A host with redundant infrastructure and decent peering matters more than an extra 2 GB of RAM.

One more spec people forget: the server communicates over UDP ports 2456–2457 by default. On a VPS or dedicated machine you control, opening those ports yourself takes about a minute. With the Crossplay backend enabled (more on that below), the server routes through a relay and doesn't even need port forwarding.

## Managed Valheim hosting vs. running your own VPS

You have two realistic routes, and each has a clear audience.

**Managed game hosting** (the brands that advertise "Valheim servers from $X/month") gives you a web panel: pick a slot count, click deploy, and you're in the game in ten minutes. Updates are usually handled for you. The tradeoff is price scaling — you pay for convenience, and once you want more RAM for mods or a bigger community, the per-GB cost climbs fast. You're also locked into their panel, their mod manager, and their restart policies.

**A VPS** flips that. You install and run the official Valheim dedicated server yourself — the same binary the managed hosts run, available free through Steam. You handle updates, firewall rules, and backups, but you control everything. A VPS with 8 GB of RAM from a budget provider typically costs less per month than an equivalent managed game-server slot plan, and the machine can do double duty as a Minecraft server, a website, or a TeamSpeak box when the Viking crew is offline.

The honest summary: if the phrase "edit a shell script" makes you want to close the browser tab, use a managed host and stop reading here — no shame in that. If you're comfortable with a terminal, or willing to follow a guide for twenty minutes, a VPS is the cheaper and more flexible path, especially for groups larger than a handful of players.

One warning that applies to *both* routes: game servers attract DDoS attacks, whether it's a banned player with a grudge or someone targeting the host's IP range. Some providers include real mitigation; others will simply null-route your IP, which is a fancy way of saying they take your server offline until the attack stops. This is worth checking before you buy, not after.

## Setting up a Valheim dedicated server on a Linux VPS

The official dedicated server is free — you're only paying for the machine it runs on. On a Linux VPS the process looks like this:

1. **Install SteamCMD** (Valve's command-line client) — it's packaged for most distros or downloadable directly from Valve.
2. **Download the server**: log in anonymously with SteamCMD and pull the Valheim dedicated server app (app ID 896660). It lands in a folder of your choosing.
3. **Install the required libraries** if you're on Linux: the official guide lists `libatomic1`, `libpulse-dev`, and `libpulse0`. If your distribution ships a GLIBC older than 2.29, use the bundled Docker script instead of upgrading half your system.
4. **Edit `start_server.sh`** (the Linux launcher). The key parameters:
   - `-name "Your Server Name"` — what shows up in the community list
   - `-world Dedicated` — the world name to create or load
   - `-password YourPassword` — required for public Steam servers
   - `-public 1` — makes the server visible in the browser; set 0 for an invite-only world
   - `-crossplay` — runs on the PlayFab backend so Xbox and Game Pass players can join; this mode uses a relay and skips port forwarding entirely
5. **Open ports 2456–2457 (UDP)** in your firewall if you're on the Steam backend. Skip this step if you're using crossplay.
6. **Run the script.** When the console prints "Game server connected," your world is live.

The server saves the world every 30 minutes by default and keeps four automatic backups (one 2 hours old, three 12 hours apart) — you can tune all of this with `-saveinterval`, `-backups`, `-backupshort`, and `-backuplong` flags. Admin, ban, and permit lists are plain text files (`adminlist.txt`, `bannedlist.txt`, `permittedlist.txt`) where you paste one platform user ID per line. Note the trap in the last one: adding anyone to `permittedlist.txt` bans *everyone else*.

Stopping the server matters more than it should: press CTRL+C in the console window. Closing the window with the X button can leave the process running in the background, which leads to "my port is already in use" confusion on the next start.

For 24/7 operation, wrap the launcher in a systemd service or a `screen`/`tmux` session so it survives your SSH session ending and restarts after a reboot. That's standard Linux housekeeping, not Valheim-specific.

## Sharktech Smart VPS: plans and why this setup fits Valheim

If you're going the VPS route, the provider needs to check three boxes for a game server: low-latency network in a region near your players, NVMe storage (the world saves and loads a lot), and DDoS protection that's actually included rather than a paid add-on.

Sharktech is a hosting company that's been around since 2003, running its own network (AS46844) with data centers in Las Vegas, Los Angeles, Denver, Chicago, and Amsterdam. Its Smart VPS line is built on Proxmox clusters with Xeon Gold CPUs and enterprise NVMe storage, and every plan — including the cheapest one — includes 60 Gbps of DDoS mitigation per IP as a standard feature, not an upsell. That last point is why the company shows up in game-hosting conversations: one of their game-server customers reports absorbing 3–8 Gbps attacks regularly without service interruption, and their own network was designed around attack mitigation from the start.

The unusual part of Smart VPS is that you buy a **resource pool**, not a single fixed VM. You get a chunk of CPU, RAM, and NVMe storage, then carve it into as many virtual machines as the resources allow — one big VM, or a handful of small ones spread across different cities. You can upgrade or downgrade at any time without redeploying. For a Valheim group this means you can start with a tiny 4 GB machine for a small world and scale up later without migrating your world files to a new server.

Here are the current Smart VPS tiers as listed at the time of writing, with all four billing cycles — note that the annual cycle applies a flat 50% discount automatically, no coupon code needed:

| Plan | vCPU (Xeon Gold) | RAM | NVMe Storage | Monthly | Quarterly (−25%) | Semi-Annual (−35%) | Annual (−50%) | Get it |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Tiny | 2 | 4 GB | 40 GB | $7.95/mo | ~$5.96/mo | ~$5.17/mo | **~$3.98/mo** | [ Order Tiny](https://bit.ly/SharKTech) |
| Small | 2 | 4 GB | 60 GB | ~$15.95/mo | ~$11.96/mo | ~$10.37/mo | **~$7.98/mo** | [ Order Small](https://bit.ly/SharKTech) |
| Medium | 4 | 8 GB | 80 GB | ~$39.95/mo | ~$29.96/mo | ~$25.97/mo | **~$19.98/mo** | [ Order Medium](https://bit.ly/SharKTech) |
| Large | 8 | 16 GB | 160 GB | $99.95/mo | ~$74.96/mo | ~$64.97/mo | **~$49.95/mo** | [ Order Large](https://bit.ly/SharKTech) |
| Colossal | 16+ | 32+ GB | up to 2000 GB | $299.99/mo | ~$224.99/mo | ~$194.99/mo | **~$149.99/mo** | [ Order Colossal](https://bit.ly/SharKTech) |

A few caveats worth knowing before you click:

- The exact vCPU/RAM/storage split within each tier is adjustable with sliders on the order page, and third-party reviews report slightly different baseline configurations per tier — so treat the specs above as starting points and confirm the exact numbers on the order form, which updates the price live as you move the sliders.
- Bandwidth on the platform runs 4–304 TB depending on tier, with a 1 Gbps port. A Valheim server with ten players uses a trivial fraction of even the smallest allowance.
- All five data center locations are available, so pick the one closest to most of your crew. Sub-millisecond latency to major backbones matters less for a co-op game than for CS:GO, but nobody wants a 150 ms Viking raid either.
- **Sharktech has a strict no-refund policy.** All payments are non-refundable. If you're unsure, the monthly cycle on a Tiny plan is a $7.95 experiment — don't prepay a year before you've confirmed the setup works for you.
- Plans are unmanaged: you get root and full OS choice (Ubuntu, Debian, AlmaLinux, and other Linux distros; Windows Server works but you bring your own license). A HostAdvice review measured 12-minute ticket response times with technically accurate answers, but support won't hold your hand through basic Linux administration.

For most Valheim groups, the math is straightforward. Two to four friends on a vanilla world: **Small** or even **Tiny** on annual billing, i.e. $4–8/month for a persistent world. Five to ten players, or a modded setup: **Medium**, whose 8 GB of RAM matches the community's real-world recommendation, at roughly $20/month on the annual cycle. The bigger tiers are for people running multiple servers or other services alongside — at which point the resource-pool model starts paying for itself, since one Large allocation can host a Valheim VM, a Minecraft VM, and a small website simultaneously.

If your crew grows into the hundreds and you want bare metal instead, Sharktech also sells dedicated servers and OpenStack-based Public Cloud plans (from $39/month for the Small tier up to $499/month for Enterprise), all with the same included DDoS protection. That's beyond what any sane Valheim world needs, but it's there.

## Keeping the world safe: updates, backups, and the community list

The world file is the only thing on the server you can't rebuild, so treat it accordingly.

**Updates.** Valheim updates ship through Steam, so updating the server means re-running the SteamCMD download command (`app_update 896690` — wait, no: it's `app_update 896660 validate`) and restarting the launcher. Players on a mismatched version can't join, so after a game patch you have a short window where nobody can connect until you update. Automating the update-and-restart as a weekly cron job is a common pattern; doing it manually after each patch is also fine for a friends-only server.

**Backups.** Beyond the server's built-in four rotating backups, copy the world folder (`~/.config/unity3d/IronGate/Valheim` on Linux by default) off the machine on a schedule. The world files are small — a few hundred MB even for a long-running, heavily explored world — so this costs nothing and turns "the disk died" into a five-minute restore instead of a funeral for the seed you played for two years.

**Visibility.** If your server doesn't show up in the community list, check three things in order: `-public` is set to 1, the password meets the minimum length, and you've given it a few minutes — newly started servers take a while to appear in the browser. If patience doesn't work, players can always join directly with `IP:port`, or via the join code if you're on the crossplay backend.

## Common problems and quick fixes

- **Server won't start on an older distro** — GLIBC below 2.29 breaks the Linux binary. Fix: use the official `docker_start_server.sh` script, which ships with the server, or move to a newer base image (Ubuntu 22.04/24.04 has no issues).
- **"Game server connected" but nobody can join** — ports. On the Steam backend, UDP 2456–2457 must be open in *both* the host firewall and the provider's panel. Switching to `-crossplay` sidesteps port forwarding entirely at the cost of routing through PlayFab relays.
- **Xbox friends can't see the Steam world** — add the `-crossplay` flag, which puts the server on the PlayFab backend where all platforms meet. Without it, only Steam players can join.
- **The server lags when everyone's in Mistlands** — that's RAM and single-thread CPU pressure. On a resource-pool VPS this is a slider adjustment and a reboot, not a migration to a new machine.
- **Someone DDoSes the server after getting banned** — this is the scenario where a provider like Sharktech earns its keep. Included 60 Gbps mitigation means the ban sticks; on a host without real protection, the same event usually means your IP goes offline for hours.

## The verdict

For most people searching "valheim server host," the answer is smaller than the internet makes it look. Your persistent world needs a couple of fast cores, 4–8 GB of RAM, 20 GB of NVMe, open ports, and a network that stays up under pressure. A managed game host will rent you exactly that with zero setup; a VPS rents you the same thing for less money once you outgrow the entry tiers, plus the freedom to mod, automate, and repurpose the machine.

If you go the VPS route, Sharktech's Smart VPS line fits this use case well: the 60 Gbps DDoS protection included on every plan removes the most annoying failure mode for game servers, the five US/EU locations keep ping reasonable for mixed crews, the NVMe storage handles world saves without drama, and the annual 50% discount brings an 8 GB machine to around $20/month — genuinely competitive with managed Valheim plans that give you far less control. Start on the monthly cycle given the no-refund policy, confirm it runs your world, then lock in the annual price.

👉 [Deploy a Sharktech Smart VPS and get your world online](https://bit.ly/SharKTech) — the Tiny plan on annual billing is about the price of a coffee per month, which is a fair trade for a world that's always waiting when the crew logs on.
