{\rtf1\ansi\ansicpg1252\cocoartf1561\cocoasubrtf200
{\fonttbl\f0\fmodern\fcharset0 Courier;}
{\colortbl;\red255\green255\blue255;\red0\green0\blue0;}
{\*\expandedcolortbl;;\cssrgb\c0\c0\c0;}
\margl1440\margr1440\vieww10800\viewh8400\viewkind0
\deftab720
\pard\pardeftab720\partightenfactor0

\f0\fs26 \cf0 \expnd0\expndtw0\kerning0
CompSci 308: VOOGASalad Analysis\
===================\
\
> This is the link to the assignment: [VOOGASalad](http://www.cs.duke.edu/courses/compsci308/current/assign/04_voogasalad/)\
\
\
Design Review\
=======\
\
### Overall Design\
\
* High level design of each sub-part\
\
1. Game engine: \
\
	The game engine has the actor which is the basic unit of a game. It can be a tower, an enemy, a base, a projectile, any thing defined by the user. In any case, actor is basically a shell of any combination of properties. Properties are any behavior such as variations of shoot (shoot heat seeking, shoot near/far target) , move (move with a path, move by mouse) , health (limited health, infinite health) and so on. \
	\
	The grid hold info about every actor in the game. It has the ability to remove, spawn actor, and retrieve actor info. It contains the `step` method for the controller to call, which goes through every actor in the game, update actor, remove inactive ones. It calls `Actor.act()` from each actor, which goes through all of that actor's properties and call them to run by the method `public void action(G grid, Integer actorID)` from the  `IActProperty` interface. \
\
	The game controller instantiates the game by reading in game data, sets up the game loop, game screen and other components to operate a chosen game. In its `step` method it calls the level to update, listens to user input, and call the grid to step. \
\
2. Authoring environment\
\
	The authoring environment contains all the graphical elements for the user to make the game. Any change and user choice in designing a game will be saved by the authoring environment into different data classes such as path data, move data and later get comprised into a single class called game dat, which can be read by the game player. \
\
	The xml component of the authoring environment will save the current version of game data into an xml file (using Xstream) that could be later read and reconverted back by the game player. \
\
3. Game Player\
\
	The game player has two main components such as the game selector and the game screen. The game selector embeds xml file to each selection of game. As a file is chosen, the xml is read and loaded into a game data object that can be read by the game controller. The game screen displays everything that is happening in the backend while the game is played. Through multiple observers/observable pairs, it can update images and animation location from the backend grid. At the same time, the game screen has many event handlers for dragging in an actor, in the case of placing a tower for example, as well as for switching to xbox controllers, managing music player and upgrades. \
\
	The game player also has a user login, a rating, a leaderboard component for user to user customization. \
\
4. Game Data\
\
	The game data is the bridge between front end and backend. It contains factories to make data for every unit in the game. The factories utilizes reflection to make data objects per choices of the user. The game data depends on the user design of their own game in the authoring environment. \
			\
	Each property has its own data object, which contains different parameters (ex: speed and the path choice for move property). These parameters are filled in from the authoring environment, and the info of them is wrapped in the one data object that is passed to the constructor of its corresponding property for consistency and reflection purpose. \
\
	Once the game controller has a game data object, it has access to every thing in the game it needs to know ( level details, all the paths constructed etc) to run the game. \
\
* How is the design tied (or not) to the chosen game genre. What assumptions need to be changed (if anything) to make it handle different genres?\
\
	The design allows for all types of tower defense game to be made. The actor as a composition of properties makes it possible to build tower, enemy, and projectile. Through property like `ActorDamageable`, we can dictate what type of actor an actor can apply damage to. Thus, we can make a tower actor that can spawn a projectile actor that attack an enemy actor and at the same time make enemy actor to attack the tower actor. \
\
	The path and layer systems dictate the movement and restriction of where an actor can be placed on, so that we can restrict placement of tower on a path in Bloons, while allowing that for pea cannon in Plants vs Zombies. \
\
	Because of this, our program can already extend to accommodate other genre. We were able to made angry bird variation, space invader, centipede games out of our engine. This has been demonstrated in our group demos. \
		\
	However, the scrolling platform would require us to change the authoring environment to auto generate more space to build the game obstacle in any direction. At the same time, we would need to implement a camera utility in the backend that traces the movement of the main actor on the game display \
\
* What is needed to represent a specific game (such as new code or resources or asset files)?\
\
	A game can be represented by a single xml file. The images used for the games should be stored in the images folder, the music should be stored in the music folder of the repository. \
\
* Describe two packages that you did not implement:\
\
	`gameengine.grid` : this package contains classes relating to the grid structure of the game engine. \
\
	`gameengine.controllers` : this package contains the controller class that manage the game loop and play. \
\
	The grid package is the most readable because the classes are wrapped into sub-packages such as `interfaces`, `classes`, and `testers`, which make it easy to know what types of classes are held within them. Every class in this package has clear javadoc comments. The grid interfaces are clearly documented for every methods that show which specific methods each grid class can allow access to. For example, the `ReadableGrid.java` contains methods to read info of actor like their location without granting access to setters method . Every other grid interface extends this most basic grid and add some more methods, like `ReadAndMoveGrid.java` adds in a `move` method. This supports encapsulation of the grid methods to customize for the properties. ( a health property should not be able to have a grid that can move actor for example) \
\
	The `ActorGrid` class in that package that implements the grid interface also implements other interfaces and even extend the observable class. This shows the flexibility of multiple inheritance. \
\
	The `controllers` package contains the controller classes to run the games and manages levels. It is less documented and each class has more complex methods. It is still readable but not as much compared to the classes in the other package. \
\
	Reading their codes, I learnt that it is a good practice to use interface when you want to encapsulate and restrict access of methods of a certain component. Reading interface also helps understanding complex classes like that. From the controllers package classes, I learnt that it is good to break down and make methods that have descriptive names in the constructor of classes instead of a huge chunk of codes. For example, the constructor of Game controller is very easy to read: \
\
    ```java\
    public GameController(GameData gameData,LoginHandler loginHandler) \{\
    		myGameData = gameData;\
    		myGameObjectUtil = new GameObjectUtil();\
    		initializeUIHandler();\
    		initializeAnimationHandler();\
    		initializeGridHandler();\
    		initializeLevelHandler();\
    		initializeInitialGameStatus();		 tsetupGameStatus(loginHandler.getActiveUser(),myInitGameStatus);\
    		setUpGameScreen(loginHandler);\
    		....\
    	\}\
    ```\
	\
### Your Design\
\
* High level \
\
	The path and movement system: \
	\
	The user makes paths in the authoring environment, each path is a list of coordinates as waypoints. These paths are indexed and stored in the `gamedata.PathData`  class. \
\
	If an actor is assigned a move with set path property, it would need a move with set path data object, which takes in the list of paths assigned to the actor from `PathData.getAssignedPaths(List<Integer> options)` and the speed to move. In the move property class like `gameengine.actor.properties.MoveWithSetPathProperty` a path is randomly chosen for the actor from the list of paths assigned to it. It would call  `util.PathUtil.getPathCoordinates(List<Grid2D> pathChosen , double increment)` to compute the list of coordinates taking into account the speed. Each step will call the `action(G grid, Integer actorID` of the property to tell the grid to move the actor to a coordinate polled from that list. \
\
	The `PathUtil` comprises of static methods that any class especially the properties class would use to compute location/point related queries such as distance of two point, angle, list of points given an increments, check for a point is within a polygon and so on. \
\
	The layer system: \
\
	The user draws layers within the authoring environment and assign each actor to a layer where they can be put on. Each layer is a list of polygons whose shapes are defined by a list of points. Similar to path, all  layers  are stored in one class called `gamedata.MapLayersData`. \
		\
	When the player tries to place something on the game by dragging it in and right click at where they want to position it. the game controller calls the `addGameObject` method in the `util.GameObjectUtil` class. It will check whether the position chosen is within the layer assigned to the actor being added in, using `PathUtil.isWithinPolygon` method. If the position of placement is not in the appropriate layer, the actor will not be spawn by the backend and there will be an exception message saying `Invalid Location`. \
\
	I think my design for this path and layer system is good its flexibility really set us out from the other groups. \
\
	In the beginning, we debated about whether to use a tile/block system instead of xy plane to make game paths. The tile system would make it easy to defined a path (list of tiles) and restrict how many and what actor can be place on each tile. However, it will be hard to make path of any shape ( curvy, spiral, etc), not even mention the program will have to manage so many tiles if the user meant to make a game that accommodates a large number of actors (each tile would have to be really small in size). In addition, the projectile of bullets or movement of actor will look rigid ( as they would have to move from tile to tile). \
\
	For this, I proposed to use the xy coordinate system because it would be much more flexible.  It allows the user to draw the path how ever they want. Each path is a list of any number of waypoints so paths can have all shapes and quantity. Besides, this separated the graphical part of the game background from paths so user can load in beautiful background image for their game. To solve the restriction problem, we used the layer comprised of polygons which can have any shape to define area on the game map that each actor can be placed. For this, I got to implement ray casting algorithm to check for whether a point is within a polygon of any shape. \
\
	In terms of code design, the data classes are just data structures to store information about path and layer. The property class, such as `MoveWithSetPathProperty` implements the `IActProperty<G extends ReadableGrid>` interface for all properties, which has a `public void action(G grid, Integer actorID)` abstract method so that it will be called by the step function for all properties to act. \
\
	The grid `G` here can be any generic grid that extends the most basic ReadableGrid. For this specific property, `G` extends the `ReadAndMoveGrid` as it needs to tell the grid to move the actor to a new coordinate by every step and nothing else. This design of grid interfaces allows for encapsulation of methods as it restricts what a property can do depending on its need. \
\
    ```java\
    public class MoveWithSetPathProperty<G extends ReadAndMoveGrid> implements IActProperty<G>\{\
    	private Queue<Grid2D> myPathCoordinates;\
    	public MoveWithSetPathProperty(MoveWithSetPathData data)\{\
    		//Apply random path to current actor\
    		myPathCoordinates = new LinkedList<>(getRandomSteps(data.getMyPaths(), data.getMySpeed()));\
    	\}\
    	\
    	.....\
    	@Override\
    	public void action(G grid, Integer actorID) \{\
    		// TODO Auto-generated method stub\
    		if (!myPathCoordinates.isEmpty())\{\
    			// poll a coordinate from myPathCoordinates to set the enemy location to\
    			Grid2D newLoc = myPathCoordinates.poll();\
    			grid.move(actorID, newLoc.getX(), newLoc.getY()); \
    		\}\
    	\}\
    	...\
    	\}\
    ```\
\
    The feature that I felt need improvement is the `ui.Player.InGame.MusicPlayer`. The music player is instantiated in `ui.Player.InGame.GenericGameScreen`- the game player, which is instantiated by `GameController`. The order of call to load song and play music has a few steps, which require to paste in a music file path to create a Media object, and make a media player from that media object and call to play. I hid this order of call by making a `setSong(string song)` method in `GenericGameScreen` class that will be called by the controller in one line. \
	\
    ```java\
    myGameScreen.setSong(myGameData.getPreferences().getMusicFilePath()); //set music for game\
    ```\
    The game screen's `setSong` method would call the music player's own `setSong` method which has all the detailed implementation within it. \
\
    ```java \
    	public void setSong(String s) \{\
    		song = new Media(new File(s).toURI().toString());\
    		mediaPlayer = new MediaPlayer(song);\
    	mediaPlayer.setCycleCount(MediaPlayer.INDEFINITE);\
    	\}\
    ```\
	\
    This way i was able to hide implementation details that need not be presented in the Game Controller. \
\
### Flexibilty\
\
* Flexibility\
	\
    As explained in the design above, the path and layer system above lets the user define any quantity and shape for path and layer they want. There is no limitation to the way they set up their game map. At the same time, the using coordinates system instead of tile system makes our projectile and actor movement looks much smoother and more dynamically ( projectiles curving and tracing enemy for example) . Projectile, tower, any actor can operate on their own specified radius. Every thing is quite intuitive on the authoring environment. With mouse click they can create paths and polygons for layer in an instant\
\
	To prevent the user from making mistakes, I created default values for path if user assigns move property for an actor without making any path. There is also a default layer which is the entire screen if none is made by the user. \
\
* Two features I did not implement\
	 \
	The shoot property classes are good because they can accommodate so many variation of shooting functionalities such as shoot multiple, shoot heat seeking projectiles, shoot with mouse, shoot target nearest or farthest to the actor that has shoot property. \
\
	The design of these classes takes advantage of multiple inheritance. They has the same structure with the `action` method from `IActProperty` interface as do all properties, and inherits from a general `ShootTargetProperty` abstract class methods specific to shoot such as `spawnProjectiles` and `getEnemyToShoot`. As soon as it locates an enemy location, the shoot property uses `ReadAndSpawnGrid` to tell the grid to spawn projectile actor that goes towards the target coordinate. With this grid interface, the shoot property classes can only do basic functionalities of a `ReadableGrid` and call it to spawn a projectile but nothing else. In addition, these calls utilize the consumer design patterns well, making it extensible to any thing. Thus, we have shoot that listens to the mouse location dictated by the user. \
\
	The `ui.Player.InGame.Actor` class is the the front end actor representation. It takes information of the `ActorData` from the backend and build the graphical representation of the actor on the game player. This class is quite convoluted in the sense that it holds not only the graphical building methods but also event handlers for all that can happen to the actor. The poor design part in this class is that it can have the ability to tell the game controller to make another actor within an actor class. This is done by the `UIHandler` object handed to the Actor class that allows it to call `addGameObject` method. This is somewhat not very encapsulated design as this kind of event action should be handled externally to an Actor class. \
\
\
\
### Alternate Designs\
\
* Design change\
\
    There were two major changes:\
    \
    First: We went from using a tile system for our game map to a coordinate system. \
	\
	Second: We thought about making 4 base actor as separate entities such as enemy, tower, projectile, base in the beginning but then switched to make actor as a shell of a combination of properties. \
\
	The second design change was very good in my opinion. The original plan was that we made four separate entity abstract classes and made variation of each type by having concrete actor classes that extend those abstract classes. However, as we proceeded to write our classes, we recognized that we were writing duplicate codes, as all of the entity had some behavior in common , such as move for example. \
\
	Our decision to make actor as just a shell of property has many advantages. First, it prevents duplicate codes, since instead of writing concrete actor classes, we just need to write the behavioral property classes once, thus taking advantage of composition. Second, since an actor is comprise of any combination of property, the user has more freedom in building up their characters in their game. They can go as far as making an actor shoot projectiles that shoot projectiles themselves, or they can make actors of the same type able to damage each other.  Third, it promotes the use of method encapsulation of a generic grid. A grid interface is assigned to each property, so it cannot have access to methods of the grid that it should not and mess the game.This composition-encapsulation design definitely makes it possible for our program to make games that features tower defense  and beyond that: from PvZ, Bloons, Base Defense, to Space Invaders, Angry Birds, and Pong. \
\
### Conclusions\
\
* Best feature of the project's current design\
	\
	Pretty much what I mentioned in the Your Design and Alternate Design section. The composition of property and encapsulation of grid prevents duplicate coding and unsafe passing of methods. From reading and implement them, I learnt that it is beneficial to take advantage of the power of interface and composition, as they allow for extensible and flexible design of objects with similar or related behaviors. \
\
* The worst feature\
\
	Probably the classes of the game screen. Even though it has taken advantage of observable/observer pattern to connect with the backend, there are still a lot of orders of calls and unsafe passing of handlers of game controllers. Some component are instantiated in quite a few layers of structure (front end Actor object was instantiated within the sliding panel class, which was instantiated in a side menu class that was instantiated in the game screen) that makes the retrieval of or connection to these object quite difficult for the controller. From reading it, I learnt that it is beneficial to make every modular. Through interface or other pattern design, anything on the program should be accessed easily without getting through layers of classes. \
\
* Change in coding perspective and practice during the semester:\
	\
	My  biggest strength as a coder/designer is that I always ensure readability of code. I always used consistent coding convention, descriptive name, and broke down complex and long chunk of codes into smaller methods for easy reading. \
\
	At the same time,  I have a good grasp of interface and design patterns such as observable, consumer pattern, and have implemented those successfully through out the semester's projects. These patterns I learnt have helped me make my code more encapsulated and extensible to changes and specifications. \
	\
	My favorite part of coding/designing is the planning because it helps me learn how to think of good design to accommodate any possibility of future specification. After practicing for a while, I develop a general sense of what is flexible or rigid design, and how to improve it by utilizing the set of available design patterns. This is a very exciting intellectual process and I think I can even apply good design principle in almost every thing other than just computer science. \
\
	Two specific things I have done this semester to improve: I spent my spring break catching up on the readings about design given in class such as CodeSmell and design pattern page. I also read a case study paper on building a document editor, which helps me a lot in contributing on the composition of properties, path, and layer system in this program. I also asked non-compsci friends to use my program and took their feedback on user interface to improve them. I thought that feedback from user who is not familiar with design is most valuable to the programmer. I did that for Slogo and this program. Feedback on user experience was part of the reason I advocated highly for using coordinate system instead of tile system for the game map and that has proven to be the right choice.\
\
\
	}