![](https://github.com/agiraudet/boidsServ/blob/main/.demo.gif)

# boidsServ
A C.W.Reynolds boid's implementation in C++ with realtime controls.

The end goal is to provide an indpendant process capable of taking inputs (both for controls and querry),
 and outputing various informations about the boids in return.
 
## Controls
Currently, only four keyboard inputs are used as realtime-controls:
* __[A]__ Create a new flock off boids. Number and position are randoms, with numbers between 50 ans 500.
* __[S]__ Delete the mmost recently created flock.
* __[Q]__ Exit the program.
* __[SPACE]__ Allow/block the Sky from being updated

On input, the Scene object will call the function setup by the Scene::setInputFnct() hook. Change it to change controls (except [Q], currently hard-coded). Scene::setInputFnct() need to be passed a pointeur to a function with this prototype:
```
void  inputExample(void *arg, int key);
```
When called, this function will received the _arg_ pointer setup by the hook, and the key symbol, as defined by the "KeyCode" field by the SDL. [See more](https://wiki.libsdl.org/SDL2/SDL_Keycode)

You can also defined a function to be called at each new cycle, with this kind of prototype:
```
void  loopExample(void* arg);
```
This is done using the Scene::setLoopFnct() hook.
This is mainly used to gather informations about the current state of the simulation/act upon it. It might later gain an option to only be called once ever N cycle. We'll see.

## Dependecies
* [SLD2](https://www.libsdl.org/) is used for all video rendering, and input management.
* [CPTL](https://github.com/vit-vit/CTPL) is used for threadPooling (but is already included in the code as a header).
* [STK](https://ccrma.stanford.edu/software/stk/index.html) is used to managed everything sound related.

## Structure and how to use
The Sky class need to be instancied only once. It will manages all the flocks of boids.
Each Flock class holds N boids.
Currently, each Flock just ignore each other, but that will change in the future.
```
                  +-----+                       +-------+
                  | SKY |-------------------+   | SCENE |
                  +-----+                   |   +-------+
                     |                      |       |
       +-------------+-------------+        |   +-------+
       |             |             |        +-->|  SDL  |
  +---------+   +---------+   +---------+       |RENDER |
  | Flock 1 |   | Flock 2 |   | Flock 3 |       +-------+
  +---------+   +---------+   +---------+
    |             |             |
    +-[Boid1]     +-[Boid1]     +-[Boid1]
    +-[Boid2]     +-[Boid2]     +-[Boid2]
    +-[Boid3]     +-[Boid3]     +-[Boid3]
    +-[Boid4]     +-[Boid4]     +-[Boid4]  
```

## Exemple main.cpp
First thing needed is an instance of a Scene object (and only one !). If the constructor is givent a configuration file as parameter, it will try and use it as settings. Otherwise, default values are used.
You can then create a Sky object, and add/remove flocks to it as you see fit. Then, using Scene::mainLoop(Sky& sky), launch you simulation.
To modify controls, look at Scene::mainLoop() again.
```
#include "Scene.hpp"
#include "Sky.hpp"

int main(void)
{
  //Create a new Scene, using .conf as a config file. Must be done first.
  Scene sc(".conf");

  //Create a new Sky, which will hold ours Flocks of Boids.
  Sky sky;

  //Add a Flock of 500 Basic Boids (of random color) to our sky.
  sky.addFlock(500);

  //Add a Flock of 50 Predator green Boids
  sky.addFlock(50, "Predator", 0, 255, 0);

  //Apply a new RuleSet to the last Flock of the Sky (aka the predator's one).
  sky[-1].setRuleset(RuleSet(0.01, 0.1, 0.1, 1.0, 15.0, 30.0, 5.0));

  //Finally, launch the simulation.
  sc.mainLoop(sky);

  return 0;
}
```

## TODO next
* Get a better visual for the boids.
* Create more different type of boids with differents ruleset
* Create a proper control system (might be either via a GUI or a command prompt)
