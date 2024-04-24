---
date: 0000-00-05
title:  "Experiments with Constructive Solid Geometry"
description: ""
comments: true
math: true
featured_image: 'csg_editor_banner2.png'
---

An exploration into constructive solid geometry techniques with the goal of making new kinds of level design & 3D art tools.

<!--more-->

## Demos

{{< youtube kCL9DabX_wk >}}
*3D Constructive Solid Geometry with boxes*

{{< youtube QbqmdJI531A >}}
*2D Constructive Solid Geometry with polygons*

## About

The motivation for this project came from ideas I had for a new kind of 3D modeling tool. Having CSG / mesh boolean operations as the basic editing operations in a 3D editor seemed like a good idea, as they're really intuitive and easily compose into more complicated shapes. The sculpting tools of [Dreams](https://youtu.be/v27LZivpffQ) were a big inspiration to me, and their idea of [sensitive tools](https://youtu.be/DpSWMQgBpHI).

The approach Dreams uses for implementing adding/cutting operations is to use signed distance functions. SDFs are nice and simple to implement, but when the list of shapes gets large, rendering it naively with ray marching would be too expensive to do in real time. Dreams has an advanced system to hierarchically convert from the SDFs into point clouds using compute shaders and a special rendering pipeline to draw them. However, I wanted to stay in the realm of triangle meshes, as that's what game engines typically work with and my goal wasn't to have the painterly art style of dreams which would require very special rendering techniques.

One option would be to convert from an SDF into a triangle mesh using marching cubes, dual contoring, or similar. Alex Evans goes through various approaches for this in his [Dreams R&D talk](https://youtu.be/u9KNtnCZDMI). The problem is that the resulting mesh needs to be really dense to keep the geometric detail, making it slow to generate and to render. The generation time is also linearly proportional to the SDF complexity, so over time editing would become slower and slower. My goal is to have a very fast algorithm to let users edit shapes in real-time and see the boolean results immediately (no matter how long they work on their model), so meshing SDFs didn't seem like the best alternative.

My best option seemed to be to look into 3D mesh boolean & CSG algorithms. I braced myself assuming it would be quite hard as they have a reputation of being fragile in most 3D software. Especially in edge cases with shapes barely touching, things often break. There are some libraries that can perform mesh booleans perfectly robustly, but they seemed too slow for real-time editing, so I chose to dive deeper into this problem myself. I was also just curious!

## Initial attempt

I took inspiration from a few papers, i.e. [Simple and Robust Boolean Operations for
Triangulated Surfaces](https://arxiv.org/pdf/1308.4434.pdf). I initially had the idea to use exact integer arithmetic, where a fractional number would be stored as two 64-bit integers. Integers would guarantee robustness, as opposed to floats which introduce small errors that could break the algorithm in edge cases. The problem with exact arithmetic is that when multiplying two numbers, the number of bits required for storing the value doubles. The algorithms often require using dot products, determinants or similar, which could bloat up the coordinates, making computations more expensive after each operation. Instead, I decided to try making an algorithm using floats and a careful use of epsilons.

The basic idea of most mesh boolean algorithms is to cut each polygon in mesh A by each polygon in mesh B, then for each formed subpolygon determine if it's inside mesh A and B, and based on those 2 inside/outside values and the boolean op (subtract/union/intersect), either discard it, keep it, or flip it. I learned about the half-edge data structure, which was useful for traversing the mesh in various ways, as well as to insert edges/vertices/etc. To make the algorithm efficient when looking for intersection pairs and for ray-casting (inside-outside detection), a spatial data structure can be used, such as a grid or an octree. I got the algorithm to be quite robust, but there were still some failure cases which I failed to get rid of. Maybe with slight changes to the algorithm, I could have made it work robustly, but with my knowledge back then I didn't believe so.

{{< youtube 0d8rbbI6TLU >}}
*Box & box intersection using floats. It still appears to work pretty well.*

## Second attempt

After a break from this project, I got back and found some more promising papers, such as [Fast Exact Booleans for Iterated CSG using Octree-Embedded BSPs](https://arxiv.org/abs/2103.02486) and more recently [EMBER: Exact Mesh Booleans via Efficient & Robust Local Arrangements](https://www.vci.rwth-aachen.de/publication/03339/), both by the same authors. Interestingly, the papers use exact integer arithmetic, but manage to avoid the problem I described before. The main idea is to represent all geometry using planes, where a plane is defined as the four fixed-size integer coefficients `A, B, C, D` of the plane equation `Ax + By + Cz + D = 0`. Edges are represented as the intersection of two planes, points by the intersection of three planes, and (convex) polygons by a plane and a list of edge planes. A nice property of boolean operations is that the result of a boolean is fully made up of polygons from either of its input meshes, possibly flipped or cut by each other's planes. It's then possible to represent the boolean result mesh using only the original planes from the two input meshes. That's perfect, as we can apply boolean operations as many times as we want without growing the bit complexity or losing any information! The coefficients will always stay within a fixed range.

After some trial and error, I followed a similar approach to the first paper. My approach was to keep a BSP tree with each leaf cell being either solid or empty, and to have an operation to convert it to a half-edge boundary representation and vice versa. To perform a boolean, mesh A's BSP tree would be cut by the faces of mesh B, then for each leaf BSP cell in A, determine if it's inside mesh A and B, and depending on the boolean op make it either solid or empty. Then, convert the BSP tree of A into a boundary mesh, and to a triangle mesh, and render it using a traditional GPU pipeline. A nice bonus about doing the round trip from BSP -> boundary mesh -> BSP is that it automatically cleans up the BSP tree from redundant splits, keeping it as simple as it can be.

{{< youtube TDFQl0qRzqs >}}
*Importing a mesh into my program which performs a boolean op and exports it, then importing it back into blender.*

{{< youtube _NOYwEfi6w4 >}}
*Rectangle selections can be implemented using the GPU by keeping track of the shape ID per each vertex, rendering the shape ID to a separate render target, and running a compute shader over it to build a list of shape IDs that are within the current selection rectangle.*

## The future

Right now, I'm looking further into a method for doing mesh booleans in a way where self-intersections are supported (as opposed to having 2 non-self-intersecting input shapes at a time). This would allow for fun things like [Sketchup-style extrusions](https://blender.stackexchange.com/questions/228336/correct-way-to-extrude-inward-intrude), and would generally be more robust when introducing more general polygonal modeling tools. I've started prototyping it in 2D this time and I'll definitely keep working on it to bring it into 3D and to finally start making some better modeling tools for it as well :)

{{< youtube pnkp85K4KR8 >}}