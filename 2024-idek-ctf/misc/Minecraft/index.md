---
ai_date: '2025-04-27 05:18:25'
ai_summary: Exploited knockback calculation from shooter's location to triangulate
  player's position in Minecraft.
ai_tags:
- ssrf
- triangulation
- exploitation
created: 2024-08-17T03:27
points: 363
solves: 29
updated: 2024-08-18T21:33
---

Minecraft arrow knockback coordinate leak
## problem

- A minecraft server that has a player at a random location who will shoot arrows.
- Those arrows will teleport on demand to my location on `/chal`.
- We need to find the location of the player.

## exploit

The thing we are exploiting is the knockback player receives when hit by a projectile.

For some reason that speed is calculated not from the location of the projectile but from the location of the shooter.

This means that by taking the knockback received by the our player at different points on the map, we can triangulate the location of the shooter.

:vimeo{vid="999955632"}

> Note that in the above video, my bot is too minimalistic to react to knockbacks, however as the player you will experience the knockback.

## solve

I will make two bots join the server, both bots will switch to creative mode and teleport up.
- bot 1 goes to `y=190` and continuously call `/chal`
- bot 2 goes up to `y=300`, then back to `y=180` to let bot 1 identify it
- bot 2 clones 1 level of bedrock under its feet and switches to survival
- bot 1 records the `entity_velocity` packets received for bot 2

I am probably over-complicating this but who cares, fancy hacking go brrr.

After running the script a few times we obtain a set of points $(x,z),(dx,dz)$.

We can then use desmos to do the triangulation.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1723882071/2024/08/3564d050485b2950b9da2a5e8c34f6d2.png)

> The black lines are almost parallel, as you can see I was pretty unlucky there, but after using two points near the target we can triangulate the shooter location quite easily.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1723951129/2024/08/cd866c258e6897a0cd6eca2fffbabd3b.png)

```flag
idekCTF{storage_tech_is_my_passion_c1bdf8b2}
```

## script

```python
from threading import Thread
from time import sleep, time
from twisted.internet import reactor
from quarry.net.client import ClientFactory, SpawningClientProtocol
from quarry.net.auth import OfflineProfile

x = 0
z = 0


class BotProtocol(SpawningClientProtocol):
    focused_uids = set()

    def packet_entity_teleport(self, buff):
        entityId = buff.unpack_varint()
        x, y, z, yaw, pitch, onGround = buff.unpack('dddbb?')
        if (y >= 200):
            self.focused_uids.add(entityId)

    def packet_entity_velocity(self, buff):
        eid = buff.unpack_varint()
        vx, vy, vz = buff.unpack('hhh')
        vx /= 8000
        vy /= 8000
        vz /= 8000
        if eid in self.focused_uids:
            print("entity_velocity", eid, (x, z), (vx, vz))

    def send_command(self, text):
        data = [self.buff_type.pack_string(text)]
        data.append(self.buff_type.pack('QQ', int(time() * 1000), 0))
        data.append(self.buff_type.pack_byte_array(b''))
        data.append(self.buff_type.pack('?', False))
        data.append(self.buff_type.pack_last_seen_list([]))
        data.append(self.buff_type.pack('?', False))
        self.send_packet("chat_command", *data)

    def player_joined(self):
        print("Player joined!")
        if self.factory.logic_mode == 1:
            def tar():
                print("arrow launcher")
                self.send_command("gmc")
                self.send_command(f"tp2 {x} 190 {z}")
                while True:
                    # print("sending /chal")
                    self.send_command("chal")
                    sleep(1)
        else:
            def tar():
                print("arrow receiver")
                self.send_command("gmc")
                self.send_command(f"clone2 {x-5} -64 {z-5} {x+5} -64 {z+5} {x-5} 179 {z-5}")
                self.send_command(f"tp2 {x} 300 {z}")
                sleep(2)
                self.send_command(f"tp2 {x} 180 {z}")
                self.send_command("gms")

        self.thread = Thread(target=tar)
        self.thread.start()


class BotFactory(ClientFactory):
    protocol = BotProtocol


factory = BotFactory(OfflineProfile.from_display_name("kumo_tp"))
factory.logic_mode = 1
factory.connect("localhost", 25565)
factory = BotFactory(OfflineProfile.from_display_name("kumo_"+str(int(time() % 10000))))
factory.logic_mode = 2
factory.connect("localhost", 25565)

reactor.run()
```

```sh
ncat --listen 25565 --keep-open --sh-exec "ncat --ssl minecraft-hacked-87aa10920020004a.instancer.idek.team 1337" 
```

### usage

Change the variables `x` and `y` to get points to use for triangulation.

```
Player joined!
arrow receiver
Player joined!
arrow launcher
entity_velocity 548 (0, 0) (0.2055, -0.07375)
entity_velocity 548 (0, 0) (0.2055, -0.07375)
entity_velocity 548 (0, 0) (0.2055, -0.07375)
entity_velocity 548 (0, 0) (0.2055, -0.07375)
entity_velocity 548 (0, 0) (0.2055, -0.07375)
entity_velocity 548 (0, 0) (0.2055, -0.07375)
entity_velocity 548 (0, 0) (0.2055, -0.07375)
entity_velocity 548 (0, 0) (0.2055, -0.07375)
```

## chunk loading

I also made a chunk scanner that spams `/clone2 <>` on a grid of size 384 (the size of 24x24 chunks that the player loads).

However, there is like 200k commands I have to send and record the response of, it will take 1h+ to scan and I had issues determining whether a command actually succeeded or if the bot was kicked for spam, so I ditched it.

## client mod

After solving the challenge I thought about a client mod that simply prints server sent velocity packets telling the player how to move.

That will make everything a lot simpler actually.

```java
import net.fabricmc.api.ClientModInitializer;
import net.fabricmc.fabric.api.client.networking.v1.ClientPlayNetworking;
import net.minecraft.client.MinecraftClient;
import net.minecraft.network.packet.s2c.play.EntityVelocityUpdateS2CPacket;
import net.minecraft.text.Text;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class VelocityRecorderMod implements ClientModInitializer {
    private static final Logger LOGGER = LogManager.getLogger("velocityrecorder");

    @Override
    public void onInitializeClient() {
        ClientPlayNetworking.registerGlobalReceiver(EntityVelocityUpdateS2CPacket.PACKET_ID, (client, handler, buf, responseSender) -> {
            EntityVelocityUpdateS2CPacket packet = new EntityVelocityUpdateS2CPacket(buf);

            client.execute(() -> {
                if (client.player != null && packet.getId() == client.player.getId()) {
                    double velocityX = packet.getVelocityX() / 8000.0;
                    double velocityY = packet.getVelocityY() / 8000.0;
                    double velocityZ = packet.getVelocityZ() / 8000.0;
                    String velocityMessage = String.format("Received Velocity Update: X=%f, Y=%f, Z=%f", velocityX, velocityY, velocityZ);
                    LOGGER.info(velocityMessage);
                    MinecraftClient.getInstance().player.sendMessage(Text.of(velocityMessage), false);
                }
            });
        });
    }
}
```