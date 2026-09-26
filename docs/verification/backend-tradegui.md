# Backend trade command verification

Verified on September 26, 2026 for Minecraft 26.2 and SkyCofl 2.0.0-pre1.

## Matching source

* SkyModCommands: `a4ba436fb8f6b8744aed181e8a76ec920e09e75a` in [MCEnvision/SkyModCommands](https://github.com/MCEnvision/SkyModCommands/tree/envy/tradegui-command).
* CoflSkyCore: `e3660b25fb3f8ac8dafe0a43f6446b31b4870aa2` in [MCEnvision/CoflSkyCore](https://github.com/MCEnvision/CoflSkyCore/tree/envy/tradegui-command), resolved from JitPack and bundled in the client.
* Fabric: [the trade fixes branch](https://github.com/MCEnvision/SkyblockModFabric/tree/envy/fabric-26.2/2.0.0-pre1/trade-fixes).

The tested JAR is `SkyCofl-2.0.0-pre1-backend-tradegui-Fabric-26.2.jar`.
Its SHA256 is `1eb7992dbdcf4454070ae5a27c7d2efbd983e2c8ebfbb84d474914eaa2991143`.

## Results

The backend build passed all 340 ordinary NUnit tests. Core's test and build
passed, followed by its separate live WebSocket integration test against the
backend fixture. That test received the real command registry and forwarded on,
off, and status through `CoflSkyCommand.processCommand`.

Fabric's `./gradlew --no-daemon test build` passed 98 tests across the client and
test server modules. The packaged JAR contains the matching Core library and no
local `com.coflnet.commands.TradeGuiCommand` class.

The Minecraft 26.2 client launched through Trident on the laptop and joined an
authenticated disposable dedicated server on node-1. It used Fabric Loader
0.19.3, Fabric API 0.161.0+26.2, and the exact JAR digest above. After the normal
custom backend confirmation, the owner verified enable, disable, status, and
Tab completion of both on and off. Client logs show typed backend responses and
feedback on the render thread. The persisted preference changed with the responses.

## Limits and rehearsal lifecycle

The command fixture inherits the production MinecraftSocket message handler,
registry, command implementation, and serializer. It skips production login and
external service bootstrap. This verifies the command protocol and client effect,
not production authentication, deployment, real Hypixel trading, or frame rates.

Closing the client reproduced a separate shutdown watchdog crash about fifteen
seconds after the render thread logged stopping. A non daemon `Timer-1` remained
waiting in `java.util.TimerThread.mainLoop`, and Trident reported exit code 248.
Core's existing flip housekeeping and chat batching code both create non daemon
timers, but the dump does not identify which owns this thread. These timer sites
predate the command changes. Normal client exit remains unverified until the
surviving timer is identified and retired correctly. The Core fork has Issues
disabled, so this finding is retained here for review. All owned processes exited.

The explicit backend fixture has a nine minute lifetime. If it expires during
manual testing, commands cannot receive responses until it is restarted and the
client reconnects. A custom localhost endpoint still requires the existing
confirmation. Do not bypass that check. Retire the owned client, server, tunnel,
and fixture after the final consumer.

See [command ownership](../architecture/command-flow.md) for the complete flow.
