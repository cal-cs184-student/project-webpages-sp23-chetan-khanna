# Project 4 Task 2

# Experimenting with different constants

## Reference view (default parameters)

![Image.png](Project%204%20Task%202.assets/Image.png)

This is what the cloth looks like with default parameters `ks = 5000`, `damping` set to `0.2`% and `density = 15`.

## Changing the spring constant `ks`

#### Increasing spring constant: `ks = 50000`

![Increased spring ks.png](Project%204%20Task%202.assets/Increased%20spring%20ks.png)

The cloth in the resting position with `ks = 50000`.

Increasing the spring constant from 5000 to 50000 did not appear to affect the speed at which the cloth “fell down” (at least significantly). However, the crease (at the upper part of the cloth) between the two pinned corners did become less prominent, and there are fewer creases (the cloth is straighter).

This is expected behavior; the springs require more force for any displacement, so the force of gravity is not causing as much of a displacement on the springs (or, perhaps more accurately, the force exerted by the lower strings on the upper ones isn’t affecting the displacement of the upper ones as much), thus causing the cloth to be less “stretchy” in a sense.

#### Decreasing spring constant: `ks = 50`

![Decreased spring ks.png](Project%204%20Task%202.assets/Decreased%20spring%20ks.png)

The cloth in the resting position with `ks = 50`.

As expected, lowering the spring constant causes the cloth to be “stretchier”. There are multiple creases at the top now, rather than just one (and the dip between the two pinned corners is steeper). The cloth retains less of its shape, as can be demonstrated by the corners of the spring being *very* stretched out. This makes sense; it is the opposite of the case above, and thus even the “small” force applied by gravity affects the displacement of the springs more—and by one spring on the other).

## Changing `density`

#### Increasing density: `density = 1500`

![High density 1500.png](Project%204%20Task%202.assets/High%20density%201500.png)

The cloth's final position with `density = 1500`. Normal shading enabled.

Increasing the density will increase the mass of the cloth.

Note how the increased density causes the cloth to drop more noticeably between the two pinned corners. This makes sense: a cloth density will mean the lower springs are pulling the upper springs down with more force (due to their larger mass).

(While the upper springs themselves may *also* have more mass, this doesn't outweigh the extra force of the lower springs—by the time we're at the top of the cloth, there are obviously many more springs pulling *down* on the cloth than pulling *up* on the cloth. “At the limit”, in other words, the mass of the upper spring would be virtually negligible compared to the mass of *all* the lower springs.)

For the same reasons, it also causes the cloth to appear “stretchier”. As expected (and for the opposite reason), the cloth will appear a lot less stretchy.

![Low density 1.png](Project%204%20Task%202.assets/Low%20density%201.png)

The cloth's final position with `density = 1`. Normal shading enabled.

## Impact of damping

A higher damping setting $$d$$ will slow down motion. During Verlet intgration, we multiply $$(1-d)$$ with $$v_t$$ (the velocity of the particle at time $$t$$) when updating a particle's position for the new timestep; in other words, we scale down a particle's velocity at every timestep.

Damping drastically changes how quickly the cloth falls down, and all it’s movement (e.g. even the movement of the cloth’s springs between themselves).

Most noticeably, it also affects how much the cloth “oscillates” once it falls down. At lower damping settings, the cloth falls but then continues to oscillate at high amplitude and frequency. This makes sense; a lower damping setting means that the cloth doesn’t lose as much energy, and thus by the time it reaches the bottom it has far too much excess energy. It continues to oscillate for a long time, only losing a very minor amount of energy each time.

Because damping (obviously) also applies to the cloth’s springs, we also note how the cloth appears a lot less “smooth”. This is because the spring’s are moving much more “violently” when a force is applied on them (and this effect compounds and travels through springs—hence the continued “instability” that lasts for long). A higher damping settings tends to “smooth” out this effect by making the slight force one spring applies on the other at one moment in time not impact the spring’s next position as much.

Naturally, a high damping setting has the opposite effect, and all motion is slowed down. The cloth falls down very slowly, and barely oscillates around the bottom before coming to rest.

![Image.png](Project%204%20Task%202.assets/Image%20(2).png)

Illustration of the lack of damping: when the cloth first falls, it has so much energy that it is oscillating. Here the cloth is nearly 90 degrees offset from it’s rest point. Also note how the normals are often pointing in very different directions; a slight force applied by one spring on the cloth has a big impact on the rest.

![Image.png](Project%204%20Task%202.assets/Image%20(3).png)

The high damping setting creates a much smoother-looking cloth, and reduces the speed at which the cloth falls down. It also causes it to come to rest almost immediately.

---

# Screenshot of `scene/pinned4.json`

![Image.png](Project%204%20Task%202.assets/Image%20(4).png)

