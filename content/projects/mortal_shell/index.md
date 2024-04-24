---
date: 2023-00-20
title:  "Mortal Shell"
description: "Bring the glands back to me..."
comments: true
math: true
featured_image: 'banner.png'
---

I got the opportunity to work on "Mortal Shell" as a gameplay programmer and designer. Mortal Shell is a soulslike action-RPG set in a shattered world where your skill and resilience is tested. Since release, the game has sold over 500 000 copies worldwide, and received a metacritic score of 76.

<!--more-->

{{< youtube r16Dc_jMWzA >}}

I was sort of a generalist on the team, implementing and refining a lot of gameplay elements and environmental objects using Unreal Engine 4's blueprints. I did a lot of polishing and bugfixing in many different areas of the game. I might have gotten a task, such as "The cathedral level needs a gate that gets unlocked by finding 3 different levers." I would then do my best to make it as polished as possible, with animation curves, utilizing existing sounds and particles and passing around ideas with the team. The core team was pretty small with only 15 people, so I got a lot of artistic freedom with the tasks.

An example of a mechanic I made is the big radial waves that the final boss in the game creates, as seen in the video (4:18)
{{< youtube id="_x_g0NE_mWE?start=258">}}

A big spreading wave is spawned when The Unchained slams into the water. I created this effect by spawning a tesselated plane with a special material for the duration of the wave. The material is using a simple sine function as a function of time and distance from the origin to calculate a world position offset value for the vertices, and to control the opaqueness. I also added a foam effect to the material by using textures, and spawned foam particle systems whenever the player goes through the wave. The wave is supposed to knock the player down on impact, unless if they dash through it or turn into stone form as in the video. To check for wave collision, I added a wave-height function into the wave blueprint that matches the one in the material that can then be called every frame. The wave shouldn't instantly knock down the player, so I also added a time-spent-in-wave threshold to it. If the player gets knocked down (see 4:30 in the video), an animation montage plays out that I combined from multiple animations where they roll on the water and rise back up. I was happy with how it all felt with everything combined!

I also need to mention the bear trap. When I joined the project, you could randomly get bitten by an almost invisible bear trap. They felt bad and I talked to the director about it. He agreed and was contemplating removing them, but I saw potential and took on the task to remake them and balance them in the level so they feel fair. The goal is to troll the player a little, and to make them keep track of another danger that lurks in the hostile world. It's a risky mechanic and if taken too far, people will hate, so it was important to balance it well.

{{< youtube id="9r2x4vcJV_k?start=51" >}}

I made the trap larger and tweaked their locations to be more visible in the level and added a subtle spotlight to them. I improved the animation transitions and the transform snapping, as it previously felt like you would get teleported in a disorienting way. I made the trap give very little damage so you'd most likely not die to it if you get bitten (though enemies could still attack you while trapped). As a fun bonus, I made it so that enemies can also get bit so you can lure them into traps if you want. So we kept the bear trap, and I don't feel bad for the evil I've caused :)
