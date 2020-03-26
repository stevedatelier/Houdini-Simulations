# Houdini-Simulations
https://github.com/stevedatelier/Houdini-Simulations/wiki

Find my Houdini notes and some of the setups I used to create volumes, oceans, fluid and more.

Every effort has been made to credit external sources.

Vellum grains

Vellum Constraints geometry node
After the grain_constraints node you need the vellum_constrains node with target_group_type set to points and constraint_type set to glue.

Glue
Each point will search for a nearby point that is not a member of its own piece. It will construct a distance constraint holding it to that point. This is useful for building systems that automatically glue together by proximity, especially when combined with breaking.

POPGrains/get_neighbours:

    // Do not potentiall collide with explicit constraints
    // to allow us to over-pack particles.
    if (!explicitcollide && find(@ec, ptj) >= 0)
        continue;
    
   append(neighbors, ptj);
    
