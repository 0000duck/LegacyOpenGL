# Design
Let's design a particle system. In general there are two ways to go about this. You can either design a base "ParticleSystem" class and then extend it into specific sub classes such as "FireParticles" or "SnowParticles" OR you can make a single mega particle system that exposes all sorts of properties such as absolute and relative velocity, then tweek these until your system is the way you want it.

As always, there is no right or wrong answer, what you use depends on what your needs are. Unity for example goes with the second route, the mega particle system. This is because they want the particles to be editable easily from the editor without code.

We on the other hand are not going to be shy about getting our hands dirty. We are going to design a particle system using the first approach. Where each new effect will take a little bit of code to create.

Lets take a bit of time and talk about things a particle system might need. Not everything we talk about on this page will make it into our implementation. Some of the stuff will be just for the purpose of discussion.