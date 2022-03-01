---
hidden: true
---

# 3-Body Simulation

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


## Deriving the equation used for simulation

The following equations use i and j subscripts to represent 2 of the three bodies.
$$m_{i}$$ is the mass of the first body.
$$m_{j}$$ is the mass of the second body.
$$r_{ij}$$ is the distance between the bodies.
$$a_{i}$$ is the acceleration of the first body.

Combining the equations of motion we can derive the following equation to calculate the force exerted on one body by another body:

$$F_{i} = m_{i}a_{i} = \frac{Gm_{i}m_{j}}{r_{ij}^2}$$

This can be modified to use a vector for acceleration and distance between bodies by multiplying the right side by a vector of length 1 given by $$\frac{\vec{r_{ij}}}{r_{ij}}$$ and the acceleration on the left hand side then becomes a vector also:

$$m_{i}\vec{a_{i}} = \frac{Gm_{i}m_{j}}{r_{ij}^3}\vec{r_{ij}}$$

We can then also divide each side by $$m_{i}$$ to show the the mass of the first body does not contribute to it's own acceleration:

$$\vec{a_{i}} = \frac{Gm_{j}}{r_{ij}^3}\vec{r_{ij}}$$


