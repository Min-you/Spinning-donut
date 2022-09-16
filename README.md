# spinning-donut
it’s a framebuffer and a Z-buffer into which I render pixels. Since it’s just rendering relatively low-resolution ASCII art, I massively cheat. All it does is plot pixels along the surface of the torus at fixed-angle increments, and does it densely enough that the final result looks solid. The “pixels” it plots are ASCII characters corresponding to the illumination value of the surface at each point: .,-~:;=!*#$@ from dimmest to brightest. No raytracing required.

So how do we do that? Well, let’s start with the basic math behind 3D perspective rendering. The following diagram is a side view of a person sitting in front of a screen, viewing a 3D object behind it.

![perspective](https://user-images.githubusercontent.com/101444459/190595287-8069a4d7-9d55-48f4-b4d5-184f2b5b6181.png)

So to project a 3D coordinate to 2D, we scale a coordinate by the screen distance z’. Since z’ is a fixed constant, and not functionally a coordinate, let’s rename it to K1, so our projection equation becomes (x′,y′)=(K1xz,K1yz). We can choose K1 arbitrarily based on the field of view we want to show in our 2D window. For example, if we have a 100x100 window of pixels, then the view is centered at (50,50); and if we want to see an object which is 10 units wide in our 3D space, set back 5 units from the viewer, then K1 should be chosen so that the projection of the point x=10, z=5 is still on the screen with x’ < 50: 10K1/5 < 50, or K1 < 25.

When we’re plotting a bunch of points, we might end up plotting different points at the same (x’,y’) location but at different depths, so we maintain a z-buffer which stores the z coordinate of everything we draw. If we need to plot a location, we first check to see whether we’re plotting in front of what’s there already. It also helps to compute z-1 =1z and use that when depth buffering because:

z-1 = 0 corresponds to infinite depth, so we can pre-initialize our z-buffer to 0 and have the background be infinitely far away
we can re-use z-1 when computing x’ and y’: Dividing once and multiplying by z-1 twice is cheaper than dividing by z twice.
Now, how do we draw a donut, AKA torus? Well, a torus is a solid of revolution, so one way to do it is to draw a 2D circle around some point in 3D space, and then rotate it around the central axis of the torus. Here is a cross-section through the center of a torus:

