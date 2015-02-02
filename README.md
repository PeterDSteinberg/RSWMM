# RSWMM
<h3>Autocalibration for EPA Stormwater Management Model (SWMM) version 5 using multi- or single objective optimization in R.
</h3>
<h5>You're fully responsible for the any problems related to this software - I'm not maintaining it currently.  It was tested with 5.0.022.  It looks like if moving to SWMM version 5.1+, the binary file reader in RSWMM.r would have to be changed (probably a minor change). See item 33 under "Build 5.1.001 (3/24/2014)" on this<a href="http://www2.epa.gov/sites/production/files/2014-10/epaswmm5_updates_0.txt"> EPA page listing SWMM changes by version.</a></h5>
<h6>Synopsis:</h6><p>This R code runs a SWMM input text file repeatedly with modification of the text file based on the last SWMM output and the next trial parameters selected by R optimization code.  The prerequisites for using the code are having a SWMM input file, calibration data, willingness to read code (it's not that bad :)), and willingness to check the output closely as the code is experimental.  These scripts currently only do autocalibration.</p>
<h6>I am working on a similar, separate and larger programming effort (in parallelized Python not R). RSWMM is only a proof of concept. The new effort extends these ideas to a more flexible interface and tiered design optimization.  If you are interested in any of that, or have any feedback on RSWMM, please let me know (see below) </h6>
<h4>Summary of How to Use this Code Efficiently</h4>
<ol>
<li>Get a SWMM .inp input text file</li>
<li>Read below (a cut and paste of runRSWMM.r) and note each comment's warnings</li>
<li>Choose multi- or single objective</li>
<li>Get your calibration time series CSV in the right format, or use an argument to accomodate your format</li>
<li>Run it after making sure you have disk space or you have edited the code to programmatically delete files</li>
</ol>
<h4>runSWMM.r is quoted below.  It is just a lot of help comments around R function calls that use SWMM binary file reader as part of autocalibration.</h4>
<pre><code>
#runRSWMM Developed by Peter Steinberg
#Version 1: December 2011
#Revision 1.1: January 1/10/2012, corrected problem in binary file reader

#General Notes
#Before editing this script do the following things:
#Move your SWMM file to a directory that can hold a lot of files
#Test that you can run your SWMM file from this directory and you haven't messed up paths to files or something
#Take your SWMM input file and replace the uncertain parameters with codes like
#   $1$, $2$,  $3$
#You can repeat codes if you want the optimization algorithm to repeat a parameter.
#For example, if you know 2 subcatchments should have the same infiltration rate, you
# can put the same code in for their infiltration rates and they will receive the same parameter
#Create a parameter bounds CSV file that looks like this (without the comment sign #):
#                 Code,Minimum,Maximum,Initial
#                 $1$,10,32,15
#                 $2$,10,31,20
#                 $3$,4,15,5
#                 $4$,2,8,7
#                 $5$,25,100,33
#                 $6$,20,75,33
#                 $7$,20,60,50

#Create a calibration time series data CSV that looks like this (without the comment sign #):
#Date      ,(CFS)
#1/1/07 0:01,0.08
#1/1/07 0:02,0.22
#1/1/07 0:03,0.38
#1/1/07 0:04,0.54
#1/1/07 0:05,0.67
#1/1/07 0:06,0.83

#REMEMBER YOU HAVE TO USE DOUBLE BACKSLASHES FOR ALL FILENAMES##########
#You have to manually create all directories you provide.  RSWMM does not make directories.

#Preliminaries: clear workspace and source the RSWMM code for a function library
rm(list=ls())
#edit this source line to reflect where you have saved RSWMM.r.
source("O:\\departments\\Water Quality\\Users\\Peter\\programs\\RSWMM\\RSWMM.r")

#If you are doing a calibration run, you need to provide the following lines
# to direct the optimizer to your files
#Calibration Data should be in a CSV with datetimes in the first column,
#and data in the second column
#The text file is assumed to have a one line header
#Call this function with the correct dateFormat for your datetimes
#the dateFormat is passed to strptime, so look for formatting information there
#for example, dates like this 1/1/07 12:00, can be read with the default dateFormat
#e.g.:
#Date      ,(CFS)
#1/1/07 0:01,0.08
#1/1/07 0:02,0.22
#1/1/07 0:03,0.38
#1/1/07 0:04,0.54
#1/1/07 0:05,0.67
#1/1/07 0:06,0.83
#
calDataCSV="O:\\departments\\Water Quality\\Users\\Peter\\programs\\RSWMM\\testingData\\CalData1.csv"

</code></pre>
*Read the rest of <a href="https://github.com/PeterDSteinberg/RSWMM/edit/master/runRSWMM.r">runRSWMM.r</a>
