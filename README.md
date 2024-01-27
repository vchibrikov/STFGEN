# STFGEN.py
STFGEN (Single-Type Fiber GENerator) is a hardcoded Python script for the generation of random fiber network according to input parameters. Current script allow to adjust input data on fiber structural (number of fibers, its length range, diameter, angle displacement, etc.) and spatial (volume occupation) properties. and provide output data of 3D bead coordinates in .xyz file format, as well as output information on generator run in .txt file format. Current repositorium consists of two Python scripts - STFGEN_O.py, which allows to generate fiber network with no bead overlays, and STFGEN_N.py, which do the same but do not handle bead overlay issues.

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
```













Figure of randomly generated fiber networks at 1% volume occupation with beads of the radius of 25 units. X-Y axis view.
![Figure 1](https://github.com/vchibrikov/STFGEN/assets/98614057/b81a64a0-bb66-4b5e-a116-083f0e4dafe2)

Figure of randomly generated fiber networks at 1% volume occupation with beads of the radius of 25 units. X-Z axis view.
![Figure_2](https://github.com/vchibrikov/STFGEN/assets/98614057/f374494a-84d3-4160-be29-0d11c051be39)

Figure of randomly generated fiber networks at 1% volume occupation with beads of the radius of 25 units. Y-Z axis view.
![Figure 3](https://github.com/vchibrikov/STFGEN/assets/98614057/fc560869-99dd-4530-af24-135e04064374)

Figure of randomly generated fiber networks at 5% volume occupation with beads of the radius of 25 units. X-Y axis view.
![Figure 4](https://github.com/vchibrikov/STFGEN/assets/98614057/7a9df709-bd09-443b-a548-ea7fc96eff2a)

Figure of randomly generated fiber networks at 5% volume occupation with beads of the radius of 25 units. Z-X axis view.
![Figure 5](https://github.com/vchibrikov/STFGEN/assets/98614057/8dfeb2c4-0a9b-4f87-b992-b6ef63faab42)

Figure of randomly generated fiber networks at 5% volume occupation with beads of the radius of 25 units. Z-Y axis view.
![Figure 6](https://github.com/vchibrikov/STFGEN/assets/98614057/230dc2e0-0345-470c-8f09-229d60381e0e)

> Limitations:
> 1. It is recommended to generate fiber network, in which bead radius is no more than 1% of the length of simulation box
> 2. Despite parameters of the simulation box are provided, current script does allow generated fibers to run over the box boundaries. The volume of the beads that run over the boundaries of simulation box are not considere in current script.










