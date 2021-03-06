[![NSF-1948997](https://img.shields.io/badge/NSF-1948997-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1948997) 
[![NSF-1948994](https://img.shields.io/badge/NSF-1948994-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1948994)
[![NSF-1948857](https://img.shields.io/badge/NSF-1948857-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1948857)

OT QA/QC Utility Tools

*  Summary
-  This repo consists of python scripts and templates to better automate
   and standardize the first-level QA/QC of data to be ingested into
   OpenTopography.  The scripts will check the lidar files for missing
   Coordinate Reference System (CRS) info, las versions, etc.  It also
   checks for uniformity of values amoung all the files, as well as
   creating boundaries of the data, and calculating area.

*  How to run the QA/QC Software
-  Clone this repo:  git clone https://www.unavco.org/gitlab/beckley/otQAQC.git
-  Rename cloned directory to OT shortname (mv otQAQC/ new_shortname)
-  cd into newly renamed directory
-  run the main level code in python (version 3), and specify your working
   directory as the first argument:

   python ot_utils.py $PWD

   This will set up the directory structure with all the necessary
   template files.
-  edit the configuration file, ingest_shortname.py as necessary.
   Specify the location of the data, name of log files, which modules to
   run, etc.
-  See section below on configuration details


*  Configuration File Description
-  To run the QA/QC code, you should initialize a configuration file to
   nulls by doing:  config = ot.initializeNullConfig().  Then set
   whatever parameters you want.  It's best to run multiple QA/QCs if
   you have multiple steps to do (ex: convert to LAZ first, then run
   Boundary creation)

-  *ingestBase*.  This is generally your base working directory and will
   be taken automatically if you are running the scripts with the default
   directory structure.  In general, this will be something like:
   
   ex: /Users/matt/OT/DataIngest/WA18_Wall

-  *shortname*.  This is the name you are going to use for the OT ingest.
   In general, your directory structure will contain this name, and you
   will use this throughout the process to identify this project.  By
   default, this is taken automatically from the ingestBase variable.
   
   ex:  WA18_Wall

-  *bounds_base*.  This is the directory that will hold all the work for
   creating the boundary of the data.  Boundaries can be created in either
   PDAL and LASTools.  They do yield slightly different results, and
   currently LASTools is *MUCH* faster.  If using the default directory
   structure, this variable will be taken automatically from the
   ingestBase variable.

-  *log_dir*.  This is the directory that will hold all the logs.  There
   is a log that contains messages as the ingest process proceeds, but
   there is also a log that is the PDAL metadata output from all the
   las/laz files.  The majority of error checking is done based on the
   PDAL metadata log file.  If using the default directory structure,
   this variable will be taken automatically from the ingestBase
   variable.

-  *scripts_dir*.  This is the directory that holds the ingest script as
   well as a sample PDAL pipeline file if needed.  The pipeline file
   should only be needed if you have to convert the files from LAS to LAZ,
   or some other PDAL operation.  If using the default directory structure,
   this variable will be taken automatically from the ingestBase
   variable.

-  *log_dir*.  Directory where you want the ingest log to be written.  This
   is usually: /Users/matt/OT/DataIngest/shortname/logs
 
-  *ingestLog*.  Full path to ingest log.  Filename is usually in the form:
                 shortname+'_ingestLog.txt'

-  *AddCRS2Header*.  Set this to 1 if you want to simply add the CRS
   info to the lidar header files.  Currently uses PDAL for this.

-  *ftype*.  This is set to either 'f' of 'd' for 'file' or
   'directory'.  This value is getting fed directly to a unix find call.
   This keyword is only used with the module, getFiles.  The default is set
   to 'f'.  The only time where setting it to 'd' will be used is when
   converting ESRI grid files, which are are set of files in a directory.
   With ESRI grid files, if you specify the directory name, GDAL will be able 
   to convert it.

-  *LAS2LAZ*.  Set this to 1 if you want to convert LAS files to LAZ
   otherwise set to 0.

-  *LAS2LAZ_method*.  Set this to 'lastools' or 'pdal'.  With pdal, it
   will read in a pipeline (the path to which you specify), and can do
   multiple operations in one run.  So, you can convert to LAZ, and
   reproject in one step.  But it is sometimes slower than LASTools.  With
   LASTools, it will just convert to LAZ.  If your original LAS files do
   not have CRS info, you will have to define them in a separate run, so it
   is a two step process.  Consider adding another EPSG keyword to feed to
   LAStools, but the config file is already confusing.
  
-  *getFilesWild*.  Regular expression to find all the files. you want to
   work on.  It's best to search by suffix: '.*\.laz$'

-  *getFilesDir*.  Base directory from where file search will start.  If
   recursive is set it will start to drill down from this directory.

-  *recursive*.  Set this to 1 if the files to ingest are in a nested
   directory structure.  This is pretty rare.  Mostly all the las/laz
   files will be in single directory, so generally leave this set to 0.
   This keyword only gets used in the module: getFiles.

-  *LAZDir_out*.  Directory where LAZ files will be written when converted
   from LAS.

-  *pipeline*.  Full path to pipeline file.  This is a JSON file that will
   be used by PDAL to do either a conversion or translation.  If LAS2LAZ
   is set, and you want to use PDAL, then you must supply a pipeline.

-  *CreatePDALInfo*.  Set this to 1 if you want to loop through all the
   LAS/LAZ files and create a PDAL log of all the metadata.  This file is
   usually stored in the logs, and used for most of the QA/QC

-  *PDALInfoFile*.  Name of the logfile containing all the PDAL metadata.
   Default is:  shortname+'_PDALInfoLog.txt'

-  *ReadPDALLog*.  Set this to 1 if you want to read in the PDAL log into
   an array for doing QAQC.  *You will need this for most operations.*

-  *CheckLAZCount*.  Set this to 1 if you want to check the count of LAZ
   files.  This is only mildly useful, and will report if there are
   other files other than LAZ in the ingest directory.

-  *MissingHCRS*.  Set this to 1 if you want to check in any of the LAZ
   files are missing the Horizontal Coordinate System Info in the
   header.  If at least 1 is missing, it will throw an error.  This is
   a serious error, so the code will enter the debugger if this occurs.
   This will help troubleshoot which file is missing the HCRS

-  *MissingVCRS*.  Set this to 1 if you want to check in any of the LAZ
   files are missing the Vertical Coordinate System Info in the
   header.  If at least 1 is missing, it will through an warning.  Code
   will not stop because many datasets don't have any vertical info.  A
   note is made in the log, but the ingest process does not stop

-  *HCRS_Uniform*.  Set this to 1 if you want to check that all of the LAZ
   files are in the same Horizontal Coordinate System.  If more than 1
   HCRS is detected, it will throw an error.  This is a serious error, so
   the code will enter the debugger if this occurs.

-  *VCRS_Uniform*.  Set this to 1 if you want to check that all of the LAZ
   files are in the same Vertical Coordinate System.  If more than 1
   VCRS is detected, it will throw an error.  This is a serious error, so
   the code will enter the debugger if this occurs.

-  *VersionCheck*.  Set this to 1 if you want to check that all the
   LAS/LAZ files are in the same version.

-  *PointTypeCheck*.  Set this to 1 if you want to check that all the
   LAS/LAZ files have the same 'Point Type' value.

-  *GlobalEncodingCheck*.  Set this to 1 if you want to check that all the
   LAS/LAZ files have the same 'Global Encoding' value.

-  *PointCountCheck*.  Set this to 1 if you want to check to make sure
   that all the lidar files have points.  If this module finds any points
   that have a point count of 0, it will issue a warning, but will not stop
   execution of the code.

-  *CreatePDALBoundary*.  Set this to 1 if you want to create a boundary
   of the datasets using PDAL.  PDAL uses a different method than
   LASTools, and there are several steps involved.  It is *MUCH* slower,
   and also seems a bit buggy.

-  *bounds_PDAL*.  Full path of shapefile that will be the initial
   boundary created from PDAL.  This file will usually be in segments, and
   needs to be dissolved with a later step.  Example value is:

   /Users/matt/OT/DataIngest/shortname/bounds/Boundary_PDAL.shp

-  *BufferSize*.  When doing the dissolve, sometimes you need to specify a
   small buffer to remove any anomalies.  Enter a value in meters.  Usually
   1 or 2 meters is fine to give good results.  This is only used when
   creating a boundary with PDAL.

-  *epsg*.  Set this to the EPSG code for the dataset.  This is only used
   when creating a boundary with PDAL.

-  *bounds_PDALmerge*.  Full path to a shapefile that will contain the
   dissolved/merged version of initial shapefile that was created.

-  *bounds_PDALmergeArea*.  Full path to a shapefile that will contain the
   area of the polygon added to the attribute table (in KM^2).

-  *bounds_PDALKML*.  Full path to the KML version of the final PDAL
   shapefile that is merged and contains the area in the attribute table.

-  *CreateLASBoundary*.  Set this to 1 if you want to create a boundary of
   the dataset using LASTools.

-  *winePath*.  Path to LASTools executables.  Default is:
                /Applications/LASTools/bin

-  *bounds_LT*.  Full path to a shapefile that will contain the boundary
   created by LASTools.

-  *randFrac*.  This is an abbrevation for "Random Fraction", and is a
   parameter that is fed into lasboundary.  This specifies the amount of
   randomly selected data to keep for processing.  This speeds the process
   up greatly.  Usually best to keep this set to 0.30 (30 %) or less.

-  *concavity*.  This is another parameter to lasboundary. The default is
   100, meaning that voids with distances of more than 100 meters are
   considered the exterior (or part of an interior hole)

-  *bounds_LTArea*.  Full path to shapefile that will add the area in KM^2
   to the boundary shapefile initially created by LASTools.  

-  *bounds_LTKML*.  Full path to the KML version of the LASTools-derived
   boundary shapefile that contains the area in the attribute table.

-  *CheckRasMeta*.  Set this to 1 if you want to get an initial check of
   the raster metadata.  This is good to do as a first check to see if the
   rasters have CRS, or are in different formats, etc.

-  *SetRasterCRS*.  Set this to 1 if you just need to add the CRS info
   to the raster header.  Note this does not do any reprojection.  It is
   simply adding the CRS info to the header of the rasters.

-  *a_srs*.  Set this to a EPSG code string.  This only gets used by the
   module, SetRasterCRS.  Value should be only the numeric code, but in 
   string form.  ex: '6339'

-  *Translate2Tiff*.  Set this to 1 if you want to convert raster files to
   tiffs.  Note you set getFilesWild to get the files you want to
   convert.  This just converts the file type, and *does not* do
   reprojection.  

-  *RasOutDir*.  Directory where you want to write out the newly created
   raster files.  If not set, output files will be written to same
   directory as input files.

-  *Warp2Tiff*.  Set this to 1 if you want to reproject the tiff
   files. Note you set getFilesWild to get the files you want to convert.
   You can specify a single output directory by setting RasOutDir=1,
   otherwise, output files will be written to the same directory as the
   input files.

-  *ras_xBlock*.  This is the size of the tiles that gdal will tile at in
   the X direction.  This is usually: 128, 256, or 512.  default is set
   to 256.  This keyword is only used in modules: Translate2TIFF, and
   Warp2TIFF.    

-  *ras_yBlock*.  This is the size of the tiles that gdal will tile at in
   the Y direction.  This is usually: 128, 256, or 512.  default is set
   to 256.  This keyword is only used in modules: Translate2TIFF, and
   Warp2TIFF.

-  *warp_t_srs*.  This is the EPSG code that you want the newly
   projected tiff to be in.  Input file must contain SRS info in the
   header.  Value should be only the numeric code, but in string form.
   ex: '6339'.  This keyword is only used for module, Warp2TIFF.
