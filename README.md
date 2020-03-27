# Houdini-Simulations
https://github.com/stevedatelier/Houdini-Simulations/wiki

Find my Houdini notes and some of the setups I used to create volumes, oceans, fluid and more.

Every effort has been made to credit external sources.

## Vellum grains

#### Vellum Constraints geometry node

After the grain_constraints node you need the vellum_constrains node with target_group_type set to points and constraint_type set to glue.

&nbsp;

#### Glue

Each point will search for a nearby point that is not a member of its own piece. It will construct a distance constraint holding it to that point. This is useful for building systems that automatically glue together by proximity, especially when combined with breaking.

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


`Crest` = Highest point of the wave

`Trough` = Lowest point of the wave

`Wavelength` = Distance from one crest/trough to the next (m)

`Wave Height` = Height from trough to crest (m)

`Wave steepness` = ratio of wave height to wavelength

`Amplitude` = distance from the centre of wave to the bottom of the trough (m)

`Wave Period` = time for one full wavelength to pass a given point (s)

These characteristics are important in determining the size of waves, the speed at which they travel, how they break on shore, and much more. We will refer back to them throughout the following unit.

&nbsp;

#### 5-wave-velocities

The speed at which this one group of waves travels across the water is known as the group velocity. The apparent speed of each individual wave in the group is known as the phase velocity. This phase velocity is the speed at which a single phase of a wave (e.g. a crest) moves across the water and it is typically double the group velocity.

In the simulation below, four groups of waves are shown with the red dot representing the phase velocity and the green dot representing the group velocity.

&nbsp;

#### Wave type / Wind-generated waves


Wind-waves are a result of wind disturbing the ocean surface and displacing water. Natural forces, such as the water’s surface tension (capillarity), or gravity, work to restore the disturbed water to its calm state, flattening the water’s surface. Different types of waves are named for their restoring force. Small winds displace small amounts of water on the surface, creating very short-wavelength capillary waves. Here, capillarity acts as the restoring force. Capillary waves are typically only a few cm in length. Larger winds create gravity waves, for which gravity acts as the restoring force. These waves can be metres to kilometers long.

&nbsp;

#### Wave type / Wind-generated waves Rogue Waves (and Interference)**

These are very large waves that form due to wave interference. What is interference? When two waves run through each other, they can add up or cancel out. This property of waves is known as interference. If the crest of one wave passes through the trough of another, they cancel out, which is called destructive interference. The resulting wave is smaller and carries less energy. Whereas if the crest of one wave passes through with the crest of another wave, they add up, which is called constructive interference. The resulting wave is bigger, carries higher energy, but are temporary (short lived).   Rogue waves are the result of constructive interference that causes the wave height to be unusually higher than the other waves around it, and can catch boaters (and people on the shore) by surprise.  

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

#### Using vellum as skin

Grouped some loops of points around the desired areas of your model (geometry). -Pin the Grouped points to the animation in the vellum configure cloth node. then run and cache a vellum sim.

&nbsp;

## Ocean

&nbsp;

#### ocean_sample VEX function

```
vector  ocean_sample(string geometry, int phase, int frequency, int amplitude, float hscale, float time, int mode, int         downsample, vector pos
    
```
```
@P += ocean_sample("spectrum.bgeo", 0, 1, 2, 0.7, @Time, 0, 0, @P);
    
```
&nbsp;

#### Foam clumping


Variable Stiffness
Note

The solver will interpret constraints to have a variable stiffness if the pbfstiffness point attribute is present, even when Variable Stiffness is disabled.

Inclusion Chance

Note

All eligible whitewater particles are collected into the controldensity_candidate group; particles that are actually used for density control are put into the controldensity group.

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

## Eliminate noise and speed up renders: Proven method for Mantra


&nbsp;

## Tutorial

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

