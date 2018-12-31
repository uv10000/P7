# **Highway Driving** 




**P7 Highway Driving Project**

Please refer to my githup repo [P7](https://github.com/uv10000/P7).

The goals / steps of this project according to the [rubric points](https://review.udacity.com/#!/rubrics/1971/view) are the following:

1 The code compiles correctly. 

2 The car is able to drive at least 4.32 miles without incident.

3 The car drives according to the speed limit.

4 Max Acceleration and Jerk are not Exceeded.

5 Car does not have collisions.

6 The car stays in its lane, except for the time between changing lanes.

7 The car is able to change lanes.

Details concerning these rubric points see below, after the "Executive Summary". 


[//]: # (Image References)

[image1]: ./examples/placeholder.png "Model Visualization"


#### 0. Executive Summary   

Please note my list of problems and open questions at the end of this summary.

My code is heavily based on Aaron's elegant and compact solution, plus only a few additional lines of code. Essentially the lines 265-303 in my main.cpp, and even those new lines are an adaptation Aaron's suggestions from the Q&A video.

To briefly summarize Aaron's idea: Use splines instead of quintic polynomials to predict (lists of) waypoints. More specifically Aaron chooses an appropriate spline to extend the sequence, or queue of waypoints into the future, boundary conditions are the tangent to the last two points in the list of waypoints, on the near end of the spline, and the center-line of the desired lane, on the far end of the spline. The geometric shape (the locus, mathematically speaking) is chopped into waypoints by taking into account the (desired) velocity. Only the first few of those waypoints, at the near end of the spline, are used to "fill up" the aforementioned list or queue of waypoints. 

Possibly as an advice to others: It took me a while to understand that the list of waypoints called "previous_path" returned from the simulator is simply the last list of waypoints as sent to the simulator one communication-step before, reduced by a few waypoints at the near end, that have actually been "processed" in the meantime (David Silver's Pacman-analogy was helpful for me). 

My code is is ok but not perfect, see detailed discussion below, yet it works surprisingly well, given the few lines of (extra) code. In most cases the car was able to drive far more than the 4.32 miles required, performing lane changes in a sensible way, and going at v_max for most of the time.

However this rather pleasing behaviour emerged only after a lot of parameter tuning and after introducing a debounce mechanism. So the behaviour is somehwhat fragile, see below. Even for that tiny model, and I suspect this would get (much) worse for a more complex model!

I know that I could further improve but I have spent far more than my time budget on this already. So please don't fail me because I could do better (of course provided you can live with this submission as it is). 



My single major generalisation of Aaron's Ansatz was to introduce three "lane-blocked" flags, one for each lane, not only one for the ego-lane alone.

The lines 289-303 exhibit stateful behaviour, implicitly encoding a state-machine. 

I programmed the car to drive "German style", that is: keep to the right whenever possible. Moreover it is not allowed to overtake on the right on German Motorways, but I relaxed  on the latter, as in your "American Highway style" traffic simulation slow vehicles clutter the leftmost lanes (which is ok by your rules, I know). If I had chosen not to overtake on the right at all, the ego-vehicle would be stuck most of the times. 

I suppose these German rules are related to the high spread in vehicle speeds due to the lack of a speed limit on the Autobahn. If there is a car approaching at 200 km/h on the leftmost lane, you do not want to a 120 km/h vehicle clutter that lane while all or some lanes to the right are free. As an aside, I observed during simulations that even with American speed limits in place, most of my remaining critical situation occured in the context of overtaking on the right.

It is nice to have such a very small state-machine (implicit in the aforementioned lines of code) handling things reasonably well.

In order to make it even better I could definitely extend the state machine. 
In my experience even small state-machines tend to exhibit unexpected/unintended behaviour, plus the complexity of state-machines grows quickly with size. Handling temporal logic well is hard for most humans I suppose. Irrespective of the description language chosen, see below. Adding "bells and whistles" may improve the behaviour but it may also introduce spurious behaviour in at least some of the many newly arising and hard-to-test situations.  

I found it beneficial to debounce things, calming things down. A pragmatic way to increase safety. I am quite confident that I could further reduce the number of incidents by defining "lane-blocked" in an even more conservative way for the respective non-ego-lanes. This would imply a reduced number of lane changes, but I think without loosing much performance in terms of going close to v_max speed most of the time in calm traffic. 

A few questions regarding state-machines, and deliberate rule-breaking:


- In the lecture you stated that using state-machines was somewhat outdated. Can you comment on this? Would you recommend to stick to a "classic Werling Ansatz" using state machines, in a (mainly highway-based) project starting today? Is this a play-safe approach or would you perceive this as outdated and obolete? What alternatives are there, what would you recommend otherwise?


- I have some experience with state-machines/state-charts. Do you really think that C++ is the optimum description language? I used Stateflow a lot and I could imagine that even today, despite of the C++ renaissance, that for the state-chart aspects of a SDC-implementation this might still be the description language of choice. However, and admittedly, even using Stateflow as the supposedly "right" description language did not protect me against problems with unexpected behaviour as described above. What do you think?

- More in the abstract, but essentially the same thing: can you comment and give advice on how to handle the aforementioned "complexity explosion" (as well as the fragility against small seemingly minor changes), possibly with an eye on formal automotive safety aspects. Ok regression testing, as you mentioned in the lecture. And possibly formal model checking methods but do you really believe in this?

- May I ask your advice on how to handle (sensible) deviations from the traffic rules. In Sebastian's "Junior" paper he described explicitly braking the rules under certain circumstances, eg crossing solid lines after having waited for a while. I think this is a very important aspect that is rarely discussed. Even if one *could* encode a behaviour compliant with the rules of the highway code perfectly in a state machine or whatever, the resulting driving behaviour would not be very good, sometimes idotic. Well, should you explicitly program that may you do certain things under certain circumstances, as long as there is no police officer watching? Humans do this, it is just common sense, isn't it? 

- A related aspect is that one should sometimes "do what the others do", i.e. learning exceptions to the rules "on the fly", by watching others. As an example consider giving way to an ambulance in a traffic jam. 

All comments welcome!






---

---



#### 1. The code compiles correctly.


Please refer to my githup repo [P7](https://github.com/uv10000/P7), cmake and make should work in the standard way, I tried this successfully from a "clean clone". 

Importang files are  src/main.cpp, this writeup file and a video. 

#### 2. The car is able to drive at least 4.32 miles without incident.
Most of the times I tried it definitely was, but there are occasional issues. Please see and judge for yourself.


#### 3. The car drives according to the speed limit.

The car is moving at 49.5 mp/h most of the time. (But whoever does in real live, see discussion about bending the rules above.)


#### 4 Max Acceleration and Jerk are not Exceeded.

They are presently not exceede, after following Aaron's concept of smoothing out velocity changes plus some debouncing. Before I introduced the 50 steps debounce counter to prevent lane-changes in close succession I observed the odd acceleration exeeded warning during a "double-lane-change". 

#### 5 Car does not have collisions.
In rare cases I had problems with other cars cutting in. As an improvement, as mentioned above, I am quite confident that I could reduce the number of incidents further by defining "lane-blocked" in an even more conservative way for the respective non-ego-lanes, without loosing much performance in terms of going close to v_max speed. 
On one single occasion, after more than 8 minutes of flawless driving my ego-vehicle cut off another vehicle, for no apparant reason. This rare but critical behaviour should be understood and fixed. However I have spent far more time than expected on this project and I would prefer to return to this at a later stage, whenever I find the time. 


#### 6 The car stays in its lane, except for the time between changing lanes.
Yes it does, by virtue of Aaron's cool spline mechanism. 


#### 7 The car is able to change lanes.


