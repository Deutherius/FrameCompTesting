# Prolog
This mess of a page started as a way to gather my thoughts, experiments and results that aim to solve 2 problems - inconsistent second layer during longer prints, and inconsistent Z offset for subsequent print jobs after the first print of the day. First issue is caused by the frame expanding in the Z direction, and is covered by the Frame Expansion Compensation fork of Klipper. The second issue is caused by the gantry bowing as it heats up, which is corrected by a bed mesh, but under wrong assumptions about what is actually Z=0.

# Why do I even want to have frame expansion compensation on?
Quite simple - this is the same part, printed with the same settings. Both of these were printed from a cold start. The only difference is that while the right one was a solo part with a first layer time of ~1m45s, the left one was a part of a 7 hour full(ish) plate with a first layer time of more than 22 minutes.

![20210816_195655](https://user-images.githubusercontent.com/61467766/130360417-a3c08e91-1d5f-4a67-bde0-77ecef6e3ce1.jpg)

Frame heats up between first and second layer, expands, and Z changes by more than 0.1 mm, resulting in a second layer that is too far from the first. So I ran the measurement scripts, figured out that my endstop was faulty, changed it, ran the measurement scripts a second time, and figured out that my frame compensation coefficient is 0.0298 mm/K. But there is still a small problem (well, multiple)...

# A bit of info
Ever since building my V2 (300 mm spec) I've been fighting an inconsistent Z offset - first print of the day (cold printer start - heat up, soak a bit, QGL, 5x5 bed mesh) has a perfect Z offset. Every other print after that is too high with the same Z offset. Makes sense, the printer expands as it heats up - this is what the frame expansion compensation tries to counteract. However, as I understand it, frame comp works by taking a reference temp at the time of homing and works from there - if I re-home when the printer is hot (and at thermal equilibrium), frame comp basically does nothing. 
I talked to whoppingpochard about this, he is of the opinion that this is caused by the gantry bowing (imputed deflection, which is higher in the middle of the gantry/bed, where I usually print). I tried to verify this by printing two objects - one in the middle and the second near the endstop corner of the bed. Frame comp is on for all tests.

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

# Further testing - frame comp off
Just to make sure my printer is calibrated well, I turned off frame comp and printed the V0 bed mounts again from a cold start.

![20210822_203048](https://user-images.githubusercontent.com/61467766/130511740-0a012731-25d8-4c19-abea-bc882683ef66.jpg)
![20210822_203121](https://user-images.githubusercontent.com/61467766/130511755-89f01d6b-5826-4f1c-866f-fad9b97729d6.jpg)

Sorry for the potato image quality, it was very late. Left pieces are without frame comp and show zero artifacting... Except that the second layer is too far from the first due to the frame heating up and expanding during the print. So at least that is consistent.

# Further testing - lower expansion coefficient
After talking a bit with alch3my, I ran the same gcode again, this time with half my calculated frame expansion coeffient, i.e. 0.015 mm/K (from the previous 0.0298). Cold start.

![20210823_211303](https://user-images.githubusercontent.com/61467766/130512194-03eeeae3-058d-4c7b-9e91-f8a74a699a92.jpg)
![20210823_211258](https://user-images.githubusercontent.com/61467766/130512206-b78d2e47-f9fc-4a3e-aa1f-b785a6d1aeca.jpg)
![20210823_211230](https://user-images.githubusercontent.com/61467766/130512212-a50ad70e-500e-436e-af18-d6469fa02d2d.jpg)
![20210823_211202](https://user-images.githubusercontent.com/61467766/130512216-145bf6aa-4ef8-4234-8b7e-99075ffc4145.jpg)

The prints look better as far as number of weird artifacts go, but the first/second layer boundary is not great. This IMO points to the calculated coeffient being correct (and better accounting for the thermal expansion of the frame during the first layers). But the artifacts are still an issue..

# Further testing - Chirp... Chirp...

Ever since turning on frame comp, I noticed that my printer was a lot chattier - it would chirp loud and very often. I think I found the cause...

[![Frame comp...?](http://img.youtube.com/vi/pssEiFEH0Fw/0.jpg)](http://www.youtube.com/watch?v=pssEiFEH0Fw "Frame comp...?")

The video was shot near the end of the previous print. The Z constantly moves up and down, about 0.1 mm by my estimation (that large jump at ~5.5 seconds is a 0.4 mm z-hop). It kinda looks like frame comp applies its calculated offset, then the print returns to "normal", rinse and repeat. This could explain some of the artifacts on the previous prints - there are points where one layer looks to be missing (there is an indent that I can force my nail in - likely too high offset from the previous layer), followed by a bulging layer that looks overextruded (layer height returns to what it should have been), namely here

![20210823_211303_marker](https://user-images.githubusercontent.com/61467766/130513145-86f0a2a6-1f5c-4884-96e7-965c2ca4f546.jpg)

and here

![20210823_211202_markers](https://user-images.githubusercontent.com/61467766/130513158-9016882c-882f-47c0-b616-e94e554f8a81.jpg)

This is speculation of course, but there is no doubt something is seriously wrong with at least my setup, and potentially frame comp as a whole. More testing will follow soon...

# Chirp testing - data logging and new testing method (no need to wait for frame to cool down)

Talked a bit with Alch3my, he wrote a script that logs data during a print. Started from a cold frame, chirp was back even at 1.8 mm of Z height (same V0 bed mounts, 0.015 mm/K). I stopped the print and data collection, which showed nothing out of the ordinary (frame comp stable, frame temps stable) - something must be broken under the hood. 

![z_comp](https://user-images.githubusercontent.com/61467766/130668454-49407bbe-6a1e-4cc4-b393-7d14d8cd825f.png)
![temp](https://user-images.githubusercontent.com/61467766/130668456-44ff4234-1ff2-4dad-a01d-939916736c3e.png)

Alch3my suggested I try running a print and disable frame comp when chirping starts (probably should have tried that sooner, duh).

So I started the same print, this time from a warm frame state (reference frame temp 33.06 °C). Zero chirping at the same Z height as before (frame temp at some 34 °C, frame comp of -0.0120 mm). So I pulled the thermistor out and tried heating it up with my breath - no dice, wouldn't go above 34.5 °C and no chirping happened (probably not a bad ideato try a different thermistor as well). So I went the otehr direction and let the thermistor hang in the outside air, which quickly cooled it to 25.57 °C (frame comp of positive 0.1091 mm) - and chirping was quickly back. I then disabled frame comp (SET_FRAME_COMP enable=0), chirping went away and print progressed smoothly. Turn frame comp on, chirps are back. Shove thermistor back in the frame, chirps gone.

This indicates to me that as the value of z comp gets further from 0, things get unstable.

Bonus picture - this is what happens when you "cool your frame" too quickly:
![20210824_200734](https://user-images.githubusercontent.com/61467766/130668068-1436aa03-c8e6-497e-83dd-569352a72db8.jpg)

# Chirp testing - longer smoothing time

Alch3my suggested a longer than default smoothing time for frame comp, so I put in 10 seconds. Thermistor out, start the print, wait for a few layers (I like my PEI), thermistor back in. Instant chirping. Disable frame comp, chirping is gone.

# Chirp testing - no bed mesh

No difference. Bed mesh ain't it chief.

# Chirp testing - fresh klipper + frame comp install

[![Fresh klipper+frame comp install, frame comp on](http://img.youtube.com/vi/vhluh9-LW8Q/0.jpg)](http://www.youtube.com/watch?v=vhluh9-LW8Q "Fresh klipper+frame comp install, frame comp on")

[![Fresh klipper+frame comp install, frame comp off](http://img.youtube.com/vi/S9NDAJhQ_qw/0.jpg)](http://www.youtube.com/watch?v=S9NDAJhQ_qw "Fresh klipper+frame comp install, frame comp off")

That about concludes my testing of frame comp for now, I'm out of ideas. If you are not, feel free to contact me on discord at Deutherius#3295.
