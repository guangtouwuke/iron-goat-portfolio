# Team Iron Goat
### Qihuan Aixinjueluo

In our process of developing the game Bulkhead, I am a member on the tech team and mainly responsible for implementing gameplay functions over the player’s character. As a first-year graduate student in IMGD, this is the first time I work with the Unreal engine. I have received great help from more experienced students Kai and John. For both of the development stages, Alpha and Beta, John divided the work into small goals for everyone on the tech side. So, my job is to try to achieve those goals and feedback my progress to the group. 

As for the challenges, before the alpha there was a bug. We expected our frost cannon could slow the enemies generally, which means the enemies would move slower and slower when being swept by the frost cannon and then come to a complete stop. However, in the real case, the enemies do not have the process of deceleration, they came to the complete stop immediately. 
![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic1.png)

This is the code fragment for the slow down part, every time the enemies being swept by the hit capsule created by the frost cannon will trigger the correspond enemy instance to call the SlowDown function, stacking up a SlowCount variable inside of the enemy instances. SlowCount is controlling the speed of the enemies. So, the bug appeared mostly likely because more than one SlowDown function was called when the enemies overlapped with the frost cannon capsule. And we tried to output the size of hitresult array, finding out there are indeed more elements there than we have expected (the collision preset of the capsule is set to ignore everything other than enemies). Therefore, there was a bug because the enemies’ skeleton has many parts, each part would trigger the SlowDown function to run once. As a result, the SlowDown function was called for over 40 times for every second of sweeping, causing the enemies to stop at once. And we solve the problem by adding a new collision named enemy capsule and set the collision capsule of the enemies to be the enemy capsule category and sweep only for the enemy capsule. In that case, every enemy would only be count once and decelerated generally.

Another problem we met right before alpha is that we cannot use debug capsule to draw the shape tracing capsule like we did for the line trace and draw debug lines because they are taking different arguments in their signatures while draw debug lines and line tracing both take starting and end point. 

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic2.png)
![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic3.png)

As we can see here, SweepMultiByChannel is a more generalized function and taking starting point, end point and the shape while the draw debug capsule is taking the specific capsule arguments: center, half height, radius and rotation. It might because I am not so familiar with the SweepMultiByChannel function. But I have tried many times, I could only use the DrawDebugCapsule to visualize the shape tracing inaccurately. I am not satisfied with the inaccuracy, so I asked John. John proposed a design of attaching a capsule onto the frost cannon all the time and change the visibility and the collision preset when fire start and fire end events are called. I realized and refined the design for the frost cannon. Here is the blueprint fragment.  

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic4.png)
![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic5.png)

For the start event, we need to deal with the snow count inside the character, so there are some previous nodes only allowing this to happen when there is positive snow count, and some subsequent nodes taking care of the snow count decreasing during the frost cannon shooting. In that case, the capsule for collision is perfectly overlapped with the drawn capsule because they are the same capsule. Also, this algorithm reduced the burden of changing the position and rotation of the capsule, we do not need to change it separately any longer. 

After Alpha, John said he was unsatisfied with the aiming experience. He thought in other shooter games, when the players start aiming, the gun should point forward by default. However, in our Alpha version, the frost cannon pointed to ground by default. Instead of just adjust the default position, we discussed the problem is that we did not have the crosshair. Lacking crosshair raised a lot of confusion about aiming. But that raised another problem, like real-life shooting, the sight is from a different starting point with the trajectory. Therefore, they can only perfectly overlap at an exact distance. Players are asked to aim from the camera position while our trajectory started from the frost cannon mesh and follow the direction of the camera. That algorithm did not work well the crosshair. So, here I worked backwards, instead of shooting to find the fire end point, I need to calculate the fire end point first then simulated the shooting behavior. To make sure the further vertex of the collision capsule or the fire end point always overlap my crosshair, I used the camera position add the weapon range * camera forward vector to get my end point. 

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic6.png)

