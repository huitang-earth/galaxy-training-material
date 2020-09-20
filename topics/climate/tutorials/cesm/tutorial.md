---
layout: tutorial_hands_on
title: General Circulation of the Atmosphere with Community Earth System Model (CESM)
zenodo_link: ''
questions:
- What is an Earth System Model?
- What is the Community Earth System Model (CESM)?
- Why using CESM?
- What are the different components of CESM?
- What is a control experiment?
- How to visualize CESM control model outputs?
- How to create a new climate scenario?
- How to analyze the CESM model results?
- How to share and publish data?
objectives:
- Learn to run the CESM model in a simple configuration (atmosphere only)
- Learn to analyze and visualize CESM data
- Learn to convert hybrid sigma-pressure (model) levels to pressure levels
- Learn to analyze and visualize control model outputs stored on a remote server
- Learn to setup new CESM experiments
- Learn to share and publish the results of your experiment
time_estimation: 12H
key_points:
- Community Earth System Model (CESM)
- xarray and dask for analyzing and visualizing model outputs
- Sharing and publishing CESM data
contributors:
- annefou

---


# Introduction
{:.no_toc}

> ### {% icon comment %} Comment
>
> This tutorial is significantly based on [The general circulation of the Atmosphere](https://nordicesmhub.github.io/GEO4962/).
>
{: .comment}

The practical aims at familiarzing you with the [Community Earth System Model (CESM)](http://www.cesm.ucar.edu/) using Galaxy interactive environment. 

> ### Agenda
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

> ### {% icon comment %} Background
>
> Add some background.
{:  .comment}

## CESM

## Get data

> ### {% icon hands_on %} Hands-on: Data upload
>
> 1. Create a new history for this tutorial. If you are not inspired, you can name it *cesm* for example...
>    {% include snippets/create_new_history.md %}
> 2. Import the file from [Zenodo](XXX) or from the shared data library
>
>    ```
>    https://zenodo.org/record/XXXX/files/XXX
>    ```
>
>    {% include snippets/import_via_link.md %}
>    {% include snippets/import_from_data_library.md %}
>
> 3. Check that the datatype is **XXX**
>
>    Files you uploaded are in XXX format. In Galaxy, Datatypes are, by default, automatically guessed. 
>
>    {% include snippets/change_datatype.md datatype="datatypes" %}
>
> 4. Rename Datasets
>
>    As "https://zenodo.org/record/XXX/files/XXX" is not a beautiful name and can give errors for some tools, it is a good practice to change the dataset name by something more meaningfull. For example by removing `https://zenodo.org/record/XXX/files/` to obtain `XXX`.
>
>    {% include snippets/rename_dataset.md %}
>
> 5. Add a tag to the dataset corresponding to `cesm`
>
>    {% include snippets/add_tag.md %}
>
{: .hands_on}



# CESM conda environment

1. Open Terminal
2. Switch to `cesm` conda environment

~~~
source activate cesm
~~~

# Get input data set for the use case

~~~
wget https://zenodo.org/record/3733998/files/inputdata_F2000climo-f19_g17.tar.gz
tar zxf inputdata_F2000climo-f19_g17.tar.gz --directory $HOME/
~~~

# Create a new CESM experiment

We will build and run CAM in its *standalone* configuration i.e. without having all the other components **active**.  

The basic workflow to run the CESM code is the following:

*   Create a New Case
*   Invoke `case.setup` to setup your newly created case
*   Build the Executable (`case.build`)
*   Run the Model and Output Data Flow (`case.submit`)

To create a new case, we will be using `create_newcase` script.  
There are many options and we won't discuss all of them. The online help provides information about how get the full usage of create_newcase.
 
~~~
create_newcase --help
~~~


> ## Command not found
> If you get an error when invoking `create_newcase` make sure you have loaded cesm in your environment:
> ~~~
> source activate cesm
> ~~~
{: .callout}

We can now create the case we will be running in this tutorial:

~~~
create_newcase --case cesm_cases/F2000climo-f19_g17 --compset F2000climo --res f19_g17 --machine espresso --run-unsupported
~~~

*   **case**: specifies the name and location of the case being created. It creates a new case in `$HOME/cases` and its name is `F2000climo-f19_g17`
*   **res**: specifies the model resolution (resolution of the grid). Each model resolution can be specified by its alias, short name or long name:
    *   alias: f19_g17 (atm/lnd_ocn/ice)

-   non-default grids are: atm:1.9x2.5  lnd:1.9x2.5  ocnice:gx1v7  
-   mask is: gx1v7
-   1.9x2.5 is FV 2-deg grid: with domain file(s): 
-   domain.lnd.fv1.9x2.5_gx1v6.090206.nc (only for mask: gx1v6 grid match: atm/lnd)
-   domain.ocn.1.9x2.5_gx1v6_090403.nc (only for mask: gx1v6 grid match: ocnice)
-   domain.lnd.fv1.9x2.5_gx1v7.181205.nc (only for mask: gx1v7 grid match: atm/lnd)
-   domain.ocn.fv1.9x2.5_gx1v7.181205.nc (only for mask: gx1v7 grid match: ocnice)
-   domain.aqua.fv1.9x2.5.nc (only for mask: null grid match: ocnice) 
						         
-   gx1v7 is displaced Greenland pole 1-deg grid with Caspian as a land feature: with domain file(s): 
-   $DIN_LOC_ROOT/share/domains/domain.ocn.gx1v7.151008.nc (only for grid match: atm/lnd)
-   $DIN_LOC_ROOT/share/domains/domain.ocn.gx1v7.151008.nc (only for grid match: ocnice) 
    
The full list of supported grid is given [here](http://www.cesm.ucar.edu/models/cesm2/config/2.0.0/grids.html).
*   **compset**: specifies the component set, i.e., component models, forcing scenarios and physics options for those models.  
    As for the resolution, the component set can be specified by its alias, short name or long name:
*   alias: F2000climo
*   long name: 2000_CAM60_CLM50%SP_CICE%PRES_DOCN%DOM_MOSART_CISM2%NOEVOLVE_SWAV  


The notation for the compset longname is:  

`TIME_ATM[%phys]_LND[%phys]_ICE[%phys]_OCN[%phys]_ROF[%phys]_GLC[%phys]_WAV[%phys][_BGC%phys]`

The compset longname has the specified order: **atm, lnd, ice, ocn, river, glc wave cesm-options**.  
    
Where:


- **Initialization Time**:	2000 
- **Atmosphere**:	CAM60	CAM cam6 physics
- **Land**:	CLM50%SP	clm5.0:Satellite phenology
- **Sea-Ice**:	CICE%PRES	Sea ICE (cice) model version 5 :prescribed cice
- **Ocean**:	DOCN%DOM	DOCN prescribed ocean mode
- **River runoff**:	MOSART	MOSART: MOdel for Scale Adaptive River Transport
- **Land Ice**:	CISM2%NOEVOLVE	cism2 (default, higher-order, can run in parallel):cism ice evolution turned off (this is the standard configuration unless you're explicitly interested in ice evolution):
- **Wave**:	SWAV	Stub wave component

    
The list of available component set is given [here](http://www.cesm.ucar.edu/models/cesm2/config/compsets.html). 
    

*   **mach**: specifies the machine where CESM will be compiled and run. We will be running CESM on Galaxy with conda espresso configuration files (`--mach espresso`)


Now you should have a new directory in `$HOME/cases/F2000climo-f19_g17` corresponding to our new case.

~~~
cd cesm_cases/F2000climo-f19_g17
~~~

Check the content of the directory and browse the sub-directories:  
- CaseDocs: namelists or similar
- SourceMods: this is where you can add local source code changes.
- Tools: a few utilities (we won't use them directly)
- Buildconf: configuration for building each component

For this tests (and all our simulations), we do not wish to have a "cold" start and we will therefore restart and continue an existing simulation we have previously run.  

~~~
./xmlchange RUN_TYPE=hybrid
./xmlchange RUN_REFCASE=F2000climo.f19_g17.control
./xmlchange RUN_REFDATE=0014-01-01
~~~

We use `xmlchange`, a small script to update variables (such as RUN_TYPE, RUN_REFCASE, etc.) defined in xml files. All the xml files contained in your test case directory will be used by cesm_setup to generate your configuration setup (Fortran namelist, etc.). 

~~~
ls *.xml
~~~


If we do not want the dates to start from 0001-01-01 we need to specify the starting date of our test simulation.

~~~
./xmlchange RUN_STARTDATE=0014-01-01
~~~

We are also going to change the duration of our test simulation and set it to 1 month only.  

~~~
./xmlchange  STOP_N=1
./xmlchange  STOP_OPTION=nmonths
~~~

Now we are ready to set-up our model configuration and build the cesm executable.  

1. Setup case

~~~
./case.setup
~~~

2. Build case

~~~
./case.build
~~~
 
 If successful, you would get:
 
 ~~~
 MODEL BUILD HAS FINISHED SUCCESSFULLY
 ~~~
 
After building CESM for your configuration, a new directory (and a set of sub-directories) are created in `$HOME/work/F2000climo-f19_g17`:

*   **bld**: contains the object and CESM executable (called **cesm.exe**) for your configuration
*   **run**: this directory will be used during your simulation run to generate output files, etc.

### Running a case

Namelists can be changed before configuring and building CESM but it can also be done before running your test case. Then, you cannot use xmlchange and update the xml files, you need to directly change the namelist files.  

The default history file from CAM is a *monthly* average, and this is what we are going to use in this lesson. 

However, it is possible to change the output frequency with the namelist variable **nhtfrq**

*   If nhtfrq=0, the file will be a monthly average
*   If nhtfrq>0, frequency is input as number of timesteps.
*   If nhtfrq<0, frequency is input as number of hours.

*For instance if we wanted to change the history file from monthly average to daily average, we would have to set the namelist variable **nhtfrq** to -24.*

**[cat](http://www.linfo.org/cat.html)** is a unix shell command to display the content of files or combine and create files. Using >> followed by a filename (here user_nl_cam) means we wish to concatenate information to a file. If it does not exist, it is automatically created. Using << followed by a string (here EOF) means that the content we wish to concatenate is not in a file but written after EOF until another EOF is found.  

Finally, we have to copy the control restart files (contains the state of the model at a given time so we can restart it). The files are stored on NIRD (they were generated from a previous simulation where the model was run for several years).



~~~
wget https://zenodo.org/record/3702975/files/F2000climo.f19_g17.control.rest.0014-01-01-00000.tar.gz
tar zxf F2000climo.f19_g17.control.rest.0014-01-01-00000.tar.gz --directory $HOME/work/F2000climo-f19_g17/run
mv $HOME/work/F2000climo-f19_g17/run/0014-01-01-00000/* $HOME/work/F2000climo-f19_g17/run/.
~~~ 

Now you can run your case:
~~~
./case.submit
~~~


### First look at your 1 month test run

During your test case run, CAM-6 generates outputs in the "run" directory.

At the end of your experiment, the run directory will only contain files that are needed to continue an existing simulation but all the model outputs are moved to another directory (archive directory). On Galaxy Climate JupyterLab this directory is semi-temporary which means data will be automatically deleted after a short period of time.  

Check your run was successful and generated all the necessary files you need for your analysis. 

~~~
ls -lrt $HOME/work/F2000climo-f19_g17/run/
~~~


~~~
ls -lrt $HOME/archive/F2000climo-f19_g17/atm/hist/
~~~
{: .language-bash}


#### What is a netCDF file?

Netcdf stands for “network Common Data Form”. It is self-describing, portable, metadata friendly, supported by many languages
(including python, R, fortran, C/C++, Matlab, NCL, etc.), viewing tools (like panoply, ncview/ncdump) and tool suites of file operators (in particular NCO and CDO).

#### Inspect a netCDF file

NetCDF files are often too big to open directly (with your favorite text editor, for instance), however one can look at the **content** of a netCDF file instead, for example to *dump* the **header** of one of the netCDF history files.

~~~
ncdump -h $HOME/work/archive/F2000climo-f19_g17/atm/hist/F2000climo-f19_g17.cam.h0.2000-06.nc
~~~

~~~
netcdf F2000climo-f19_g17.cam.h0.2000-06 {
dimensions:
        lat = 96 ;
	lon = 144 ;
	time = UNLIMITED ; // (1 currently)
        nbnd = 2 ;
        chars = 8 ;
        lev = 32 ;
        ilev = 33 ;
variables:
        double lat(lat) ;
                lat:_FillValue = -900. ;
                lat:long_name = "latitude" ;
                lat:units = "degrees_north" ;
        double lon(lon) ;
                lon:_FillValue = -900. ;
                lon:long_name = "longitude" ;
                lon:units = "degrees_east" ;
        double gw(lat) ;
                gw:_FillValue = -900. ;
                gw:long_name = "latitude weights" ;
    ....
~~~
{: .output}

# Visualize your model data with Python Notebook

# Conclusion
{:.no_toc}

We have now learnt how to run CESM and analyze climate data generated by CESM. 
