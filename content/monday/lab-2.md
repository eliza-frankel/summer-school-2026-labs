# Lab 2 - Exploring MESA Custom Colors!

The MESA colors module allows us to generate synthetic photometry while running MESA stellar evolution models! It is a great way to merge observational and theoretical astronomy. With the colors module, we can specify what filter system and stellar atmosphere we want to use, and on top of regular MESA outputs (effective temperature, luminosity, age, etc.) we get bolometric magnitude, M_bol, bolometric flux, F_bol, and many synthetic magnitudes. For more information on the colors module, look at https://github.com/MESAHub/mesa/tree/main/colors.

In this lab, we'll explore how changing some physics in the star can change the output on a color magnitude diagram (CMD). CMDs show the relationship between the brightness and color of a star. We'll also explore the differences between a star evolving in magnitude space versus theoretical space (HR diagrams).

### Step 1 - Lab 2 Prep

Last lab we made a working directory that has everything we want to start lab 2. The first thing we will do is copy lab 1 into a new working directory:

``` terminal
cp -r lab1 lab2
cd lab2
```
Lets clean this directory and get rid of our outputs from Lab 1:

``` terminal
./clean
./mk
rm -r LOGS
```

> [!CAUTION]
> ELIZA - UPDATE WITH WHATEVER OUTPUTS ARE FROM LAB 1 THAT THEY NEED TO REMOVE

### Step 2 - Building the inlist

For this lab, we are going to start with the same inlist as before, but we'll be adding a few things. 
We will be changing parameters in `&controls`, `&pgstar`, `&colors`. Let's start with `&controls`!

#### `&controls`

This is the section with the main stellar evolution parameters. Our goal is to change the stellar input parameters to see how they change evolution! The first thing we want to change is how the atmospheric boundary conditions are controlled. Look through the _controls_ tab under star defaults in the [documentation](https://docs.mesastar.org/en/26.4.1/reference.html) to the right parameters to change. What does it control specifically?

> [!tip]
> **HINT (add to drop down Eliza)**
> 
> The section _atmospheric boundary conditions_ has everything we'll need to start. `atm_option` is the main parameter we'll use. This changes how surface temperature (Tsurf) and surface pressure (Psurf) are evaluated at outer boundary conditions.

Lets first use a **T($\tau$)** relationship. This defines how the atmospheric pressure structure is obtained by integrating the hydrostatic equilibrium equation,

$$
\frac{dP}{d\tau} = \frac{g}{\kappa}.
$$

Here, we assume that gravity, _g_, is spatially constant. There are 3 options for the **T($\tau$)** relationship: `Eddington`, `solar_Hopf`, and `Krishna_Swamy`. Try changing the atmospheric boundary relation to each of these and run them.

> [!tip]
> **Solution (add to drop down Eliza)**
> ``` fortran
> atm_option = 'T_tau' 
> atm_T_tau_relation = 'Eddington' 
> atm_T_tau_opacity = 'varying' 
> ```





#### `&colors`

This is where we can enable synthetic photometry and determine what filters we'd like to use. Let's look through the [documentation](https://github.com/MESAHub/mesa/tree/main/colors).

The first thing we need to do is to make sure the colors module is on. By default, custom colors is turned off.

``` fortran {&colors}
use_colors = .true.
```

Next we need to decide what filter system, stellar atmosphere table, and Vega SED file to use. For this lab, we want to use the 2MASS filters and the Kurucz 2003 atmosphere tables. To find out what systems are available, let's move to data directory and start exploring! 

``` bash {}
cd $MESA_DIR/data/colors_data/
```

Once you've found the right filters and atmosphere tables, add them to your inlist.

``` fortran {&colors}
instrument = '/data/colors_data/ADD/FILTERS/HERE'
stellar_atm = '/data/colors_data/ADD/MODELS/HERE/'
vega_sed = '/data/colors_data/VEGA_SED/HERE/'
```

> [!tip]
> **Solution (add to drop down Eliza)**
> ``` fortran
> instrument = '/data/colors_data/filters/2MASS/2MASS'
> stellar_atm = '/data/colors_data/stellar_models/Kurucz2003all/'
> vega_sed = '/data/colors_data/stellar_models/vega_flam.csv'
> ```

> [!CAUTION]
> Proper syntax is important! Make sure that for the `instrument` directory there _isn't_ a '/' at the end, but for `stellar_atm` there _is_ a '/'

Now let's decide the distance of the star (in cm). For apparent magnitude, you can do any distance you want. For absolute magnitude, the distance should be 10 parsecs, or 3.0857 x 10<sup>19</sup> cm. Update the distance parameter to be 10 pc.

> [!tip]
> **Solution (add to drop down Eliza)**
> ``` fortran
> distance = 3.0857d19
> ```

