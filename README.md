# STFGEN.py
STFGEN (Single-Type Fiber GENerator) is a hardcoded Python script for the generation of random fiber network according to input parameters. Current script allow to adjust input data on fiber structural (number of fibers, its length range, diameter, angle displacement, etc.) and spatial (volume occupation) properties. and provide output data of 3D bead coordinates in .xyz file format, as well as output information on generator run in .txt file format. 
Current repositorium consists of two Python scripts - STFGEN_O.py, which allows to generate fiber network with no bead overlays, and STFGEN_N.py, which do the same but do not handle bead overlay issues. Both codes are pretty much comparable, so that the description below is given for the STFGEN_O.py script, being also sencefull for the STFGEN_N.py one.

- Visual Studio Code version: 1.85.1
- Python version: 3.7.7.

> Warning! There are no guaranties that this software will run on your machine.

Following script consist of several principle blocks of the code, which are explained below.
### Importing necessary libraries
 ```
# Import libraries
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
import random
import os
from datetime import datetime
 ```
### Defining date and time 
> Here, it is done for the purpose of proper naming convention of output files.
```
# Get the current date and time
current_date = datetime.now().strftime('%H.%M.%S_%d.%m.%Y')
```
### Defining folders for output files
> Make sure to modify this segment prior to the code use
```
# Define output folders
output_xyz_folder = '____'
output_stats_folder = '____'
output_length_folder = '____'
```
# Defining input variables
> Make sure to modify this segment prior to the code use.
```
# Define variables
box_length = ___ # Length of simulation box
box_width = ___ # Width of simulation box
box_thickness = ___ # Thickness of simulation box
sphere_radius = ___ # Radius of the single bead
fiber_length_mean = ___ # Mean length of modeled fibers. Experimental value
fiber_length_sd = ___ # Standard deviation of modeled fibers. Experimental value
volume_occupied = 0 # Minimum (starting) volume of simulation box occupied (a.u.)
max_volume_occupied = ___ # Maximum volume of simulation box occupied (a.u.)
cutoff_distance = ___ # Minimum distance between the beads of two different fibers
sphere_overlay = sphere_radius # Distance, on which beads of the same fiber are allowed to overlay, mimicking bead-bead interaction
bead_bead_angle_mean = ___ # Mean of an angle between consequtive beads of fiber (in deg.)
bead_bead_angle_sd = ___ # Standard deviation of an angle between consequtive beads of fiber (in deg.)
fiber_index = 0 # Starting fiber number in box
bead_volume = (4/3) * np.pi * sphere_radius**3 # Volume of single bead
overlay_volume = ((np.pi / 12) * (4 * sphere_radius + sphere_radius) * (2 * sphere_radius - sphere_radius)**2) # Area of bead-bead overlay
bead_bead_overlay_ratio = overlay_volume / (bead_volume) # Area of bead-bead overlay as a ratio to bead area
total_beads = int((box_length * box_width * box_thickness * max_volume_occupied) / (bead_volume - bead_volume * bead_bead_overlay_ratio)) # Calculate total number of beads in box
total_fibers = int(total_beads / (fiber_length_mean / sphere_radius)) # Calculate total number of fibers in box
```
### Generate normal distribution of fiber length
> This block of code generates normal distribution of fiber length according to the input values of its mean and standard deviation. Additionally, code ensures that the minimum fiber length is the diameter of one bead. At the very end of the block, array of generated values is sorted from highest to lowest. It allows to efficiently generate longer fibers within the network, allowing to overcome sterical hindrances, which may occur while shorter fibers are already in simulation box.
```
# Generate random values of fiber length according to input means and standard deviation
random_fiber_length = np.random.normal(0, 1, total_fibers)
random_fiber_length = random_fiber_length * fiber_length_sd + fiber_length_mean
random_fiber_length = np.clip(random_fiber_length, 0, None)
random_fiber_length = np.maximum(random_fiber_length, sphere_radius)
random_fiber_length = np.round(random_fiber_length / sphere_radius) * sphere_radius
random_fiber_length[random_fiber_length < sphere_radius] = sphere_radius
random_fiber_length = np.sort(random_fiber_length)[::-1]
```
### Initializing arrays to store bead positions and fiber indices
```
# Initialize arrays to store bead positions and fiber indices
x_positions = []
y_positions = []
z_positions = []
fiber_indices = []
```
### Defining function to check overlay of the beads of different values
> Current script allows overlay of the beads of the same fiber, assuming it as an interbead junction. However, bead overlay of the different fibers if restricted, so that the minimum distance between the beads of different fibers equals to two radii and some cutoff distance. Cutoff distance defines closes distance between the bead edges.
```
# Function to check beads of different fibers to overlap
def check_overlap(pos1, pos2, radius, same_fiber):
    if same_fiber:
        return False # Allow overlap for the same fiber
    else:
        distance = np.linalg.norm(pos1 - pos2)
        return distance < radius * 2 + cutoff_distance # Interfiber bead overlay restricted
```
### Starting fiber generation
> Current block takes the fiber length from a previously generated array, calculates beads number within a fiber, and defines some random position in 3D for the first bead.
```
# Starting fiber generation
print(f'Starting generation for volume occupied: {volume_occupied}')

# Start generate beads of each fiber within array of fiber length
for fiber_length in random_fiber_length:

    # Define number of beads in each fiber
    n_beads = int(fiber_length / sphere_radius) 

    # Genearting the first bead of the fiber
    for bead_number in range(1):
        
        # Generate random position for the first bead in the fiber
        fiber_positions = np.array([[random.randint(sphere_radius, box_length - sphere_radius),
                                    random.randint(sphere_radius, box_width - sphere_radius),
                                    random.randint(sphere_radius, box_thickness - sphere_radius)]])
        
        # Generate first bead within the simulation box
        if (fiber_positions < sphere_radius).any() or (fiber_positions > box_length - sphere_radius).any():

            # Generate random position for the first bead in the fiber
            fiber_positions = np.array([[random.randint(sphere_radius, box_length - sphere_radius),
                                        random.randint(sphere_radius, box_width - sphere_radius),
                                        random.randint(sphere_radius, box_thickness - sphere_radius)]])
```
### Handling bead overlay
> While generated bead overlay existing one, some new random position is generated for that bead.
```
        # Collision detection for the first bead of each fiber
        while any(check_overlap(fiber_positions[0], np.array([x_positions[i], y_positions[i], z_positions[i]]), sphere_radius, fiber_indices[i] == fiber_index) for i in range(len(x_positions))):
            
            # Regenerate random position for the first bead in the fiber
            fiber_positions = np.array([[random.randint(sphere_radius, box_length - sphere_radius),
                                        random.randint(sphere_radius, box_width - sphere_radius),
                                        random.randint(sphere_radius, box_thickness - sphere_radius)]])
```
### Technical block - adding atomic symbols and fiber index to generated bead
```
        # Append atomic symbol "C" to the global atomic symbols array
        atomic_symbols = ["C"]

        # Add fiber number
        fiber_index += 1
```
### Generating second bead of the fiber
> In current script, second bead defines directionality, in which fiber will be continuing to generate. Thus, it is randomly done by defining one of 3! possible directions in 3D (east, west, north, south, top, bottom) by calling direction_x, direction_y and direction_z lines. Further on, random_angle_theta and random_angle_phi defines some random bead displacement in spherical coordinates (Figure 1) for chosen direction. Defined theta and phi angles allow to calculate bead offset, which further allows to define bead position.
```
    # Generate the second bead of each fiber
    for bead_number in range(2):

        # While the second bead does not overlay the already generated beads
        while True:

            # For the second bead, generate random directionality is spherical coordinates
            random_angle_theta = np.deg2rad(np.random.randint(0, 180))
            random_angle_phi = np.deg2rad(np.random.randint(0, 360))

            # Generate fiber directionality in x, y, and z coordinates
            direction_x = 2 * np.random.randint(2) - 1 # Randomly select -1 or 1
            direction_y = 2 * np.random.randint(2) - 1 # Randomly select -1 or 1
            direction_z = 2 * np.random.randint(2) - 1 # Randomly select -1 or 1

            # Calculate bead displacement in spherical coordinates
            x_offset = sphere_radius * np.sin(random_angle_theta) * np.cos(random_angle_phi)
            y_offset = sphere_radius * np.sin(random_angle_theta) * np.sin(random_angle_phi)
            z_offset = sphere_radius * np.cos(random_angle_theta)

            # Calculate bead coordinates
            bead_position = np.array([fiber_positions[-1, 0] + x_offset * direction_x,
                                        fiber_positions[-1, 1] + y_offset * direction_y,
                                        fiber_positions[-1, 2] + z_offset * direction_z])
           # Generate second bead within the simulation box
            if (bead_position[0] < sphere_radius).any() or (bead_position[0] > box_length - sphere_radius).any() or (bead_position[1] < sphere_radius).any() or (bead_position[1] > box_width - sphere_radius).any() or (bead_position[2] < sphere_radius).any() or (bead_position[2] > box_thickness - sphere_radius).any():

                # For the second bead, generate random directionality is spheric coordinates
                random_angle_theta = np.deg2rad(np.random.randint(0, 180))
                random_angle_phi = np.deg2rad(np.random.randint(0, 360))

                # Generate fiber directionality in x, y, and z coordinates
                direction_x = 2 * np.random.randint(2) - 1 # Randomly select -1 or 1
                direction_y = 2 * np.random.randint(2) - 1 # Randomly select -1 or 1
                direction_z = 2 * np.random.randint(2) - 1 # Randomly select -1 or 1

                # Calculate bead displacement in spherical coordinates
                x_offset = sphere_radius * np.sin(random_angle_theta) * np.cos(random_angle_phi)
                y_offset = sphere_radius * np.sin(random_angle_theta) * np.sin(random_angle_phi)
                z_offset = sphere_radius * np.cos(random_angle_theta)

                # Calculate bead coordinates
                bead_position = np.array([fiber_positions[-1, 0] + x_offset * direction_x,
                                            fiber_positions[-1, 1] + y_offset * direction_y,
                                            fiber_positions[-1, 2] + z_offset * direction_z])
```
Figure 1. Bead generation in spherical coordinate system.
![Figure 8](https://github.com/vchibrikov/STFGEN/assets/98614057/f0aae5da-48e9-4d87-8651-e2b786281d79)
### Handling bead overlay
> Similar to the first bead, while overlay is detected, new position of the second bead is generated by defining some other random directionality, displacement angles, offsets and coordinates.
```
                       # Check if the regenerated bead overlaps with any previously generated beads
            if not any(check_overlap(bead_position, np.array([x_positions[i], y_positions[i], z_positions[i]]), sphere_radius, fiber_indices[i] == fiber_index) for i in range(len(x_positions))):
                break  # Exit the loop if no overlap
```
Example of the set of randolmy generated set of first and second beads is provided on Figure 2.
Fig. 2 Randomly generated first and second beads of each fiber at 5% volume occupation of fibers, with beads of the radius of 25 units. X-Y axis view.
![Figure 7](https://github.com/vchibrikov/STFGEN/assets/98614057/0086d7cb-f772-470b-b687-f02e4423e7dd)
### Technical block - defining atomic symbol, adding position of the second bead to the general list, fiber index, calculating occupied volume and printind respected message
```
	# Append atomic symbol, index and bead positions to the list
        atomic_symbols.append("C")
        x_positions.append(bead_position[0])
        y_positions.append(bead_position[1])
        z_positions.append(bead_position[2])
        fiber_indices.append(fiber_index)
        fiber_positions = np.vstack([fiber_positions, bead_position])

        # Calculate the volume occupied by the generated beads
        volume_occupied = (bead_volume * len(x_positions)) - ((bead_volume * bead_bead_overlay_ratio) * (len(x_positions) - fiber_index))

	# Print terminal statement on code processing
        print(f'Added bead {str(bead_number + 1).zfill(5)} of {n_beads} for fiber {fiber_index} of {len(random_fiber_length)}. Volume occupied: {round(volume_occupied * 100 / (box_length * box_thickness * box_width), 5)}%')
```
### Generating beads from the third till fiber end
> Current block of code allows for further bead generation in predefined direction. For each bead, displacement is defined by some random value of angle displacements random_angle_theta and random_angle_phi, which correspond to the one, defined earlier, yet modified by some random value of angle displacements random_angle_theta_modifier and random_angle_phi_modifier.
```
   # For each fiber, generate beads from the third to the end of the fiber length
    for bead_number in range(3, round(n_beads + 1), 1):

	# If there is no bead overlay
        overlay_detected = False
        while True:

	    # Define directionality modifier in spherical coordinates
            random_angle_theta_modifier = np.deg2rad(np.random.uniform(-bead_bead_angle_sd, (bead_bead_angle_sd + 1)))
            random_angle_phi_modifier = np.deg2rad(np.random.uniform(-bead_bead_angle_sd, (bead_bead_angle_sd + 1)))
            random_angle_theta += random_angle_theta_modifier
            random_angle_phi += random_angle_phi_modifier

            # Calculate bead displacement in spherical coordinates
            x_offset = sphere_radius * np.sin(random_angle_theta) * np.cos(random_angle_phi)
            y_offset = sphere_radius * np.sin(random_angle_theta) * np.sin(random_angle_phi)
            z_offset = sphere_radius * np.cos(random_angle_theta)

            # Calculate bead coordinates
            bead_position = np.array([fiber_positions[-1, 0] + x_offset * direction_x,
                                        fiber_positions[-1, 1] + y_offset * direction_y,
                                        fiber_positions[-1, 2] + z_offset * direction_z])
            
            # Generate beads within the simulation box
            if (bead_position[0] < sphere_radius).any() or (bead_position[0] > box_length - sphere_radius).any() or (bead_position[1] < sphere_radius).any() or (bead_position[1] > box_width - sphere_radius).any() or (bead_position[2] < sphere_radius).any() or (bead_position[2] > box_thickness - sphere_radius).any():

	        # Define directionality modifier in spherical coordinates
                random_angle_theta_modifier = np.deg2rad(np.random.uniform(-bead_bead_angle_sd, (bead_bead_angle_sd + 1)))
                random_angle_phi_modifier = np.deg2rad(np.random.uniform(-bead_bead_angle_sd, (bead_bead_angle_sd + 1)))
                random_angle_theta += random_angle_theta_modifier
                random_angle_phi += random_angle_phi_modifier

                # Calculate bead displacement in spherical coordinates
                x_offset = sphere_radius * np.sin(random_angle_theta) * np.cos(random_angle_phi)
                y_offset = sphere_radius * np.sin(random_angle_theta) * np.sin(random_angle_phi)
                z_offset = sphere_radius * np.cos(random_angle_theta)

                # Calculate bead coordinates
                bead_position = np.array([fiber_positions[-1, 0] + x_offset * direction_x,
                                            fiber_positions[-1, 1] + y_offset * direction_y,
                                            fiber_positions[-1, 2] + z_offset * direction_z])
```
### Handling bead overlay
> While generated bead overlay existing one, some new positions are generated for the beads from the third till the last one
```
            # Check if the regenerated bead overlaps with any previously generated beads
            if not any(check_overlap(bead_position, np.array([x_positions[i], y_positions[i], z_positions[i]]), sphere_radius, fiber_indices[i] == fiber_index) for i in range(len(x_positions))):
                break  # Exit the loop if no overlap

            # While overlay is detected
            overlay_detected = True

        if overlay_detected:

            # Identify indices of beads in the current fiber
            fiber_indices_to_delete = [i for i, idx in enumerate(fiber_indices) if idx == fiber_index]

            # Ensure there are more than two beads in the fiber before attempting to delete
            if len(fiber_indices_to_delete) > 2:

                # Delete beads with the same fiber index, starting from the third bead
                del_indices = fiber_indices_to_delete[2:]

                # Check if the array size is greater than the index to delete
                if len(fiber_positions) > max(del_indices):

                    # Delete corresponding elements using NumPy
                    x_positions = np.delete(x_positions, del_indices)
                    y_positions = np.delete(y_positions, del_indices)
                    z_positions = np.delete(z_positions, del_indices)
                    fiber_indices = np.delete(fiber_indices, del_indices)
                    fiber_positions = np.delete(fiber_positions, del_indices, axis=0)
```
### Technical block - defining atomic symbol, adding position of the second bead to the general list, fiber index, calculating occupied volume and printind respected message
```
        # Append the bead position to the global position arrays
        x_positions.append(bead_position[0])
        y_positions.append(bead_position[1])
        z_positions.append(bead_position[2])

        # Append the fiber index to the fiber_indices array
        fiber_indices.append(fiber_index)

        # Append the bead position to the fiber_positions array
        fiber_positions = np.vstack([fiber_positions, bead_position])

        # Calculate the volume occupied by the generated beads
        volume_occupied = (bead_volume * len(x_positions)) - ((bead_volume * bead_bead_overlay_ratio) * (len(x_positions) - fiber_index))

	# Print terminal statement on code processing
        print(f'Added bead {str(bead_number).zfill(5)} of {n_beads} for fiber {fiber_index} of {len(random_fiber_length)}. Volume occupied: {round(volume_occupied * 100 / (box_length * box_thickness * box_width), 5)}%')
```
### Plot output structure in Python (optionally)
```
# Plot the 3D structure (optional)
# fig = plt.figure()
# ax = fig.add_subplot(111, projection = '3d')
# ax.scatter(x_positions, y_positions, z_positions, c = 'b', marker = 'o', s = sphere_radius / 2)
# ax.set_xlabel('X')
# ax.set_ylabel('Y')
# ax.set_zlabel('Z')
# Show the plot
# plt.show()
```
### Write output coordinates to .xyz file
> Current block of code defines filename and filepath for an output .xyz file, write the number of the beads simulated, its coordinates, and fiber number. Note - each single value of bead coordinate is divided by bead radius for nicer visual representation of the fiber structure in Blender.
```
# Define filepath and filename of a .xyz output file
output_xyz_filename = ['STFGEN', '_', current_date, '_NETWORK_VC.xyz']
output_xyz_filename = ''.join(output_xyz_filename)
output_file_path = os.path.join(output_xyz_folder, output_xyz_filename)

# Write coordinates to an XYZ file
with open(output_file_path, 'w') as f:

    # Write the total number of atoms as the first line
    f.write(f'{len(x_positions)}\n')

    # Write header information as the second line (modify this based on Blender's requirements)
    f.write("STFGEN_0\n")
    
    # Write atomic coordinates
    for i in range(len(x_positions)):
        f.write(f'C {x_positions[i]/sphere_radius} {y_positions[i]/sphere_radius} {z_positions[i]/sphere_radius} {fiber_indices[i]}\n')
    
# Print terminal statement on code processing
print(f'Coordinates saved to {output_file_path}')
```
### Write output statistics
> Current block of code writes output statistics of bead generation, considering:
> * simulation supervisor;
> * theoretical values of box length, box width, box thickness, mean fiber length, standard deviation of fiber length, bead radius, bead volume, bead overlay, bead-bead overlay volume, bead-bead overlay ratio, occupied volume, standard deviation of bead-bead displacement;
> * experimental values of the number of beads simulated, fibers simulated, occupied volume simulated, mean fiber length simulated and standard deviation of fiber length simulated.
```
# Write output statistics
mean_fiber_length = np.mean(random_fiber_length)
std_fiber_length = np.std(random_fiber_length)

# Define filepath and filename of an output statistics file
output_stats_filename = f'STFGEN_{current_date}_STATS_ALL_VC.txt'
output_stats_filepath = os.path.join(output_stats_folder, output_stats_filename)

# Write output statistics
with open(output_stats_filepath, 'w') as txt_file:
    txt_file.write(f'Simulation Supervisor: ____\n')
    txt_file.write(f'Box length - theoretical: {box_length}\n')
    txt_file.write(f'Box width - theoretical: {box_width}\n')
    txt_file.write(f'Box thickness - theoretical: {box_thickness}\n')
    txt_file.write(f'Mean fiber length - theoretical: {round(fiber_length_mean, 5)}\n')
    txt_file.write(f'Standard deviation of fiber length - theoretical: {round(fiber_length_sd, 5)}\n')
    txt_file.write(f'Bead radius - theoretical: {round(sphere_radius, 5)}\n')
    txt_file.write(f'Bead overlay - theoretical: {round(sphere_overlay, 5)}\n')
    txt_file.write(f'Bead volume - theoretical: {round((4 / 3) * np.pi * sphere_radius**3, 5)}\n')
    txt_file.write(f'Bead-bead overlay volume - theoretical: {round(bead_bead_overlay_ratio, 5)}\n')
    txt_file.write(f'Occupied volume - theoretical: {round(max_volume_occupied, 5)}\n')
    txt_file.write(f'Standard deviation of bead-bead displacement (radians) - theoretical: {str(bead_bead_angle_sd)}\n')
    txt_file.write(f'Beads simulated: {len(x_positions)}\n')
    txt_file.write(f'Fibers simulated: {total_fibers}\n')
    txt_file.write(f'Occupied volume simulated: {round(volume_occupied / (box_length * box_thickness * box_width), 5)}\n')
    txt_file.write(f'Mean fiber length simulated: {round(mean_fiber_length, 5)}\n')
    txt_file.write(f'Standard deviation of fiber length simulated: {round(std_fiber_length, 5)}\n\n')

# Print terminal statement on code processing
print(f'Information saved to {output_stats_filepath}')
```
### Write output fiber length
```
# Define filepath and filename of an output fiber length file
output_length_filename = f'STFGEN_{current_date}_LENGTH_VC.txt'
output_length_filepath = os.path.join(output_length_folder, output_length_filename)

# Write output fiber length
with open(output_length_filepath, 'w') as txt_file:
    txt_file.write('List of fiber lengths:\n')
    for fiber_length in random_fiber_length:
        txt_file.write(f'{fiber_length}\n')

# Print terminal statement on code processing
print(f'Length saved to {output_length_filepath}')
```
### Examples
Fig. 3-5. Randomly generated fiber networks at:
* 2% volume occupation;
* 5 units bead radius;
* 50±5 units fiber length;
* 300 * 300 * 300 cubic units of simulation box;
* 1 units cutoff distance;
* 0±5 degrees of bead-bead displacement. 

Fig. 3. X-Y axis view.
![2_X_Y](https://github.com/vchibrikov/STFGEN/assets/98614057/9c4c81af-4f41-42d1-b239-f370ed95d6a2)

Fig. 4. X-Z axis view.
![2_X_Z](https://github.com/vchibrikov/STFGEN/assets/98614057/1d222d78-346f-4f9d-820c-0a78f9ceb6ab)

Fig. 5. Y-Z axis view.
![2_Y_Z](https://github.com/vchibrikov/STFGEN/assets/98614057/4aaba709-4c1f-4486-a20f-b515c5d984bf)


Fig. 6-8. Randomly generated fiber networks at:
* 5% volume occupation;
* 5 units bead radius;
* 50±5 units fiber length;
* 300 * 300 * 300 cubic units of simulation box;
* 1 units cutoff distance;
* 0±5 degrees of bead-bead displacement. 

Fig. 6. X-Y axis view.
![5_X_Y](https://github.com/vchibrikov/STFGEN/assets/98614057/5118b966-572f-4e01-a445-817ed05a4c3a)

Fig. 7. X-Z axis view.
![5_X_Z](https://github.com/vchibrikov/STFGEN/assets/98614057/d96e5406-f5a8-4df8-994b-c5faa5d94076)

Fig. 8. Y-Z axis view.
![5_Y_Z](https://github.com/vchibrikov/STFGEN/assets/98614057/fa4ba755-b6f0-46eb-bdaa-4220c180a9e5)


Fig. 9-11. Randomly generated fiber networks at:
* 10% volume occupation;
* 5 units bead radius;
* 50±5 units fiber length;
* 300 * 300 * 300 cubic units of simulation box;
* 1 units cutoff distance;
* 0±5 degrees of bead-bead displacement. 

Fig. 9. X-Y axis view.
![10_X_Y](https://github.com/vchibrikov/STFGEN/assets/98614057/212b0ed1-0f0f-4f20-8cca-728f886cef88)

Fig. 10. X-Z axis view.
![10_X_Z](https://github.com/vchibrikov/STFGEN/assets/98614057/d250f2e6-7796-4f65-8966-88907e3cc1dd)

Fig. 11. Y-Z axis view.
![10_Y_Z](https://github.com/vchibrikov/STFGEN/assets/98614057/c2487dec-fdad-49bc-8645-79abc1a95dcf)


Fig. 12-14. Randomly generated fiber networks at:
* 20% volume occupation;
* 5 units bead radius;
* 50±5 units fiber length;
* 300 * 300 * 300 cubic units of simulation box;
* 1 units cutoff distance;
* 0±5 degrees of bead-bead displacement. 

Fig. 12. X-Y axis view.
![20_X_Y](https://github.com/vchibrikov/STFGEN/assets/98614057/e067b520-3cbd-482c-ac89-304c49a1d85b)

Fig. 13. X-Z axis view.
![20_X_Z](https://github.com/vchibrikov/STFGEN/assets/98614057/47d309bd-b002-4551-8be0-c9dd7cf86bbe)

Fig. 14. Y-Z axis view.
![20_Y_Z](https://github.com/vchibrikov/STFGEN/assets/98614057/63d175a6-2e5a-4da1-834c-455b36273663)


### Limitations
* It is recommended and suggested to generate fiber network, in which bead radius is up to 1% of the minimum input box parameter (length, width, thickness), while the maximum fiber length does not exceed 20% of the of the minimum input box parameter (length, width, thickness).* At some boundary conditions (too small simulation box, too long fibers, too big bead radius), angle displacement values may allow for the generation of sharp interbead angles, appearing as fiber kink in macroscale.











