---
title: "My God, It’s Full Of Stars: Testing Point Mass Attraction with Catch2"
description: 
date: 2019-02-28
tags:
  - c++
  - n-body
  - catch2
  - tdd
  - testing
layout: layouts/post.njk
image: /img/particleBackground.jpg
---

Welcome back to the n-body-problem series. Last time we've been setting up our project structure and CMake setup. According to some helpful [reddit][1] comments, I adapted our CMake setup a bit, but I'm still not absolutely sure if the project structure, as it is now, will last to the end of the project. If you've missed any of the earlier posts, you can find them here:

![Hero Image: Particles](/img/particleBackground.jpg)

1. [My God, it’s Full Of Stars: The n-body-problem series][2]
2. [My God, it’s Full Of Stars: And then there was CMake][3]

Because we want to exercise [TDD][4], we somehow need to define some test cases first before we can implement our solver library. A simple scenario would be two points with an equivalent mass $M$, a distance of $r$ and no initial velocity and acceleration. Clearly we could do hundreds of different complex tests with it. But i would like to start with two very simple and specific tests, point masses accelerating and moving only in x- and y-direction sole. This way we can check that, at least the basis, assumptions, formulas, and implementations are correct.

We have also to think about an anomaly of the equation of gravitational force, which we use to calculate the acceleration. What happens in the case of two masses are reaching the point of singularity, in other words, collide? If two points are at the same position in space, the distance r between both points is zero. If this happens, the accelerating force $F$ is getting infinite, and therefore the acceleration of both points does. Both points would fly apart from each other into the void.

There are basically two ways for us to solve this issue. The first solution would be to merge both masses into one. Every fan of black holes would probably agree with this solution, but there is on small problem with it. What happens to [merging stars][5] is absolutely not understood well and would be therefore hard for us to model. Clearly, our model is not perfect and has a lot of flaws, but model merging stars would blow up its complexity. The second solution is rather simple, we never allow masses to collide. This can be simply done by adding an error term $\epsilon$ to our gravitational force. Additional we have to add $(r_{j} - r_{i})$ to get the correct sign of the force. As a result, we get the force $F_{ij}$ applied onto point mass $i$ from point mass $j$.

$$F_{ij}=m_{i}a_{i}=G\frac{m_{i}m_{j}\left( r_{j} - r_{i} \right)}{\left( \left| r_{j} - r_{i} \right| \right)^3 + \epsilon^3}$$

How do we start now? We have several options to implement so-called [Explicit and Implicit][6] methods, e.g. Euler, Euler-Cromer, Leapfrog, Verlet, and a few others. We will start with the simplest, but rather inaccurate, explicit Euler-Method.

Let's start with a simple test case called "Explicit Euler algorithm with two point mass" in the solverTest executable.

```cpp
TEST_CASE( "Explicit euler algorithm with two point mass", "[euler]" ) {

    Solver solver;

    ParticleBuilder particleBuilder;

    Particle particleA = particleBuilder
            .position()
            .mass()
            .build();

    Particle particleB = particleBuilder
            .position()
            .mass()
            .build();

    std::vector<Particle> particles = { particleA, particleB };

    const double EPSILON = 1e-3;

    SECTION( "Two still standing point mass are attracting each other" ) {
        std::vector<Particle> result = solver.solve(particles, EPSILON);

        REQUIRE(false);
    }
}
```

This test case describes our, till now, rough design of how we would like to use the solver. We need a Solver to execute the computation, a vector of Particle's, and a `ParticleBuilder` with some sort of [fluent interface][7] to easily generate particles for us. The Euler algorithm does not only need the particles to calculate, but it needs also a time step $t$ as you remember from my first [post][2], we will call it `EPSILON` in the source code (don't get confused with $\epsilon$ we need to resolve the singularity). You can get this start state by downloading [v0.2.0][8].

Our first test will check two particles for the following results in the x-direction test:

$$M_{ij}=\frac{m_{i}m_{j}}{m_{i}}, r=\left( r_{j} - r_{i} \right)$$

$$G=6.67408\cdot10^{-11}\frac{m^3}{kg\cdot s^2}, M_{ij}=10^{10}kg, r_{ij}=-1m, \epsilon=10^{-3}m$$

$$a_{ij}=G \cdot M_{ij}\frac{r}{\left| r \right|^3+\epsilon^3}=-0.6674079993\frac{m}{s^2}$$

$$v_{ij}=v_{0ij}+a_{ij}t, v_{0ij}=0, t=0.1s \rightarrow v_{ij}= -0.06674079993 \frac{m}{s}$$

$$r_{ij}=r_{0ij}+v_{ij}t+a_{ij}t^2, r_{0ij}=0.5m, t=0.1s \rightarrow r_{ij}=0.48665184m$$

All 3 results are valid in case of pure x-direction and y-direction moving point masses. After we implemented the tests and made it compiling the solverTest file looks like this:

