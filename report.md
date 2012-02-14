# Dunking-Ratus

Our robot is called **Dunking-Ratus** and his goal is to grab balls and *"dunk"* them in a basket. To do so, we use three sensors and two motors :

  - *ultrasonic sensor* to measure distances
  - *color sensor* to identify balls
  - *compass* to orientate on the field

This little robot was made during our last semester in Eurecom and will compete soon against other robots like him !

![picture](images/photo.JPG)

# Strategy

Since a lot of the given sensors have great precision errors, we tried to keep our robot as simple as possible in its design, to not be stuck by to complicated algorithms paired with hazardous measures from our sensors.

During the initialization phase of the game, the robot records a *north* and *south* position to be able to find its way home. After that, it immediatly start finding a ball.

The way the robot move around on the field is by turning in right angles at rondom times or when facing a wall. This way, the robot travels in all the field space randomly. When it finds a ball, detecting it with its color sensor, the robot grabs it and raises it using its arm. Then it will find its way to the good basket using the *north* or *south* value depending on the color of the ball.

After repositionning in front off the baskets by measuring its distance to the wall with the ultrasonic sensor, the robot finally **dunks** it the basket (hopefully)

# Source code

# Team work

Our team is composed of three young and foolish boys : **Amaury Heackmann**, **Pierre Guilleminot** and **Alexandre Lachèze**. 

Our work was divided in four steps : mechanical build, strategy elaboration, implementation, testing. These steps were iterated several times to improve each of one.

Since our team is quite small, most of the time, we worked together on the same step.

When separated and to improve our productivity in a team environement we used the version control system [Git] and the hosting website [Github].

Finally, we also develloped this webpage as introduction material.


[Git]: http://git-scm.com
[Github]: https://github.com