Then, I draw the capsule between the end point and the start point which is still the gun mesh. In this case, the crosshair perfectly instructed where the gun is pointing to. There is a small tricky detail I encountered while realizing this algorithm. Because the end point is updated real time, so its distance from the starting point is not fixed, which means the size of the capsule need to be updated. The set capsule size function provided in Unreal takes two arguments: radius and half height. Unreal does not provide the document for what is radius and what is half height. 

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic7.png)
![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic8.png)

The second picture I am showing here is from the Nvidia doc. It seems that Unreal has a different definition on half height. In Unreal, the half-height finds out to include the radius. In another word, in Unreal, the distance between two vertexes is two half heights. 
Likely, I adjusted the aiming for the collecting snow function. Collecting snow does not require a visualization of the trajectory, so it is possible to use camera as the fire start point to simplify the math. If we use the gun muzzle as the start point, the end point can be below the ground, when the trajectory is not visible, it can seem dislocated like the following picture. 

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic9.png)

So, I choose to make camera the start point which can solve the problem and provide a more smooth experience of aiming. 

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic10.png)

I have moved my camera so that the debug line can be seen, we can see the trajectory is from a higher position instead of the gun. When we remove the debug line, here is the effect.

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic11.png)

Also, I want to share a little trick about my footstep event. On adding animnotify, a sound can be added and the animnotify can be called as event in blueprint in the animblueprint automatically. Sometimes, I want it can be also called in other blueprints such as character blueprints so that we can have more control over it and link animnotify to logical manipulation. What I did is create an animnotify blueprint and a blueprint interface in the content explore.

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic12.png)

This is what happened in the interface.

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic13.png)

Then we link the interface in the animnotify blueprint instance.  

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic14.png)

Then, I put the notify in the responding animation. Finally I implent the interface in any of the blueprints. 

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic15.png)

The following digram is about all the character potention behaviours in the gameplay until Beta. The blue nodes are tasks I am partly contributed to and the purple nodes are the tasks I mainly responsible for. 

![alt text](https://github.com/guangtouwuke/iron-goat-portfolio/blob/master/pic16.png)

So in this process, what I learnt from all the challenges is that the algrothms we use is very flexiable. It is possible for us to use different apporches to solve specific problems. Like the snow collection I mentioned above, a better way should be looking for where the line trace hit the ground from the hit result and use the location as the fire end point and use the muzzle as the fire start point. I think if the trajectory is not visible, just using the camera as fire start point can simplify the problem. Also, I think try to play with the functions more should also be a better way of sloving problems. Just like the capsule half height problem I encountered, this could be hard to debug with out enough document, the only way is that play with the numbers so that you might feel something unnatural there. 
I think being patient is also a very important point during this process. I have no previous experience with Unreal engine before, and during the process, many functions have been rewritten for a better performace. At the very beginning of the program, I can just repeat and learn from what Kai and John has already made. So, there can be very little sense of accomplishemt and may casue frustration. But I feel better since I can do something useful for the group. 
Also I think an important part is finding what is actully going wrong during the process. Learning from existing games is usually a good way. Like John was unsatisfied with the aiming. But it turned out can be resolved with adding a crosshair, John might not feel the problem is related to the crosshair at the very beginning. So we asked him what he think is a good example, and we watched the gameplay together to find out what we need to do next. 
We used github for version control. Due to Kai’s earier experience about branches, we decided to use the same branch all the time. That casued a big problem that no more than one people can make adjustment to the game. If I pulled the version and before I adjusted and pushed it to the git, someone else pushed another version, I would not be able to push my version of game. This version control method is a little bit cumbersome because sometimes we need to figure out what to do first, some times copy the code we wrote, then pull the newest version and make adjustmenst as fast as possbile and push it back on. But generally, this worked well. We did not experience big progress loss. Some of our team are in China and some of us are here in worcester. John likes to work in the early morning while Kai and I sleep really late. Sometimes I hand the game to John when he woke up and then I went sleep. Another thing is all of our tech team respond to messages very quick so we can warn each other before we make changes. 
But we can also see this choice is not possible for professional teams with a bigger size or work generally in the same period of time. We do need to improve our way of doing version control when we work on it more during summer. 
