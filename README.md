# Houdini Physical Simulations

## Find my notes and some of the code snippets, setups I used to create volumes, oceans, fluid and more inside SideFX Houdini

<p align="right"><small><sup>by Steve d'Atelier</sup></small></p>


### How to use

You can clone, or directly download this repository. It contains examples [...].hip and vex/myLib.h files which are helpful for you to expand on and improve. I recommend you check the README.md file for helpful comments and explanations. Where necessary I include related functions from myLib.h or attach screenshots. I have logged some simulations there as well as important tips on how to significantly reduce noise in your render and greatly improve Mantra rendering speed. Some of the examples listed are influenced by real-world physical observations. 

### Topics

* [Vellum Constraints](#Vellum-Constraints)
* [Vellum grains Glue](#Glue)
* [Wave characteristics](#Wave characteristics)
* [Tank depth and level](#Tank depth and level)
* [5-wave-velocities](#5-wave-velocities)
* [Wave type / Wind-generated waves](#Wave-type-/-Wind-generated-waves)
* [Wave type / Wind-generated waves Rogue Waves (and Interference)](#Wave type / Wind-generated waves Rogue Waves (and Interference))
* [Wind force](#Wind-force)
* [POP Wind](#POP-wind)
* [ocean_sample VEX function](#ocean_sample VEX function)
* [Foam clumping](#Foam clumping)
* [Whitewater / Bind Arrays](#Whitewater / Bind Arrays)
* [Using vellum as skin](#Using vellum as skin)
* [Enforce Prototypes](#Enforce Prototypes)* 
* [Get transform of objects with optransform](#Get transform of objects with optransform)
* [optransform to do a motion control style camera](#optransform to do a motion control style camera)
* [Control Pyro based on vex rules](#Control Pyro based on vex rules)
* [Dynamic clouds and volumes](#Dynamic clouds and volumes)
* [Mantra limit parameters](#Mantra limit parameters)
* [Mantra / Understanding the difference between direct and indirect lighting.](#nderstanding the difference between direct and indirect lighting)
* [Mantra / Eliminate firefly noise: Avoid using sunlights](#Eliminate firefly noise: Avoid using sunlights)
* [Mantra / Reduce noise significantly](#Reduce noise significantly)
* [Mantra / Results](#Results)
* [Python Scripting hou ](#VPython Scripting hou )
* [Speed-up Render time / Opt for Physically Based Rendering](#Opt for Physically Based Rendering)
* [Speed-up Render time / Dicing:shading quality](#Dicing:shading quality)
* [Speed-up Render time / Pixel samples](#Pixel samples)
* [Speed-up Render time / Max Ray samples](#Max Ray samples)
* [Speed-up Render time / Noise level](#Noise level)
* [Speed-up Render time / Light quality. All= 1](#Light quality. All= 1)
* [Speed-up Render time / Scholastic samples](#Scholastic samples)
* [Speed-up Render time / Refract limit](#Refract limit)
* [Speed-up Render time / PBR shading](#PBR shading)
* [Speed-up Render time / Shading quality multiplier](#Shading quality multiplier)
* [Reading and modifying the voxel value](#Reading and modifying the voxel value)
* [Packed Primitives](#Packed Primitives)
* [Blurring attributes with vex and point clouds](#Blurring attributes with vex and point clouds



## Vellum grains

#### Vellum Constraints

After the grain_constraints node you need the vellum_constrains node with target_group_type set to points and constraint_type set to glue.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/ezgif.com-optimize_Muddy.gif" width="60%">.


&nbsp;

#### Glue

Each point will search for a nearby point that is not a member of its own piece. It will construct a distance constraint holding it to that point. This is useful for building systems that automatically glue together by proximity, especially when combined with breaking.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/ezgif.com-optimize-2.gif" width="60%">.

POPGrains/get_neighbours:
```
    // Do not potentiall collide with explicit constraints
    // to allow us to over-pack particles.
    if (!explicitcollide && find(@ec, ptj) >= 0)
        continue;
    
    append(neighbors, ptj);
    
 ```


 &nbsp;
    
## Waves

#### Wave characteristics

&nbsp;

#### Tank depth and level

Look for Particle Fluid Tank: Increasing the Water level to 3 or 4 will create interesting waves. 

Resize the tank
In the network editor, go up to the Scene level, then double-click the fliptank_initial object to go inside.
Click the wavetank node.
Use the handles in the viewport or the parameters in the parameter editor to change the size of the box.

Change the fluid level
In the network editor, go up to the Scene level, then double-click the fliptank_initial object to go inside.
Modify the Water Level parameter to adjust the fluid level in the tank.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/8-wave-characteristics.gif" width="60%">.

###### Source: [Unversity of Brtish Columbia](https://www.eoas.ubc.ca/courses/atsc113/sailing/met_concepts/08-met-waves/8b-wave-characteristics/index.html)

`Crest` = Highest point of the wave

`Trough` = Lowest point of the wave

`Wavelength` = Distance from one crest/trough to the next (m)

`Wave Height` = Height from trough to crest (m)

`Wave steepness` = ratio of wave height to wavelength

`Amplitude` = distance from the centre of wave to the bottom of the trough (m)

`Wave Period` = time for one full wavelength to pass a given point (s)

These characteristics are important in determining the size of waves, the speed at which they travel, how they break on shore, and much more. We will refer back to them throughout the following unit.


##### Before
<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/ezgif.com-optimize%20(8).gif" width="60%">.



##### After
<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/water_lvl.jpg" width="60%">.

*These numeric values are **"just for the purpose of this demonstration."** Your measurements will certainly need to be different. Without tests, I can't predict what your simulation will do.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/ezgif.com-optimize%20(2).gif" width="60%">.


&nbsp;

#### 5-wave-velocities

The speed at which this one group of waves travels across the water is known as the group velocity. The apparent speed of each individual wave in the group is known as the phase velocity. This phase velocity is the speed at which a single phase of a wave (e.g. a crest) moves across the water and it is typically double the group velocity.

In the simulation below, four groups of waves are shown with the red dot representing the phase velocity and the green dot representing the group velocity.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/5-wave-velocities.gif" width="60%">.

###### Source: Kraaiennest - Own work, GFDL, https://commons.wikimedia.org/w/index.php?curid=3651297

&nbsp;

#### Wave type / Wind-generated waves


Wind-waves are a result of wind disturbing the ocean surface and displacing water. Natural forces, such as the water’s surface tension (capillarity), or gravity, work to restore the disturbed water to its calm state, flattening the water’s surface. Different types of waves are named for their restoring force. Small winds displace small amounts of water on the surface, creating very short-wavelength capillary waves. Here, capillarity acts as the restoring force. Capillary waves are typically only a few cm in length. Larger winds create gravity waves, for which gravity acts as the restoring force. These waves can be metres to kilometers long.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/8-wind-wave-types.jpg" width="60%">.


&nbsp;

#### Wave type / Wind-generated waves Rogue Waves (and Interference)**

These are very large waves that form due to wave interference. What is interference? When two waves run through each other, they can add up or cancel out. This property of waves is known as interference. If the crest of one wave passes through the trough of another, they cancel out, which is called destructive interference. The resulting wave is smaller and carries less energy. Whereas if the crest of one wave passes through with the crest of another wave, they add up, which is called constructive interference. The resulting wave is bigger, carries higher energy, but are temporary (short lived).   Rogue waves are the result of constructive interference that causes the wave height to be unusually higher than the other waves around it, and can catch boaters (and people on the shore) by surprise.  

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/8-rogue.jpg" width="40%">.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/ezgif.com-optimize.gif" width="60%">.


&nbsp;

## Vellum and Forces

&nbsp;

#### Wind force

Wind is an velocity sensitive force. If an object is already traveling at the wind’s velocity, then it will not receive any extra force from it. The Wind Force DOP simulates a pushing force like wind, that can increase the velocity of objects but not past its own speed.

Note: If the wind does not apply velocity, increase the Scale Force value on the Wind tab of the Wind Force node in the parameter editor.

&nbsp;

#### POP Wind

The POP Wind node applies wind to particles by updating their targetv and airresist attributes.

Unlike POP Force, which continuously accelerates particles, POP Wind instead acts as a drag pulling particles to the ambient wind speed.

This operator modifies the targetv, and airresist attributes.

Using Wind on Particles

Click the Wind on Particles button on the Particles shelf tab.


&nbsp;


## Ocean

&nbsp;

#### ocean_sample VEX function

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/ezgif.com-optimize%20(3).gif" width="60%">.

```
vector  ocean_sample(string geometry, int phase, int frequency, int amplitude, float hscale, float time, int mode, int         downsample, vector pos
    
```
```
@P += ocean_sample("spectrum.bgeo", 0, 1, 2, 0.7, @Time, 0, 0, @P);
    
```
&nbsp;

#### Foam clumping


&nbsp;

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/ezgif.com-optimize%20(7).gif" width="60%">.


**Variable Stiffness

The solver will interpret constraints to have a variable stiffness if the `pbfstiffness` point attribute is present, even when `Variable Stiffness` is disabled.

&nbsp;

**Inclusion Chance

All eligible `whitewater` particles are collected into the `controldensity_candidate` group; particles that are actually used for density control are put into the `controldensity` group.

&nbsp;

## Whitewater / Bind Arrays

You can bind arrays by appending `[]`, as in

```i[]@connected_pts = neighbours(0, @ptnum);```
For example, the following code loads the `foo` attribute as a vector and copies it to the `P` (position) attribute. You don’t need to specify the type of the `P` attribute because it’s one of the known attributes Houdini casts automatically.

```
@P = v@foo;

```
The following code sets the `x` component of the `Cd` attribute to the value of the whitewater attribute. You don’t need to specify the type of the `Cd` attribute because it’s one of the known attributes. You don’t need to specify the type of the `whitewater` attribute because it’s a float and unknown attributes are cast as float automatically.

```
@Cd.x = @whitewater;

```

&nbsp;

#### Using vellum as skin

Grouped some loops of points around the desired areas of your model (geometry). -Pin the Grouped points to the animation in the vellum configure cloth node. then run and cache a vellum sim.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/ezgif.com-optimize%20(6).gif" width="60%">.


&nbsp;


#### Enforce Prototypes	

```
vector temp = @P;
temp += {0.1, 0.2, 0.3};
@density = volumesample(@OpInput1, 0, temp);

```

&nbsp;

#### Get transform of objects with optransform

```
matrix m = optransform('/obj/cam1');
@P *=m;

```

&nbsp;

#### optransform to do a motion control style camera

```
matrix m = optransform('/obj/moving_cube');
@P *= invert(m);

```

But what about rotation? Looking at the rivet parameters, it supports rotation, but only via `@N` and `@up`. At this point I realised I knew how to go from `@N` and `@up` to a matrix or quaternion, but not the other way. After sleeping on it, realised its easier than I expected, and added this to my wrangle:

```
matrix m = optransform('/obj/moving_cube');
@P *= invert(m);

```
```
matrix3 m_rot = matrix3(m);
@N = {0,0,1}*invert(m_rot);
@up = {1,0,0}* invert(m_rot);

```

&nbsp;

## Pyro

&nbsp;

#### Control Pyro based on vex rules

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/ezgif.com-optimize%20(4).gif" width="60%">.


you can use the fit function for that:


```
float drag = fit(@P.y,0,10,0,0.1); // zero drag at y=0, drag or 0.1 at y=10
v@vel *= 1-drag;

```
if you want to clamp the velocity it's a bit different:

```
float maxspeed = fit(@P.y,0,10,100,1); 
v@vel = normalize(v@vel)* min(length(v@vel),maxspeed);

```


&nbsp;

## Dynamic clouds and volumes

Using the Noise Field dynamics node can yield some petty realistic results when working with volumes.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/ezgif.com-optimize%20(1).gif" width="60%">.


LOCALS

channelname
This DOP node defines a local variable for each channel and parameter on the Data Options page, with the same name as the channel. So for example, the node may have channels for Position (positionx, positiony, positionz) and a parameter for an object name (objectname).

This previous value is always stored as part of the data attached to the object being processed. This is essentially a shortcut for a dopfield expression like:

```
dopfield($DOPNET, $OBJID, dataName, "Options", 0, channelname)

```

Turbulent Noise VOP node
Can compute three types of 1D and 3D noise with the ability to compute turbulence with roughness and attenuation.

This operator can compute three types of 1D and 3D noise with the ability to compute turbulence with roughness and attenuation:

```
Perlin noise              (string value "pnoise")
Original Perlin noise     (string value "onoise")
Sparse Convolution noise  (string value "snoise")
Alligator noise           (string value "anoise")
Simplex noise             (string value "xnoise")
Zero Centered Perlin      (string value "correctnoise")

```

&nbsp;

# Noise and render speed: Mantra

Reduce noise and speed up your renders: A proven method for Mantra

&nbsp;

## Reduce and eliminate noise

&nbsp;

#### Mantra limit parameters

These controls are on the Limits tab of the Mantra render node. I would leave these parameters to there default values except for diffuse limit and volume limit (if there actually volumes are in your scene). Raise diffuse limit to 4. You can also rais color limit to a number like 10 or more, but keep in mind that could increase render time.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/BasicRayTracing_1.png" width="60%">.

&nbsp;

#### Understanding the difference between direct and indirect lighting. 

Indirect Rays can be described as rays which deal with objects and their surface properties. This generally means that rays travel from some position in the scene in directions determined by the shader attached to the object. Refraction rays will travel “through” objects, Reflection Rays will bounce, and Diffuse Rays will scatter in a random direction within a hemispherical distribution. 

Direct lighting gives you a darker image with more pronounced shadows.


&nbsp;

#### Eliminate firefly noise: Avoid using sunlights

If you have to use a sun-like source, I suggest faking it with an area lights. The Houdini sunlight creates big white dots on your render which I refer to as fireflies. 

The Fix: The best way to remove those white dots is to avoid using sun lights altogether.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/Screenshot%20(3103).jpg" width="100%">.
<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/Screenshot%20(3101).jpg" width="100%">.

&nbsp;

#### Reduce noise significantly

When using HDR/Envlight, you surround the entirety of your scene with a light that has a very limited sample count and unfortunately some brights spots which Mantra can't always calculate. 

The Fix: Switch the "rendering node" dropdown option to "raytracing" or "direct lighting" in your envlight node. Note: Only one of them will have the desired results, so make two render tests.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/Screenshot%20(3106).jpg" width="60%">. 
<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/Screenshot%20(3108).jpg" width="60%">.

&nbsp;

#### Results

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/Screenshot%20(3102).jpg" width="60%">.


&nbsp;

#### hou.RopNode class

Python Scripting  hou 

```
render(frame_range=(), res=(), output_file=None, output_format=None, 
to_flipbook=False, quality=2, ignore_inputs=False, method=RopByRop, ignore_bypass_flags=False, ignore_lock_flags=False, verbose=False, output_progress=False)

```
 
&nbsp;


## Speed-up Render time



#### Opt for Physically Based Rendering

Physically based rendering/raytracing (PBR) 
should be your default choice for almost any rendering. PBR simulates real-world light, making it easier to understand. With PBR you get shadows, reflections, secondary bounces, and so on "for free" instead of having to use workarounds or write complex shaders. You can also repurpose traditional methods to achieve lighting effects, for example putting cucoloris or gobo geometry in front of a light, or using a very powerful light to "blow out" part of the scene

The raytracing engine will see the shader as:

```
surface
clay(vector diff=1; float rough=.1; vector Cd=1)
{
    vector        nml;

    nml = frontface(normalize(N), I);
    Cf  = diffuse(nml, -normalize(I), rough);
    Cf *= Cd * diff;
}
The PBR engine will see the shader as:

surface
clay(vector diff=1; float rough=.1; vector Cd=1)
{
    F = (Cd * diff) * diffuse();
}

```

The two rendering engines will often render nearly identical images since both engines often make use of the `pbrlighting()` shader. This shader takes a BSDF input and computes the resulting color.

This can be seen in the `plastic.vfl` shader that’s shipped with Houdini. The shader looks something like:

```
import pbrlighting;
surface
plastic(
    vector diff=0.8;
    vector spec=0.2;
    float rough=0.2;
    vector Cd=1;
    ...
    export vector direct=0;
    export vector indirect=0;
    export vector indirect_emission=0;
    ...
)
{
    float        exponent = 1.0 / rough;

    F = (Cd * diff) * diffuse() + spec * phong(exponent);

    pbrlighting(
            "direct", direct,
            "indirect", indirect,
            "indirect_emission", indirect_emission,
            ...
            "inF", F);

    Cf = direct + indirect + indirect_emission;
}

```

&nbsp;

#### Dicing:shading quality

This is the most important part, the part you need to remember. The absolute best way to significantly reduce render times is by adjusting shading quality. The lower the number, the faster your render. There were no noticeable differences in my 2K rendrers after I lowered shading quality. 

Dicing > Shading Quality: Dial your number to as low as 0.25

&nbsp;

#### Pixel samples

Keep in mind the higher these two numbers, the slower your render time will be. Pixel samples serve as a global multiplier for the adjustments you make below. It isn't necessary to play with these since we are already working with the controls below.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/Screenshot%20(3101)_render-setup.jpg" width="50%">.

&nbsp;

#### Max Ray samples

Try 10, 100, or even 300. Make several renders tests. One of them could cost you a few minutes. I've had excellent results with 100.

&nbsp;

#### Noise level

In the physical world. The noise level of real film grains is 0.05 or 0.08. Noise will affect both the quality and speed of your renders. 

I always select 0.07-0.05 for volumes, 0.05 for heavy scenes of all kind, and 0.03-0.01 when I can spare the time. With render images of 4k or 5k for resolution, going below 0.01 is silly. 

&nbsp;

#### Light quality. All= 1

1 is good enough.

&nbsp;

#### Scholastic samples

Turn it on.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/VolumeSamplingStochastic.jpg" width="50%">.

&nbsp;

#### Refract limit

2 or 4 will work.

<img src="https://github.com/stevedatelier/Houdini-Simulations/blob/master/img/RefractLimit.jpg" width="50%">.

&nbsp;

#### PBR shading

You can add bsdf values together and scale them by vectors or floats. Multiplying a BSDF by a color vector makes the surface reflect that color.

```
    // Multiply the diffuse (hemispherical) BSDF by the texture color,
    // multiply a phong BSDF by 0.5, and then add the two BSDFs together
    F = texture(map) * diffuse() + 0.5 * phong(20);
    // Red diffuse surface
    F = diffuse() * {1.0, 0, 0}

```

&nbsp;

#### Shading quality multiplier

Dicing

Houdini render property `vm_shadingfactor`
`IFD` property `renderer:shadingfactor`
A global multiplier on all per-object shading quality (vm_shadingquality) parameters in the scene. This parameter can be used to globally increase or decrease shading quality. The shading quality used for an object is determined by…

```
shadingquality = object:shadingquality * renderer:shadingfactor

```

&nbsp;

# Tutorial

&nbsp;


#### Reading and modifying the voxel value

    // Modify @foo, @bar, and @baz in the same way
    // (when Bind each volume to density is on)
    @density += sin(@P.x)
    
    
&nbsp;

#### Packed Primitives

```
surface showversion() 
{
    string    rname, rversion;
    if (!renderstate("renderer:name", rname))
        rname = "Unknown renderer";
    if (!renderstate("renderer:version", rversion))
        rversion = "Unknown version";
    printf("Image rendered by %s (%s)\n", rname, rversion);
}

vector mapToScreen(vector NDC_P)
{
    // Given a point in NDC space, find out which pixel it
    // maps to.
    vector    result;
    if (!renderstate("image:resolution", result))
        result = {640, 486, 0};
    return result * NDC_P;
}

```

&nbsp;

#### Blurring attributes with vex and point clouds

You have `Cd` on a grid, you want to blur it. One way is to iterate through each point, look at its neighbours, add up the result, and divide by the number of neighbours. Because it's a grid, and the point numbering is consistent, you could do something like

```
int width = 50; // say the grid is 50 points wide
vector left = point(0,'Cd', @ptnum-1);
vector right = point(0,'Cd', @ptnum+1);
vector top = point(0,'Cd', @ptnum+50);
vector bottom = point(0,'Cd', @ptnum-50);
@Cd = left + right + top + bottom;
@Cd /= 4.0;

```

Works, but clunky, and it only works with a grid. Using the `neighbour()` and `neighbourcount()` functions is more generic (borrowed from odforce post):

```
int neighbours = neighbourcount(0, @ptnum);
vector totalCd = 0;
for (int i = 0; i < neighbours; i++)
{
    int neighPtnum = neighbour(0, @ptnum, i);    
    totalCd += point(0, "Cd", neighPtnum);
}
@Cd = totalCd/neighbours;

```

Groovy, nice and generic now.

However, there's an even shorter cleaner way, using point clouds:

```

int mypc = pcopen(0, 'P', @P, 1, 8);
@Cd = pcfilter(mypc, 'Cd');

```

### To do
Any suggestions? questions? Please ask.

&nbsp;

### Resources

In this tutorial I am focusing on fluid & pyro simulations using the physcal poperties in Houdini.
For more on Vex syntax check SideFX wiki page http://www.tokeru.com/cgwiki/?title=HoudiniVex
You'll find additonal examples on Odforce.net.

Remember that the more precise you are, the more beautiful your simulations will be. The possibilities are limitless. To be successful, focus on reduction. This means that you want to find solutions that will save you time. Let yourself be inspired by the wonderful world of physics. There are several courses and readings on the Internet to help you understand the importance of the laws of nature and the laws of gravity - I know what you think *. "Physicists have discovered that gravity is not a force, so what should I worry about it?"* Well to that I answer, until someone solves the problem of quantum gravity, you're stuck with Einstein's general relativity, so *just work with it!