```cpp
double Inverse(double value) { return -value; }

Particle GenerateStandardParticle(double xPosition, double yPosition)
{
    ParticleBuilder particleBuilder;

    return particleBuilder
            .position({xPosition, yPosition})
            .mass(1e10)
            .build();
}

TEST_CASE( "Explicit euler algorithm with two point mass", "[euler]" ) 
{
    Solver solver;

    const std::vector<Particle> particlesX = { GenerateStandardParticle(0.5, 0),
                                               GenerateStandardParticle(-0.5, 0)};

    const std::vector<Particle> particlesY = { GenerateStandardParticle(0, 0.5),
                                               GenerateStandardParticle(0, -0.5)};

    const double EPSILON = 1e-3;

    //Solution
    const double acceleration = -0.6674079993; //m/s^2
    const double velocity = -0.06674079993; //m/s
    const double position = 0.48665184; //m

    SECTION( "Two still standing point mass are attracting each other in x-direction" ) {
        std::vector<Particle> result = solver.solve(particlesX, EPSILON);

        Particle &particle = result.front();
        REQUIRE(particle.acceleration.x == Approx(acceleration));
        REQUIRE(particle.velocity.x == Approx(velocity));
        REQUIRE(particle.position.x == Approx(position));

        particle = result.back();
        REQUIRE(particle.acceleration.x == Approx(Inverse(acceleration)));
        REQUIRE(particle.velocity.x == Approx(Inverse(velocity)));
        REQUIRE(particle.position.x == Approx(Inverse(position)));
    }

    SECTION( "Two still standing point mass are attracting each other in y-direction" ) {
        std::vector<Particle> result = solver.solve(particlesY, EPSILON);

        Particle &particle = result.front();
        REQUIRE(particle.acceleration.y == Approx(acceleration));
        REQUIRE(particle.velocity.y == Approx(velocity));
        REQUIRE(particle.position.y == Approx(position));

        particle = result.back();
        REQUIRE(particle.acceleration.y == Approx(Inverse(acceleration)));
        REQUIRE(particle.velocity.y == Approx(Inverse(velocity)));
        REQUIRE(particle.position.y == Approx(Inverse(position)));
    }
}
```

Let's have a look at the `SECTION`'s first. We are executing two tests, one testing acceleration, velocity and position in pure x-direction, and another one in a pure y-direction. Both tests are using the Inverse() function which is inverting the expected results for testing the second particle. The [`Approx`][9] wrapper class of [Catch2][10] is overloading the comparison operators and provides, next to the standard configuration which covers most of the cases, 3 different configuration methods.

1. `epsilon`: Sets an allowed relative difference: `100.5 == Approx(100).epsilon(0.01)`
2. `margin`: Sets an allowed absolute difference: `104 == Approx(100).margin(5)`
3. `scale`: In case the resulting value has a different scale than the expected value, e.g. through different unit systems or unit transformations

To generate our particles we use the ParticleBuilder which itself is providing a fluent interface for easily configuring and generating point masses. For such an easy case, using a builder pattern with a fluent interface won't be necessary but we will need it later on for a more advanced generation.

```cpp
class ParticleBuilder {
public:
    ParticleBuilder() : mMass(0) {}

    ParticleBuilder & position(const Vector2D &position);
    ParticleBuilder & velocity(const Vector2D &velocity);
    ParticleBuilder & acceleration(const Vector2D &acceleration);
    ParticleBuilder & mass(double mass);
    Particle build() const;

private:
    double mMass;
    Vector2D mAcceleration;
    Vector2D mVelocity;
    Vector2D mPosition;
};
```

Vector2D is, till now, a simple data structure which might be extended later with more functionality. Our test results are failing as expected.

```bash
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
solverTest is a Catch v2.6.1 host application.
Run with -? for options

-------------------------------------------------------------------------------
Explicit euler algorithm with two point mass
Two still standing point mass are attracting each other in x-direction
-------------------------------------------------------------------------------
/mnt/c/Develop/gravity/solverTest/src/solverTest.cpp:39
...............................................................................

/mnt/c/Develop/gravity/solverTest/src/solverTest.cpp:43: FAILED:
REQUIRE( particle.acceleration.x == Approx(acceleration) )
with expansion:
0.0 == Approx( -0.6674079993 )

-------------------------------------------------------------------------------
Explicit euler algorithm with two point mass
Two still standing point mass are attracting each other in y-direction
-------------------------------------------------------------------------------
/mnt/c/Develop/gravity/solverTest/src/solverTest.cpp:53
...............................................................................

/mnt/c/Develop/gravity/solverTest/src/solverTest.cpp:57: FAILED:
REQUIRE( particle.acceleration.y == Approx(acceleration) )
with expansion:
0.0 == Approx( -0.6674079993 )

===============================================================================
test cases: 1 | 1 failed
assertions: 2 | 2 failed
```

Next time we will start to implement the rather simple Euler-Method and extend our tests with a couple of more interesting cases, such as singularity and performance tests. As usual, you can get the sources via GitHub [v0.3.0][11].

[1]: https://www.reddit.com/r/cpp/comments/aqi7bk/and_then_there_was_cmake/
[2]: https://thoughts-on-coding.com/2019/02/08/the-n-body-problem-series/
[3]: https://thoughts-on-coding.com/2019/02/14/and-then-there-was-cmake/
[4]: https://en.wikipedia.org/wiki/Test-driven_development
[5]: https://en.m.wikipedia.org/wiki/Stellar_collision
[6]: https://en.m.wikipedia.org/wiki/Explicit_and_implicit_methods
[7]: https://en.wikipedia.org/wiki/Fluent_interface
[8]: https://github.com/Ben1980/gravity/releases/tag/v0.2.0
[9]: https://github.com/catchorg/Catch2/blob/master/docs/assertions.md#floating-point-comparisons
[10]: https://github.com/catchorg/Catch2
[11]: https://github.com/Ben1980/gravity/releases/tag/v0.3.0