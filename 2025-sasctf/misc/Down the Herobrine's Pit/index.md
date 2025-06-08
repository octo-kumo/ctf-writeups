---
created: 2025-05-24T15:31
updated: 2025-05-24T15:40
---

So a minecraft jail?

I made my alt travel to me from a few thousand blocks away using baritone for some extra wood so I can make a trapdoor to go through the hole.

Wait I am given a lot of trapdoors?

Oh...

## analysis

Using free-cam I see a chest at the bottom of this maze, and I realized that there is no way I'll navigate this manually.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1748115384/2025/05/2963da3a492a0fa72bbcef9efbaa759b.png)

So I wrote (with GPT) a small client side mod to A* search a path to the chest at the bottom.

```java
package me.kumo.pathfinder.client;

import com.mojang.brigadier.Command;
import com.mojang.brigadier.arguments.IntegerArgumentType;
import me.x150.renderer.event.RenderEvents;
import me.x150.renderer.render.CustomRenderLayers;
import me.x150.renderer.render.WorldRenderContext;
import me.x150.renderer.util.Color;
import net.fabricmc.api.ClientModInitializer;
import net.fabricmc.fabric.api.client.command.v2.ClientCommandManager;
import net.fabricmc.fabric.api.client.command.v2.ClientCommandRegistrationCallback;
import net.fabricmc.fabric.api.client.command.v2.FabricClientCommandSource;
import net.minecraft.client.MinecraftClient;
import net.minecraft.client.render.VertexConsumerProvider;
import net.minecraft.client.util.BufferAllocator;
import net.minecraft.client.world.ClientWorld;
import net.minecraft.text.Text;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.Vec3d;

import java.util.*;

public class PathfinderClient implements ClientModInitializer {
    private static List<Vec3d> pathPoints = Collections.emptyList();

    @Override
    public void onInitializeClient() {
        // Register client-side command: /pathfind <x> <y> <z>
        ClientCommandRegistrationCallback.EVENT.register((dispatcher, regAccess) -> dispatcher.register(ClientCommandManager.literal("pathfind").then(ClientCommandManager.argument("x", IntegerArgumentType.integer()).then(ClientCommandManager.argument("y", IntegerArgumentType.integer()).then(ClientCommandManager.argument("z", IntegerArgumentType.integer()).executes(ctx -> {
            int x = IntegerArgumentType.getInteger(ctx, "x");
            int y = IntegerArgumentType.getInteger(ctx, "y");
            int z = IntegerArgumentType.getInteger(ctx, "z");
            BlockPos target = new BlockPos(x, y, z);
            ClientWorld world = MinecraftClient.getInstance().world;
            BlockPos start = MinecraftClient.getInstance().player.getBlockPos();
            List<BlockPos> path = findPath(world, start, target);
            FabricClientCommandSource source = ctx.getSource();
            if (path == null) {
                source.sendError(Text.literal("No path found to " + target));
                return 0;
            }
            List<Vec3d> pts = new ArrayList<>();
            for (BlockPos p : path) {
                pts.add(new Vec3d(p.getX() + 0.5, p.getY() + 0.5, p.getZ() + 0.5));
            }
            pathPoints = pts;
            source.sendFeedback(Text.literal("Rendered path of length " + path.size()));
            return Command.SINGLE_SUCCESS;
        }))))));

        RenderEvents.AFTER_WORLD.register(ms -> {
            if (pathPoints.isEmpty()) return;
            VertexConsumerProvider.Immediate vcp = VertexConsumerProvider.immediate(new BufferAllocator(1024));
            WorldRenderContext rc = new WorldRenderContext(MinecraftClient.getInstance(), vcp);

            for (int i = 0; i < pathPoints.size() - 1; i++) {
                Vec3d a = pathPoints.get(i);
                Vec3d b = pathPoints.get(i + 1);
                rc.drawLine(ms, CustomRenderLayers.LINES_NO_DEPTH_TEST.apply(0d), a, b, new Color(0xFF0000FF));
            }
            vcp.draw();
        });
    }

    private List<BlockPos> findPath(ClientWorld world, BlockPos start, BlockPos goal) {
        Set<BlockPos> closed = new HashSet<>();
        PriorityQueue<Node> open = new PriorityQueue<>(Comparator.comparingInt(n -> n.f));
        Map<BlockPos, Integer> gScore = new HashMap<>();

        gScore.put(start, 0);
        open.add(new Node(start, null, 0, heuristic(start, goal)));
        List<BlockPos> dirs = Arrays.asList(new BlockPos(1, 0, 0), new BlockPos(-1, 0, 0), new BlockPos(0, 1, 0), new BlockPos(0, -1, 0), new BlockPos(0, 0, 1), new BlockPos(0, 0, -1));
        int maxIters = 1000_000;
        while (!open.isEmpty() && maxIters-- > 0) {
            Node curr = open.poll();
            if (curr.pos.equals(goal)) {
                return reconstruct(curr);
            }
            closed.add(curr.pos);

            for (BlockPos d : dirs) {
                BlockPos next = curr.pos.add(d);
                if (next.getY() > 114) continue;
                if (closed.contains(next)) continue;
                if (world.isOutOfHeightLimit(next)) continue;
                if (!world.getBlockState(next).isAir()) continue;

                int tentativeG = curr.g + 1;
                if (tentativeG < gScore.getOrDefault(next, Integer.MAX_VALUE)) {
                    gScore.put(next, tentativeG);
                    open.add(new Node(next, curr, tentativeG, heuristic(next, goal)));
                }
            }
        }
        return null;
    }

    private int heuristic(BlockPos a, BlockPos b) {
        return Math.abs(a.getX() - b.getX()) + Math.abs(a.getY() - b.getY()) + Math.abs(a.getZ() - b.getZ());
    }

    private List<BlockPos> reconstruct(Node node) {
        List<BlockPos> path = new ArrayList<>();
        for (Node n = node; n != null; n = n.parent) path.add(n.pos);
        Collections.reverse(path);
        return path;
    }

    private static class Node {
        BlockPos pos;
        Node parent;
        int g;
        int f;

        Node(BlockPos pos, Node parent, int g, int h) {
            this.pos = pos;
            this.parent = parent;
            this.g = g;
            this.f = g + h;
        }
    }
}
```

It seems to work well (after some debugging).

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1748115347/2025/05/f53fc282d166037c3bdd8ed5218196c5.png)

## the flag

So the path has been found, I have to navigate down there using trapdoors, avoiding fall damage, etc.

![2025-05-24_13.57.58.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1748115176/2025/05/a72cd0a46bd5825868074dc5832c33b6.png)

Well the journey took a lot more effort than I would prefer.

I have now become a master of minecraft 1x1 tunnel navigation using trapdoors.

I would strangle the author from behind while stabbing him several times in the chest if I ever find him on the streets.

![2025-05-24_15.27.00.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1748115158/2025/05/587dfeb72ebb7898df596a6fed817932.png)

```flag
SAS{c4v3_d1v3rs_f0r_n0_r34s0n_wh3n_th3y_h3ar_th3r3_1s_a_d1amond_d0wn_th3_h3r0br1n3s_p1t}
```
