# LCC2005

Alex Chubaty & Eliot McIntire
12 May 2017

This is an example model of forest dynamics.
It includes simple versions of forest succession in Canada's boreal forest, a simple cellular automaton fire model, some GIS data operations to select subset areas of a larger dataset, a reclassification module from a many-class land cover to a few-class land cover, and a simple agent movement model.
These are all toy versions of each of these types and so are not useful for real world applications; but they allow highlighting of some of `SpaDES`'s capabilities.

This example uses the notion of a 'module group' or 'parent' module that contains several child modules:

- `caribouMovementLcc`
- `cropReprojectLccAge`
- `fireSpreadLcc`
- `forestAge`
- `forestSuccessionBeacons`
- `LccToBeaconsReclassify`

This code also shows how modules can have different time units, specifically, the child modules which have 2 different time units (month for caribou, year for others).

The code will download all modules and data required to run the `spades` call at the end.
Visualizations will be quick because a device will be opened that is not within RStudio.

```{r download-modules, eval=FALSE}
library(SpaDES)

# set the main working directory
workDirectory <- file.path(dirname(tempdir()), "LCC2005")

# set the directories
inputDir <-  file.path(workDirectory, "simInputs")
moduleDir <- file.path(workDirectory, "modules")
outputDir <- file.path(workDirectory, "simOutputs")
cacheDir <- file.path(workDirectory, "cache")

setPaths(modulePath = moduleDir, inputPath = inputDir, outputPath = outputDir, cachePath = cacheDir)

# Alternatively, directory paths can be created using `checkPaths`
checkPath(moduleDir, create = TRUE)

downloadModule("LCC2005", moduleDir) # default `data=FALSE` doesn't download data

```



"LCC2005" is a module group. Module groups make loading multiple modules easier: only the name of the module group needs to be specified in the `simInit` call, which will then initialize the simulation with the child modules. 

```{r module-group-init, eval=FALSE}
# setup simulation

times <- list(start = 2005.0, end = 2020.0, timeunit = "year")
parameters <- list(
  .globals = list(burnStats = "fireStats"),
  fireSpreadLcc = list(drought = 1.2), # in
  caribouMovementLcc = list(N = 1e3, startTime = times$start + 1, 
                            glmInitialTime = NA_real_)
)
modules <- list("LCC2005")
paths <- list(
  cachePath = cacheDir,
  modulePath = moduleDir,
  inputPath = inputDir,
  outputPath = outputDir
)

# This next step will set up the simulation using the defined parameters. It will also download data if they do not yet exist locally
options("spades.moduleCodeChecks" = FALSE) # code checking is for advanced users
mySim <- simInit(times = times, params = parameters, modules = modules,
                 paths = paths)
```

Now that the `mySim` object has been initialized, we can run the simulation by calling `spades`:

```{r run-SpaDES, eval=FALSE}
dev.useRSGD(FALSE) # do not use Rstudio graphics device
dev() # opens external (non-RStudio) device, which is faster
clearPlot()

mySimOut <- spades(mySim)

# Plot Canada Map
Plot(mySimOut$lcc05, title = "Land Cover, with simulated area")
# Plot small Yellow polygon showing area simulated
Plot(mySimOut$inputMapPolygon, addTo = "mySimOut$lcc05", gp = gpar(col = "yellow", lwd = 2))
```

In addition to running a model, there are many things built into SpaDES that allow a user to explore the modules and models.

```{r Exploring model, eval = FALSE}
openModules(mySim) # this will try to open the modules in a text editor, or give instructions how to do it manually

### Simulation overview: note the child modules are initialized
moduleDiagram(mySim)
objectDiagram(mySim)

# show modules that are included in this simList
modules(mySim)

# show all objects contained within the final simList
ls(mySimOut)

# more detail
ls.str(mySimOut)

# show parameters that can easily be changed in simInit call
params(mySim)
```
