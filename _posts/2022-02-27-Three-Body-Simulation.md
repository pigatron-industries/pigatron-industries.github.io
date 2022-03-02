---
hidden: true
---

# Three Body Simulation

This is how I went about writing the code to simulate 3 planets based on Newtons laws of motion.
The resulting code was the basis of a eurorack hardware module which outputs the position of each planet as an X and Y voltage.


## Newton's equations of motion

The 2 equations that we will need are, Newton's second law:

$$F = ma$$

and, Newton's equation of universal gravitation:

$$F = \frac{Gm_{1}m_{2}}{r^2}$$


## Coordinate system

We will simulate the 3 bodies in 2 dimensions. The properties of each body, such as position, velocity and acceleration, will be represented by a 2D vector.

The origin of the system will be located at the centre of mass of the three bodies. This is becasue the centre of mass of any number of bodies always remains motionless.

![Three body system coordinates](/assets/images/three_body_simulation_coordinates.drawio.png)

## The maths

The following equations use i and j subscripts to represent 2 of the three bodies.

- $$m_{i}$$ is the mass of the first body.
- $$m_{j}$$ is the mass of the second body.
- $$r_{ij}$$ is the distance between the bodies.
- $$a_{i}$$ is the acceleration of the first body.

Combining the equations of motion we can derive the following equation to calculate the force exerted on one body by another body:

$$F_{i} = m_{i}a_{i} = \frac{Gm_{i}m_{j}}{r_{ij}^2}$$

To make acceleration a vector we can multiply the right hand side by a vector of length 1 representing the direction of body 2 in relation to body 1. To get a vector of length one we divide the vector by its length:

$$\frac{\vec{r_{ij}}}{r_{ij}}$$

Combining with Newtons's equations gives:

$$m_{i}\vec{a_{ij}} = \frac{Gm_{i}m_{j}}{r_{ij}^3}\vec{r_{ij}}$$

We can then also divide each side by $$m_{i}$$ to show the the mass of the first body does not contribute to it's own acceleration:

$$\vec{a_{ij}} = \frac{Gm_{j}}{r_{ij}^3}\vec{r_{ij}}$$

This gives the acceleration of one of the bodies caused by one of the other bodies. To get the accelaration of one body caused by the other two bodies we can simply add them together:

$$\vec{a_{1}} = \frac{Gm_{2}}{r_{12}^3}\vec{r_{12}} + \frac{Gm_{3}}{r_{13}^3}\vec{r_{13}}$$

### Euler method

From Newton's equations, we can calculate an acceleration based on the positions of the bodies. We need to turn this into a velocity and a position and to do this we can use Euler's method for solving differential equations. It uses a series of time steps to calculate the positions at certain points in time. It is the simplest way to approximate differential equations but also has the largest error, which is proportional to the size of the time step used.

It can be applied to the previous velocity and accelaration by multiplying by the time step size, $$\delta t$$:

$$\vec{v}_{1} = \vec{v}_{0}+\vec{a} \delta t$$

$$\vec{p}_{1} = \vec{p}_{0}+\vec{v} \delta t$$

There are more accurate methods such as the Runge-Kutta method, but this is also more complicated and would require us to apply Newton's equation multiple times for each sample.

## The code

We will use a vector class to store the various properties of a body and a Body class to store the current state. A Body has three variable properties; position, velocity, acceleration and one fixed scalar property; mass:

The class looks like this:

``` cpp
class Body {
    public:
        float mass = 1;
        Vector<2> position;
        Vector<2> velocity;
        Vector<2> acceleration;
};
```

The main simulation class called ThreeBody needs to have an array of 3 bodies and a time step variable:

``` cpp
class ThreeBody {
    ...
    private:
        Body bodies[3];
        float dt;
};
```

The bulk of the work is done in the calculateAcceleration function. This calculates the acceleration for a single body, the index being passed in as a parameter.
It loops through the other bodies, and implemets Newton's equations to add each acceleration to the current bodies acceleration:

``` cpp
void ThreeBody::calculateAcceleration(int i) {
    bodies[i].acceleration = Vector<2>(0, 0);
    for(int j = 0; j < BODIES; j++) {
        if(j != i) {
            Vector<2> r = bodies[j].position - bodies[i].position;
            float dist = r.length();
            bodies[i].acceleration += r * ((G * bodies[j].mass) / (dist*dist*dist));
        }
    }
}
```

Note the use of a Vector class with overriden operators means the calculation using vectors looks a lot easier to read han if e ere dealing direclty with x and y coordinates.

To put it all together, the main process function calls this for each body

``` cpp
void ThreeBody::process() {
    for(int i = 0; i < BODIES; i++) {
        calculateAcceleration(i);
    }
    ...
}
```

Then it applies Eulers method to each body by multiplying acceleration by the time step, then multiplying velocity by the time step to get the position.

``` cpp
for(int i = 0; i < BODIES; i++) {
    bodies[i].velocity += bodies[i].acceleration * dt;
    bodies[i].position += bodies[i].velocity * dt;
}
```

