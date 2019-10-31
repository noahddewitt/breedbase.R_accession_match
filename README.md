breeDBase.R
===========
### An R package for generating breeDBase upload templates

This R package (**currently a work in progress**) can be used to create classes of 
various breeDBase data types (such as an Accession, a phenotyping trial Plot, etc).  
One or more instances of a class can be passed to a `buildTemplate` or `writeTemplate` 
function to create and/or write an upload template to be used for adding data through 
the breeDBase website.


## Installation

Currently, the breedbase package can be installed directly from GitHub using `devtools`.

```R
library(devtools)
install_github("TriticeaeToolbox/breeDBase.R")
```


## Documentation

Documentation for all Classes and functions can be found in the package's R documentation 
or online at [https://triticeaetoolbox.github.io/breeDBase.R/reference](https://triticeaetoolbox.github.io/breeDBase.R/reference).


## Examples


#### Accessions and Pedigrees

1) Create new Accessions

```R
jerry <- Accession(
     accession_name = "JERRY", 
     species_name = "Triticum aestivum",
     properties = list(
         synonyms = c("ND9257", "PI632433"),
         institute_codes = "NDSU",
         organization_names = "North Dakota State University"
     )
)

caledonia <- Accession(
     accession_name = "CALEDONIA", 
     species_name = "Triticum aestivum",
     properties = list(
         synonyms = "PI610188",
         institute_codes = "CNL",
         organization_names = "Cornell University"
     )
)

my_cross <- Accession(
    accession_name = "MY_CROSS",
    species_name = "Triticum aestivum",
    properties = list(
        institute_codes = "CNL",
        organization_names = "Cornell University"
    )
)

# Add all accessions to a vector
accessions <- c(jerry, caledonia, my_cross)
```


2) Create a Pedigree for the `MY_CROSS` Accession

```R
my_cross_pedigree <- Pedigree(my_cross, female_parent = jerry, male_parent = caledonia)
```


3) Write the Accession and Pedigree upload templates to files

```R
writeAccessionTemplate(accessions, '/path/to/accessions.xls')
writePedigreeTemplate(my_cross_pedigree, "/path/to/pedigrees.txt")
```


#### Locations

