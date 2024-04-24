---
date: 2023-07-20
title:  "The reference massacre"
description: ''
comments: true
math: true
featured_image: 'angry_angry_dad.png'
---

Quick test: Can you spot the bug in the following C++ code?

```
void spawn_explosion(GameWorld* world, const Vector3& explode_at, float size)
{
	for (int i = world->entities.count - 1; i >= 0; i--)
	{
		if (length(world->entities[i].position - explode_at) < size)
		{
			// This unlucky fellow was too close to the explosion
			array_remove(&world->entities, i);
		}
	}
	
	// spawn some epic particles, etc
	...
}
```

It's tricky, because by itself, there is no bug. But now consider the following:

```
void deal_damage_to_player(GameWorld* world, EntityID player_id) {
	Entity& player = world->entities[player_id];
	
	if (player.is_carrying_dynamite) {
		// Let's spawn an explosion at the player! That will also kill the player,
		// so we don't need to manually remove the player from the world.
		spawn_explosion(world, player.position);
	}
	else {
		player.hp--;
		if (player.hp <= 0) {
			array_remove(&world->entities, player_id);
		}
	}
}
```

See it yet?

The following code could crash the program. Let's say there are 100 entities in the world, and `deal_damage_to_player` gets called on the player, and the player happens to be the last element in the entity array (`player_id=99`). Then, `spawn_explosion` will be called with `explode_at` as the `position` field of the player. The loop will run, and the player, being very close to the explosion under its feet, will get removed from the array. Then, on the next iteration, another entity's distance to the explosion will be compared, and... *CRASH*

The program reads memory through the `explode_at` reference, but that memory is still pointing to the 99th element of the array, which is not part of the array anymore! Many different things could then happen depending on the array implementation, but likely (one would hope!) we'd get an assertion failure at least in development builds.

Worse, if instead of reverse-iterating through the entities, we iterated forwards and did `i--` after removing an element from it, and if the player was the *first* element in the array, the program wouldn't crash, but instead, the explosion would first happen at the player, then it would *move* to the next entity, then next, then next, ..., until all entities would be exploded. A true massacre!

This is not very good. One innocent "const &" made the entire thing explode, literally. Whether or not it makes sense to pass Vector3 by const& in terms of speed is not the point here, you can easily imagine similar situations with bigger/more complex types. The problem is that we're taught to use `const &` as an optimization to avoid a copy, and still think of it as just as a regular, but immutable, parameter. Most of the time this is fine, but not always - and then it can really bite you. It's still just a pointer, and you should think of it as such.

It's worrying to me that many new programming languages adopt this "const &" calling convention as *the default* and hide it from the programmer. Sure, on paper it might be sometimes faster. But usually when I'm passing a struct as a parameter, it's quite small - likely less than a few SIMD registers. And stack memory is so fast that it cannot truly be the problem, at least not in code which you're not optimizing under a microscope. As programmers we already have a lot of mental overhead, and turning *any parameter* into a read-only pointer means that you now need to also think about *everything* that you're possibly modifying either inside the function, or in any recursive function call, and if *any* of that could be passed into it as a parameter in *any* function call in your program, or by some other person who's calling your library function.

I'm calling it: those who decide const-reference-by-default is the right choice in their codebase or language design will hit a painful bug sooner or later.
