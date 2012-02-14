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

After repositionning in front off the baskets by measuring its distance to the wall with the ultrasonic sensor, the robot finally **dunks** it the basket (hopefully).

# Team work

Our team is composed of three young and foolish boys : **Amaury Heackmann**, **Pierre Guilleminot** and **Alexandre Lachèze**. 

Our work was divided in four steps : mechanical build, strategy elaboration, implementation, testing. These steps were iterated several times to improve each of one.

Since our team is quite small, most of the time, we worked together on the same step.

When separated and to improve our productivity in a team environement we used the version control system [Git] and the hosting website [Github].

Finally, we also develloped this webpage as introduction material.


# Source code


```c
#define NOCOLOR 0
#define RED 1
#define BLUE 2
#define NEAR 15
#define RIGHT 0
#define LEFT 1
#define RANDOM 2
#define SPEED 100
//#define MIDDLE
//#define E 0;
//#define W 1;
#define N 1;
#define S 2;

int COLOR=NOCOLOR;
int NORTH=-1;
int EAST;
int WEST;
int SOUTH=-1;
int DIRECTION;
int READY2DUNK=0;
int DUNKED=0;
mutex Clock;
mutex R2Dlock;
mutex Dlock;
task findBall();
task dunk();
task randomWalk();
task walkToBasket();
task victoryLapDance();
task debug(); //A FAIRE


int init()  //initialise les capteurs et le generateur aleatoire
{
	SetSensor(S1,SENSOR_TOUCH); //Configure l'entree 1 en touch sensor
	SetSensorLowspeed(S2); //Configure l'entree 2 en light sensor
	SetSensorLowspeed(S3); //Configure l'entree 3 en compass sensor
	SetSensorLowspeed(S4); //Configure l'entree 3 en ultrasound sensor
}

safecall int getCompass()  //acces protege au compas
{
	return SensorHTCompass(S3);
}


safecall int getTouch()
{
	return Sensor(S1);
}

safecall int getUltraSound()  //acces protege a l'ultrason
{
	return SensorUS(S4);
}


safecall int getColor()
{
	byte red;
	byte blue;
	byte green;
	byte colorNum;
	ReadSensorHTColor(S2,colorNum,red, green, blue);
	if(red>120)
	{
		return RED;
	}
	else
	{
		if(blue>120)
		{
			return BLUE;
		}
		else
		{
			return NOCOLOR;
		}
	}
}


void turn(int dir)  //quart de tour (aléatoire, ou pas)
{
//int ref=getCompass();
	int rightORleft = (dir==RANDOM) ? Random(2) : dir;
	if(rightORleft==RIGHT)
	{
//while(getCompass()<ref+70)
		{
			OnFwd(OUT_A, 35);
			OnRev(OUT_C, 35);
			Wait(2800);
		}
		Coast(OUT_AC);
	}
	else
	{
//while(getCompass()>ref-70)
		{
			OnFwd(OUT_C, 35);
			OnRev(OUT_A, 35);
			Wait(2800);
		}
		Coast(OUT_AC);
	}
}



void reposition(int dest, int speed)
{
	if(getCompass()>=dest)
	{
		while(getCompass()>=dest)
		{
			OnFwd(OUT_C, speed);
			OnRev(OUT_A, speed);
			Wait(100);
		}
	}
	else
	{
		while(getCompass()<=dest)
		{
			OnFwd(OUT_A, speed);
			OnRev(OUT_C,speed);
			Wait(100);
		}
	}
	Coast(OUT_AC);
}

void initCompass()
{
	Wait(10000);
	SOUTH=getCompass();
	turn(LEFT);
	turn(LEFT);
	Wait(10000);
	NORTH=getCompass();
}


task randomWalk()  //random walk
{
	int color=NOCOLOR;
	int timer=Random(24)+8;
	Acquire(Dlock);
	DUNKED=0;
	Release(Dlock);
	OnFwd(OUT_AC,SPEED);

#ifdef MIDDLE
	Coast(OUT_AC);
	Wait(4500);
	reposition(WEST,30);
	reposition(WEST,20);
	DIRECTION=W;
	OnFwd(OUT_AC,SPEED);
#endif

	while(color==NOCOLOR)
//while(true)
	{
		if(getUltraSound()<NEAR||timer==0)  //si il y a un obstacle
		{
			Coast(OUT_AC);          //arrete d'avancer
			turn(RANDOM);
			timer=Random(24)+8;
			OnFwd(OUT_AC,SPEED);       //redemarre
		}
		timer=timer-1;
		Wait(250);             //tempo
		Acquire(Clock);
		color=COLOR;
		Release(Clock);
	}
	ExitTo(walkToBasket);
}

task walkToBasket()
{
//TODO
	int dir;
	int col;
	Acquire(Clock);
	col=COLOR;
	Release(Clock);
	if(col==RED)
	{
		dir=NORTH;
	}
	else
	{
		dir=SOUTH;
	}

	reposition(dir,50);
	Wait(1000);
	reposition(dir,30);
	OnRev(OUT_AC,SPEED);
	while(getTouch()!=1)
	{
		Wait(200);
	}
	Coast(OUT_AC);
	OnFwd(OUT_AC,40);
	Wait(1000);
	Coast(OUT_AC);
	turn(LEFT);
	Wait(2000);
	if(getUltraSound()>45)
	{
		OnFwd(OUT_AC,40);
		while(getUltraSound()>47)
		{
			Wait(200);
		}
	}
	else
	{
		OnRev(OUT_AC,40);
		while(getUltraSound()<47)
		{
			Wait(200);
		}
	}
	Coast(OUT_AC);
	turn(RIGHT);
	turn(RIGHT);
	Wait(2000);
	if(getUltraSound()>45)
	{
		OnFwd(OUT_AC,40);
		while(getUltraSound()>47)
		{
			Wait(200);
		}
	}
	else
	{
		OnRev(OUT_AC,40);
		while(getUltraSound()<47)
		{
			Wait(200);
		}
	}
	Coast(OUT_AC);
	turn(LEFT);
//reposition(dir,20);
//Wait(1000);
//reposition(dir,20);
//Wait(1000);
	OnRev(OUT_AC,SPEED);
	while(getTouch()!=1)
	{
		Wait(200);
	}
	Coast(OUT_AC);
	OnFwd(OUT_AC,40);
	Wait(1000);
	Coast(OUT_AC);

	Acquire(R2Dlock);
	READY2DUNK=1;
	Release(R2Dlock);



/*reposition(NORTH,20);
Wait(1000);
reposition(NORTH,20);
OnRev(OUT_AC,SPEED);*/

	int dunk=0;
while(dunk==0)
{
	Wait(500);
	Acquire(Dlock);
	dunk=DUNKED;
	Release(Dlock);
}
Acquire(Clock);
COLOR=NOCOLOR;
Release(Clock);
//ExitTo(victoryLapDance);
ExitTo(randomWalk);
}

task victoryLapDance()
{
	OnFwd(OUT_AC, SPEED);
	Wait(5000);
	Coast(OUT_AC);
	RotateMotor(OUT_B, -50,90);
	OnFwd(OUT_B,-10);
	OnFwd(OUT_A, 75);
	OnRev(OUT_C,75);
	Wait(10000);
	Coast(OUT_ABC);
	ExitTo(randomWalk);
}


task findBall()
{
	int col;
//start randomWalk;
	while(true)
	{
		col=getColor();
		if(col!=NOCOLOR)
		{
			Acquire(Clock);
			COLOR=col;
			Release(Clock);
//stop randomWalk;
			RotateMotor(OUT_B, -50,90);
			OnFwd(OUT_B,-10);
			break;
		}
//Wait(100);
	}
	ExitTo(dunk);
}

task dunk(){
//TODO
	while(true)
	{
		Acquire(R2Dlock);
		if(READY2DUNK==1)
		{
			break;
		}
		Release(R2Dlock);
		Wait(200);
	}
	Release(R2Dlock);
//Coast(OUT_AC); //A deplacer dans R2D ac getTouch (walkToBasket)
	RotateMotor(OUT_B, -80,100);
	Wait(2000);
	RotateMotor(OUT_B, 40,180);
	Off(OUT_B);

	Acquire(Dlock);
	DUNKED=1;
	Release(Dlock);

	Acquire(R2Dlock);
	READY2DUNK=0;
	Release(R2Dlock);

	ExitTo(findBall);
}


task debug()
{
	while(true)
	{
		TextOut(0, LCD_LINE3, "Compas");
		TextOut(0,LCD_LINE4, "     ");
		NumOut(0,LCD_LINE4, getCompass());
		/*TextOut(0, LCD_LINE3, "UltraSound");
		TextOut(0,LCD_LINE4, "     ");
		NumOut(0,LCD_LINE4, getUltraSound());
		TextOut(0, LCD_LINE5, "Color");
		TextOut(0,LCD_LINE6, "     ");
		NumOut(0,LCD_LINE6, getColor());*/
			TextOut(0, LCD_LINE1, "NORTH");
		TextOut(0,LCD_LINE2, "     ");
		NumOut(0,LCD_LINE2, NORTH);
		/*TextOut(0, LCD_LINE3, "WEST");
		TextOut(0,LCD_LINE4, "     ");
		NumOut(0,LCD_LINE4,WEST);*/
			TextOut(0, LCD_LINE5, "SOUTH");
		TextOut(0,LCD_LINE6, "     ");
		NumOut(0,LCD_LINE6, SOUTH);
		/*TextOut(0, LCD_LINE7, "EAST");
		TextOut(0,LCD_LINE8, "     ");
		NumOut(0,LCD_LINE8, EAST);*/

			Wait(500);
	}
}


task main()
{
	init();
	StartTask(debug);
	initCompass();
	StartTask(findBall);
	StartTask(randomWalk);

}
```



[Git]: http://git-scm.com
[Github]: https://github.com