Locations are required to have latitude and longitude coordinates as well as their elevation. 
These can be added manually as parameters or geocoded using the 
[DataScienceToolkit](http://www.datasciencetoolkit.org/) API.

- Add location information directly as parameters, if it is already known

```R
loc1 <- Location(
    name = "Batavia, NY",
    abbreviation = "BAT",
    country_code = "USA",
    country_name = "United States of America",
    program = "Cornell",
    type = "Field",
    latitude = 42.98054,
    longitude = -78.23176,
    altitude = 274
)
```

- If either the `latitude`, `longitude` or `altitude` parameters are not provided, 
they will be queried using the `name` as the location.

```R
loc2 <- Location(
    name = "Batavia, NY",
    abbreviation = "BAT",
    country_code = "USA",
    country_name = "United States of America",
    program = "Cornell",
    type = "Field"
)
```

- Alternatively, you can query a more specific location string separately and use 
the results to build the Location
```R
geo <- geocodeLocation("2 Caldwell Drive, Ithaca, NY")
loc3 <- Location(
     name = "Caldwell - Ithaca, NY", 
     abbreviation = "ITH_CALD",
     country_code = "USA",
     country_name = "United States of America",
     program = "Cornell",
     type = "Field",
     latitude = geo$latitude,
     longitude = geo$longitude,
     altitude = geo$altitude
)
```


#### Plots

A series of Plots are used to describe a phenotyping trial layout.  Each plot is assigned 
a unique name, a plot number, the name of the Accession used in the plot, and the block 
it is located in.  If the physical layout of the plots is known, each Plot can be assigned 
a row and column number.

 - A plot layout can be generated by manually creating each Plot with the desired parameters

```R
# Create the Plots
plot1 <- Plot(
    plot_name = "FARM-2019-UNH_PLOT1", 
    accession_name = "SL18-UCONN-S131", 
    plot_number = 1, 
    block_number = 1,
    properties = list(
        row_numer = 1,
        col_number = 1
    )
)
plot1 <- Plot(
    plot_name = "FARM-2019-UNH_PLOT2", 
    accession_name = "SL18-UCONN-S31", 
    plot_number = 2, 
    block_number = 1,
    properties = list(
        row_numer = 1,
        col_number = 2
    )
)

# Combine the Plots for the layout
plots <- c(plot1, plot2)

# Write the Plot layout template
writePlotLayout(plots, "/path/to/plots.xls")
```

- Alternatively, a simple plot layout can be generated automatically when given a list of 
Accessions and basic layout properties.

**Example**: A trial with 24 plots that has a maximum of 6 columns.  The blocks have a
dimension of 3 columns by 2 rows.  The Plots will start in the top left corner and work 
across columns and down rows.
    
```R
# Generate the list of Accessions
# here we're using a dummy list of Accessions named ACC_A - ACC_X
accessions <- lapply(LETTERS[c(1:24)], function(x) { Accession(paste0("ACC_", x), "Saccharina latissima") })
   
# Create the Plots
plots <- createPlots(
    trial_name = "TEST-TRIAL", 
    accessions = accessions, 
    max_cols = 6, 
    max_cols_per_block = 3, 
    max_rows_per_block = 2
)
  
# The generated layout:
#                                     Col1                Col2                Col3                Col4                Col5                Col6
# ===== Row1 =====      ===== Plot 1 =====  ===== Plot 2 =====  ===== Plot 3 =====  ===== Plot 4 =====  ===== Plot 5 =====  ===== Plot 6 =====
# Row1: Plot Name         TEST-TRIAL_PLOT1    TEST-TRIAL_PLOT2    TEST-TRIAL_PLOT3    TEST-TRIAL_PLOT4    TEST-TRIAL_PLOT5    TEST-TRIAL_PLOT6
# Row1: Accession Name               ACC_A               ACC_B               ACC_C               ACC_D               ACC_E               ACC_F
# Row1: Block                            1                   1                   1                   2                   2                   2
# Row1: Control                      FALSE               FALSE               FALSE               FALSE               FALSE               FALSE
# ===== Row2 =====      ===== Plot 7 =====  ===== Plot 8 =====  ===== Plot 9 ===== ===== Plot 10 ===== ===== Plot 11 ===== ===== Plot 12 =====
# Row2: Plot Name         TEST-TRIAL_PLOT7    TEST-TRIAL_PLOT8    TEST-TRIAL_PLOT9   TEST-TRIAL_PLOT10   TEST-TRIAL_PLOT11   TEST-TRIAL_PLOT12
# Row2: Accession Name               ACC_G               ACC_H               ACC_I               ACC_J               ACC_K               ACC_L
# Row2: Block                            1                   1                   1                   2                   2                   2
# Row2: Control                      FALSE               FALSE               FALSE               FALSE               FALSE               FALSE
# ===== Row3 =====     ===== Plot 13 ===== ===== Plot 14 ===== ===== Plot 15 ===== ===== Plot 16 ===== ===== Plot 17 ===== ===== Plot 18 =====
# Row3: Plot Name        TEST-TRIAL_PLOT13   TEST-TRIAL_PLOT14   TEST-TRIAL_PLOT15   TEST-TRIAL_PLOT16   TEST-TRIAL_PLOT17   TEST-TRIAL_PLOT18
# Row3: Accession Name               ACC_M               ACC_N               ACC_O               ACC_P               ACC_Q               ACC_R
# Row3: Block                            3                   3                   3                   4                   4                   4
# Row3: Control                      FALSE               FALSE               FALSE               FALSE               FALSE               FALSE
# ===== Row4 =====     ===== Plot 19 ===== ===== Plot 20 ===== ===== Plot 21 ===== ===== Plot 22 ===== ===== Plot 23 ===== ===== Plot 24 =====
# Row4: Plot Name        TEST-TRIAL_PLOT19   TEST-TRIAL_PLOT20   TEST-TRIAL_PLOT21   TEST-TRIAL_PLOT22   TEST-TRIAL_PLOT23   TEST-TRIAL_PLOT24
# Row4: Accession Name               ACC_S               ACC_T               ACC_U               ACC_V               ACC_W               ACC_X
# Row4: Block                            3                   3                   3                   4                   4                   4
# Row4: Control                      FALSE               FALSE               FALSE               FALSE               FALSE               FALSE 
```

**Example:** The same 24 plots, but the plots are assigned using a zig-zag method (ie left to right in the first row, right to left in the second row, etc).  Additionally, the controls are specified using the names of the Accessions used as controls.

```R
# Generate the list of Accessions
accessions <- lapply(LETTERS[c(1:24)], function(x) { Accession(paste0("ACC_", x), "Saccharina latissima") })

# Create the Plots
plots <- createPlots(
    trial_name = "TEST-TRIAL", 
    accessions = accessions, 
    max_cols = 6, 
    max_cols_per_block = 3, 
    max_rows_per_block = 2,
    zig_zag = TRUE,
    controls = c("ACC_D", "ACC_L", "ACC_O", "ACC_T")
)

# The generated layout:
# ===== Row1 =====      ===== Plot 1 =====  ===== Plot 2 =====  ===== Plot 3 =====  ===== Plot 4 =====  ===== Plot 5 =====  ===== Plot 6 =====
# Row1: Plot Name         TEST-TRIAL_PLOT1    TEST-TRIAL_PLOT2    TEST-TRIAL_PLOT3    TEST-TRIAL_PLOT4    TEST-TRIAL_PLOT5    TEST-TRIAL_PLOT6
# Row1: Accession Name               ACC_A               ACC_B               ACC_C               ACC_D               ACC_E               ACC_F
# Row1: Block                            1                   1                   1                   2                   2                   2
# Row1: Control                      FALSE               FALSE               FALSE                TRUE               FALSE               FALSE
# ===== Row2 =====     ===== Plot 12 ===== ===== Plot 11 ===== ===== Plot 10 =====  ===== Plot 9 =====  ===== Plot 8 =====  ===== Plot 7 =====
# Row2: Plot Name        TEST-TRIAL_PLOT12   TEST-TRIAL_PLOT11   TEST-TRIAL_PLOT10    TEST-TRIAL_PLOT9    TEST-TRIAL_PLOT8    TEST-TRIAL_PLOT7
# Row2: Accession Name               ACC_L               ACC_K               ACC_J               ACC_I               ACC_H               ACC_G
# Row2: Block                            1                   1                   1                   2                   2                   2
# Row2: Control                       TRUE               FALSE               FALSE               FALSE               FALSE               FALSE
# ===== Row3 =====     ===== Plot 13 ===== ===== Plot 14 ===== ===== Plot 15 ===== ===== Plot 16 ===== ===== Plot 17 ===== ===== Plot 18 =====
# Row3: Plot Name        TEST-TRIAL_PLOT13   TEST-TRIAL_PLOT14   TEST-TRIAL_PLOT15   TEST-TRIAL_PLOT16   TEST-TRIAL_PLOT17   TEST-TRIAL_PLOT18
# Row3: Accession Name               ACC_M               ACC_N               ACC_O               ACC_P               ACC_Q               ACC_R
# Row3: Block                            3                   3                   3                   4                   4                   4
# Row3: Control                      FALSE               FALSE                TRUE               FALSE               FALSE               FALSE
# ===== Row4 =====     ===== Plot 24 ===== ===== Plot 23 ===== ===== Plot 22 ===== ===== Plot 21 ===== ===== Plot 20 ===== ===== Plot 19 =====
# Row4: Plot Name        TEST-TRIAL_PLOT24   TEST-TRIAL_PLOT23   TEST-TRIAL_PLOT22   TEST-TRIAL_PLOT21   TEST-TRIAL_PLOT20   TEST-TRIAL_PLOT19
# Row4: Accession Name               ACC_X               ACC_W               ACC_V               ACC_U               ACC_T               ACC_S
# Row4: Block                            3                   3                   3                   4                   4                   4
# Row4: Control                      FALSE               FALSE               FALSE               FALSE                TRUE               FALSE
```



## Current Status

The package currently contains functions for the following data types:

- Accession
- Pedigree
- Location
- Plot