---
author: admin
comments: true
date: 2010-10-24 23:33:19
layout: post
slug: concept-open-lobby-and-game-servers
title: 'Concept: Open Lobby and Game Servers'
wordpress_id: 157
---

This is just an idea that popped into my head, and this blog needs more content.

Has this ever happened to you? You just bought or downloaded a relatively old game, or perhaps you're just returning to a game you played ages ago. Now, there are always people willing to play. However, to play a multiplayer game, you need servers. And that's precisely the thing old games lack.

<!-- more -->

Now, what if there was some kind of protocol that would avoid this situation? Here's what I had in mind. The player decides he wants to play old game X. So, he goes to lobby.example.com which is a website that acts as a lobby. If necessary, they download the necessary software. Once that's done, they click Play X. The lobby server will automatically find a lobby (matchmaking) or create one if there isn't any. Then, the players wait until the lobby is full, or they invite their friends. In the background, a game server is picked from a list on the lobby server, or one of the players is selected, and ping-tested. Assuming the ping to the potential server is acceptable from all parties, one of the players (the one with the highest uplink to the server) will upload the necessary server software to the server, and the server executes this software in a modified version of QEMU. It also fills in a template for any files specific to this game/session, for example the map, amount of players, et cetera. Then, the server is launched, and the players are sent to a link for which a handler exists on the player's machine. The software then contacts the server, waits for it to be ready, and then starts the game itself, connecting to the server.

This has a great amount of benefits to the system we have now. For example, rather than having one server host one game, every system will be classified by how many 'shares' it can contribute. A share would be a unit of CPU, RAM and bandwidth, similar to the system vps.net uses. A share could just be 500 MHz, 256 MB RAM and 1 MB/s bandwidth. Every game would have a classification of how many shares it would require to run smoothly, and a server with that many shares available is taken. So a server that has 2 GHz, 1 GB of RAM and 4 MB/s would have 4 shares, and could host 4 games of 1 share simultaneously. The QEMU set-up would ensure that these limits are not breached. They would also take care of any security issues, since a QEMU-hosted OS doesn't have access to the host OS. Preferably, a digital signature would be introduced, similar to what Java has, to ensure security. Any game could be packaged in this manner, assuming it could be hosted on a lightweight Linux OS.

This is all speculative though, so any input is welcomed. Is this feasible at all? I know it could do wonders for indie games.

[![](http://ardaxi.com/wp-content/uploads/2010/10/v2wrB1-150x150.png)](http://ardaxi.com/wp-content/uploads/2010/10/v2wrB1.png)
