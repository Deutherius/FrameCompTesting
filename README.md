# Why do I even want to have frame expansion compensation on?
Quite simple - this is the same part, printed with the same settings. Both of these were printed from a cold start. The only difference is that while the right one was a solo part with a first layer time of ~1m45s, the left one was a part of a 7 hour full(ish) plate with a first layer time of more than 22 minutes.

![20210816_195655](https://user-images.githubusercontent.com/61467766/130360417-a3c08e91-1d5f-4a67-bde0-77ecef6e3ce1.jpg)

Frame heats up between first and second layer, expands, and Z changes by more than 0.1 mm, resulting in a second layer that is too far from the first. So I ran the measurement scripts, figured out that my endstop was faulty, changed it, ran the measurement scripts a second time, and figured out that my frame compensation coefficient is 0.0298 mm/K. But there is still a small problem (well, multiple)...

# A few tests with frame expansion compensation
Ever since building my V2 I've been fighting an inconsistent Z offset - first print of the day (cold printer start - heat up, soak a bit, QGL, 5x5 bed mesh) has a perfect Z offset. Every other print after that is too high with the same Z offset. Makes sense, the printer expands as it heats up - this is what the frame expansion compensation tries to counteract. However, as I understand it, frame comp works by taking a reference temp at the time of homing and works from there - if I re-home when the printer is hot (and at thermal equilibrium), frame comp basically does nothing. 
I talked to whoppingpochard about this, he is of the opinion that this is caused by the gantry bowing (imputed deflection, which is higher in the middle of the gantry/bed, where I usually print). I tried to verify this by printing two objects - one in the middle and the second near the endstop corner of the bed.

# First print
Cold start, QGL and bed mesh after a short soak, 2h print

![20210822_163323](https://user-images.githubusercontent.com/61467766/130359986-38c2738a-0c54-44e0-aa35-0edd0361148b.jpg)

Both objects have a good first layer (this is the squish I'm looking for in the test), no sign of the second/third layers being too far from the first, so that is great (means frame comp is doing what it should). However...

![20210822_163406](https://user-images.githubusercontent.com/61467766/130360041-27afc769-a14e-4e97-823c-70f751d84ff4.jpg)
![20210822_163358](https://user-images.githubusercontent.com/61467766/130360046-5235a03c-9d13-4e8d-8d1a-2b067c8de524.jpg)

The prints are weird and don't look good, there are artifacts and top layer looks overextruded. I think someone else mentioned a similar issue before and flow needs to be re-tuned after frame comp. But...

# Second print
Hot start, NOT re-homed and NOT re-meshed

![20210822_163423](https://user-images.githubusercontent.com/61467766/130360084-54dab6a1-e157-490d-bf7e-cdd71ae57df9.jpg)

Has a consistent squish at both center and edge of the bed. 

![20210822_163434](https://user-images.githubusercontent.com/61467766/130360100-89692f81-77fc-4e95-9efa-9f881e427493.jpg)

Both objects still have a bad top layer, BUT the bearing fit is nice, so the print itself is not overextruded, just the top layer.

# Third print
Hot start, NEW home, NO mesh at all

![20210822_170818](https://user-images.githubusercontent.com/61467766/130360212-106a1fc9-7ef7-4a28-aeee-7d941c968d7c.jpg)

Shows the outer print having a good first layer squish and the center one being too far. This is consistent with pre-frame comp operation (printer heats up, Z offset changes). Since the pritner was homed, frame comp took a new reference temp and didn't do anything during the print.

![20210822_170828](https://user-images.githubusercontent.com/61467766/130360240-079bbb7c-e733-4930-8925-faf823362ff0.jpg)

Outer print shows a good top layer, while the inner print has a weird line on the top (might be a fluke, but usually caused by slight overextrusion).

# Fourth print 
Hot start, NEW home and NEW mesh

![20210822_163518](https://user-images.githubusercontent.com/61467766/130360282-857dc124-089f-4962-8229-42a6b2bb0f38.jpg)

Is too far from the bed in both locations (consistent behaviour with the "bowl" bed mesh).

![20210822_163526](https://user-images.githubusercontent.com/61467766/130360319-63b76607-2ceb-4218-89d6-362b60c3f506.jpg)

None of the objects are overextruded. This is what I would normally see before activating frame comp. This is also the setup that I would guess most people use - QGL, mesh and home before every print.

# Conclusions
Whoppingpochard seems to be right - as the gantry bows, z offset at different parts of the bed changes. Bed mesh can counteract this, BUT takes the center point as the basis for the compensation, which in my case is too high.

I can see three options going forward, none of which are ideal:
1) Disable frame comp, return to the old ways (print single parts or very small plates to heat up the printer, adjust Z offset, print full plates)
2) Enable frame comp, take a QGL and mesh at a cold state (heated and soaked bed, cold frame) and then NEVER re-home or mesh again until the printer is cold again... And somehow solve the weird overextrusion-like artifacts. Easy.
3) Regardless of frame comp, just set the z offset for hot operation and let the pritner idle and heat up for hours before actually printing something. Fun.
