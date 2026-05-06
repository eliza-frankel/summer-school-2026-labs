# Lab 2 - Exploring MESA Custom Colors!


The MESA colors module allows us to generate synthetic photometry while running MESA stellar evolution models! It is a great way to merge observational and theoretical astronomy. With the colors module, we can specify what filter system and stellar atmosphere we want to use, and on top of regular MESA outputs (effective temperature, luminosity, age, etc.) we get bolometric magnitude, M$_{bol}$, bolometric flux, F$_{bol}$, and many synthetic magnitudes. For more information on the colors module, look at https://github.com/MESAHub/mesa/tree/main/colors.


One major age dating technique for stellar populations is through the use of isochrones. Isochrones are single-aged, chemically homogenous populations that show a snapshot of stellar evolution. They're made by evolving stars with the same chemical composition but different initial masses, and then finding what point in evolution each star is at at a particular age. Larger stars burn hotter and brighter, leaving the main sequence much quicker than a lower mass star. For example, at 10 Gyr we can see a 0.8 $M_{\odot}$ still on the main sequence, while a 5 $M_{\odot}$ star will be long past the Red Giant Branch. Because of this, we can build isochrones and use them to determine the age of stellar populations. One caveat to this is that they use the assumption that all the stars are at relatively the same distance and formed from the same materials at relatively the same time. _The best stellar populations to use isochrones when age dating stars is in clusters because we can make these assumptions._

**ELIZA ADD ISOCHRONES...FROM DR. JOYCE? FROM ME?**


In this lab, we'll learn how atmospheric boundary conditions and the convective mixing length parameter can impact stellar evolution in both observational and theoretical coordinates. We'll also build isochrones to explore other techniques for age dating stellar populations and planet hosts.


### Step 1 - Lab 2 Prep

Last lab we made a working directory that has everything we want to start lab 2. The first thing we will do is copy lab 1 into a new working directory:

```bash
cp -r lab1 lab2
cd lab2
```
Lets clean this directory and get rid of our outputs from Lab 1:

```bash
./clean
./mk
rm -r LOGS
```
In Lab 1, we explored magnetic braking. Let's turn it off for this lab in the `&controls` section of `inlist_project`

```fortran
! Enable magnetic braking.
use_other_torque    = .false.
```

> [!CAUTION]
> ELIZA - UPDATE WITH WHATEVER OUTPUTS ARE FROM LAB 1 THAT THEY NEED TO REMOVE

### Step 2 - Building the inlist

For this lab, we are going to start with the same inlist as before, but we'll be adding a few things. Start by opening up `inlist_project` in a text editor.
We will be changing parameters in `&colors`, `&controls`, and the `inlist_pgstar`. Let's start with `&colors`!

#### `&colors`

This is where we can enable synthetic photometry and determine what filters we'd like to use. Let's look through the [documentation](https://github.com/MESAHub/mesa/tree/main/colors).

The first thing we need to do is to make sure the colors module is on. By default, custom colors is turned off.

```fortran
use_colors = .true.
```

Next we need to decide what filter system, stellar atmosphere table, and Vega SED file to use. For this lab, we want to use the 2MASS filters and the Kurucz 2003 atmosphere tables. To find out what systems are available, let's move to data directory and start exploring! 

```bash
cd $MESA_DIR/data/colors_data/
```

Once you've found the right filters and atmosphere tables, add them to your inlist.

  {{< details title="Hint" closed="true" >}}

  ```fortran
  instrument = '/data/colors_data/ADD/FILTERS/HERE'
  stellar_atm = '/data/colors_data/ADD/MODELS/HERE/'
  vega_sed = '/data/colors_data/VEGA_SED/HERE/'
  ```
  {{< /details >}}




  {{< details title="Solution" closed="true" >}}

  ```fortran
  instrument = '/data/colors_data/filters/2MASS/2MASS'
  stellar_atm = '/data/colors_data/stellar_models/Kurucz2003all/'
  vega_sed = '/data/colors_data/stellar_models/vega_flam.csv'
  ```

  {{< /details >}}
  

> [!CAUTION]
> Proper syntax is important! Make sure that for the `instrument` directory there _isn't_ a '/' at the end, but for `stellar_atm` there _is_ a '/'

Now let's decide the distance of the star (in cm). For apparent magnitude, you can do any distance you want. For absolute magnitude, the distance should be 10 parsecs, or 3.0857 x 10<sup>19</sup> cm. Update the distance parameter to be 10 pc



{{< details title="Solution" closed="true" >}}

```fortran
  distance = 3.0857d19
```

{{< /details >}}


#### `&controls`

This is the section with the main stellar evolution parameters. Our goal is to change the stellar input parameters to see how they change evolution! The first thing we want to change is how the atmospheric boundary conditions are controlled. Look through the _controls_ tab under star defaults in the [documentation](https://docs.mesastar.org/en/26.4.1/reference.html) to the right parameters to change. What does it control specifically?


{{< details title="Hint" closed="true" >}}

The section atmospheric boundary conditions has everything we'll need to start. `atm_option` is the main parameter we'll use. This changes how surface temperature (Tsurf) and surface pressure (Psurf) are evaluated at outer boundary conditions.

{{< /details >}}

Lets first use a **T($\tau$)** relationship. This defines how the atmospheric pressure structure is obtained by integrating the hydrostatic equilibrium equation,

$$
\frac{dP}{d\tau} = \frac{g}{\kappa}.
$$

Here, we assume that gravity, _g_, is spatially constant. There are 3 options for the **T($\tau$)** relationship: `Eddington`, `solar_Hopf`, and `Krishna_Swamy`. Try changing the atmospheric boundary relation to each of these and run them.


  {{< details title="Solution" closed="true" >}}

  ```fortran
  atm_option = 'T_tau' 
  atm_T_tau_relation = 'Eddington' 
  atm_T_tau_opacity = 'varying' 
  ```

  {{< /details >}}



#### `inlist_pgstar`
Now let's edit the custom plots we see as our star evolves. The first thing we are going to do is either remove or comment out the custom plots we used in Lab 1.
Remove the following lines:

```fortran
Grid1_plot_name(7) = 'History_Track2'

History_Track2_title = 'total AM'
History_Track2_xname = 'log_star_age'
History_Track2_yname = 'log_total_angular_momentum'
History_Track2_xaxis_label = 'log_star_age'
History_Track2_yaxis_label = 'log_total_angular_momentum'

History_Track2_ymin = 47
History_Track2_ymax = 50
```

While the star evolves, we want to see how an HR diagram compares to a color-magnitude diagram. Luckily, we have an HR diagram already there! 

**Question** Let's evolve the star using `./rn` and wait until our plotting windows pop up. What do you notice about the HR diagram?

{{< details title="Answer" closed="true" >}}

Nothing shows up in the plotting window! This is because of the lines under "! set static plot bounds" - our star is evolving outside of the log(Teff) and log(L) limits given. Let's remove the following lines:

```fortran
! set static plot bounds
 HR_logT_min = 3.5
 HR_logT_max = 4.6
 HR_logL_min = 2.0
 HR_logL_max = 6.0
```

{{< /details >}}


