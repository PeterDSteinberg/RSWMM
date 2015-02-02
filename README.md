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

#if you have a non-stadard date format, you can provide that as an argument below, but in either case
# you have to call the function that reads the calData
#getCalDataFromCSV(CSVFile=calDataCSV,dateFormat="%m/%d/%y %H:%M")
calDataObj<-getCalDataFromCSV(CSVFile=calDataCSV)

Provide a path for the CSV containing optimization history.  This is an empty file to start out.
#Make sure you have created the directories that will hold this file
optFile="O:\\departments\\Water Quality\\Users\\Peter\\programs\\RSWMM\\testingData\\Optimization History.csv"

#Provide a path for the CSV containing parameter bounds
#For ease, make your parameter bounds file in this format (without the comment symbols):
#                 Code,Minimum,Maximum,Initial
#                 $1$,10,32,15
#                 $2$,10,31,20
#                 $3$,4,15,5
#                 $4$,2,8,7
#                 $5$,25,100,33
#                 $6$,20,75,33
#                 $7$,20,60,50
parFile="O:\\departments\\Water Quality\\Users\\Peter\\programs\\RSWMM\\testingData\\Parameter Ranges 1.csv"
parametersTable=getParmeterBounds(parFile)


#Initialize the iteration count and the optimization history, in case you
#want to stop the model before the optimization function is complete. If you
# press the STOP button before the optimzation function returns, you can check your
#csv provided above or the variable optimizationHistory for intermediate results

iteration=1
optimizationHistory=data.frame()


#Select single or multiobjective optimization by setting one of the two following variables to TRUE
#Set useOptim to TRUE if you are doing single objective optimization, otherwise FALSE
useOptim=FALSE
#set useMCO to TRUE if you are doing multiobjective optimization, otherwise FALSE
useMCO=TRUE


##################################################################################
#######################Single Objective Calibration Begins Here##################
#####################################################################################
#If you are doing multi-objective optimization, this section is ignored


#Provide options for single objective optimization
#Initialize the options object
optimOpt={}

#Pick one of six methods for calibration:
# may be one of  = c("Nelder-Mead", "BFGS", "CG", "L-BFGS-B", "SANN", "Brent"),
optimOpt$method ="SANN"
#Look at the documentation in RSWMM.R for the getSWMMTimeSeriesData function
#to develop a function call that returns your time series
# of interest from the SWMM binary file
# a few notes on this funtion:
#you always have to pass in headobj=headobj.  This is so that you don't reread the header on every iteration
#you select an iType, which determines whether you are looking for a node, link, subcatchment, or system variable
#you provide a vIndex, which is the parameter you want to return (e.g. depth)
#you provide the nameInOutputFile.  This will be the same as the node, link, or subcatchment number in the input file.
#if you are getting a system variable's results, you can leave nameInOutputFile as ""
#you should provide a function call in quotes that will subsequently be evaled in the objective function
optimOpt$functionCallToEvalForASWMMTimeSeries='getSWMMTimeSeriesData(headObj=headObj,iType=3,nameInOutputFile="",vIndex=4)'

#Put the parameter bounds and intialization in the optimization options: you shouldn't need to change these
# lines if you have imported them using RSWMM formats/functions
optimOpt$lower= c(as.vector(parametersTable["Minimum"]))$Minimum
optimOpt$upper=c(as.vector(parametersTable["Maximum"]))$Maximum
optimOpt$initial=c(as.vector(parametersTable["Initial"]))$Initial
#Provide a base name for the input/output files that are created
#RSWMM will add the necessary extensions.  It also adds random text so that it is thread safe, and
# you can run more than one RSWMM.r optimization at the same time
optimOpt$baseOutputName="O:\\departments\\Water Quality\\Users\\Peter\\programs\\RSWMM\\testingData\\OptTest1\\Opt Ex1 Post"
#Provide a SWMM template file that has the replacement codes
optimOpt$SWMMTemplateFile="O:\\departments\\Water Quality\\Users\\Peter\\programs\\RSWMM\\testingData\\Example1-Post.inp"
#Provide a path to SWMM.exe.  The binary file reader is written for SWMM 5.0.022.  For earlier versions of SWMM,
# you would have to edit the binary file reader because the output format has changed.
optimOpt$SWMMexe="C:\\Program Files (x86)\\EPA SWMM 5.0\\SWMM5.exe"
#Look at RSWMM.R's performanceStatsAsMinimization function and select one of the performance statistics
optimOpt$performanceStat="sumOfSquaredError"

#The following function call does the optimization.  It should not need any edits,
#unless you want to look at the optim function documentation and provide more specific
# control parameters.

#set the working directory in R to be the directory of the swmm template so that LID reports 
#show up in the right place #this is a 1/24/2012 edit
setwd(dirname(optimOpt$SWMMTemplateFile))

if(useOptim){
  out=optim(optimOpt$initial,objectiveFunction,
  gr = NULL, baseOutputName= optimOpt$baseOutputName,
     SWMMTemplateFile=optimOpt$SWMMTemplateFile,
     SWMMexe=optimOpt$SWMMexe,
       functionCallToEvalForASWMMTimeSeries=optimOpt$functionCallToEvalForASWMMTimeSeries,
     performanceStat=optimOpt$performanceStat,
      method = optimOpt$method,
      lower = optimOpt$lower, upper = optimOpt$upper,
      control = list(), hessian = FALSE)
}

#################################################################################
#########################End of Single Objective Calibration Section#############
#################################################################################






##################################################################################
######################### Start of Multiobjective Optimization ##################
#################################################################################
#If you are doing single objective optimization by setting useOptim to TRUE, this
#section is ignored.





#initialize the multiobjective optimization options object
mcoOpt={}
#Look at the documentation in RSWMM.R for the getSWMMTimeSeriesData function
#to develop a function call that returns your time series
# of interest from the SWMM binary file
# a few notes on this funtion:
#you always have to pass in headobj=headobj.  This is so that you don't reread the header on every iteration
#you select an iType, which determines whether you are looking for a node, link, subcatchment, or system variable
#you provide a vIndex, which is the parameter you want to return (e.g. depth)
#you provide the nameInOutputFile.  This will be the same as the node, link, or subcatchment number in the input file.
#if you are getting a system variable's results, you can leave nameInOutputFile as ""
#you should provide a function call in quotes that will subsequently be evaled in the objective function
mcoOpt$functionCallToEvalForASWMMTimeSeries='getSWMMTimeSeriesData(headObj=headObj,iType=3,nameInOutputFile="",vIndex=4)'
#Provide upper and lower bounds.  No need to edit these lines if you have imported parameters using RSWMM functions/formats
mcoOpt$lower=c(as.vector(parametersTable["Minimum"]))$Minimum
mcoOpt$upper=c(as.vector(parametersTable["Maximum"]))$Maximum
#Provide path to SWMM.exe
mcoOpt$SWMMexe="C:\\Program Files (x86)\\EPA SWMM 5.0\\SWMM5.exe"
#Select one or more performance stats listed in RSWMM.R
mcoOpt$performanceStats=c("sumOfSquaredError","linearCorrelationTimesMinus1")
#Provide a base output name for SWMM input/output files
mcoOpt$baseOutputName="O:\\departments\\Water Quality\\Users\\Peter\\programs\\RSWMM\\testingData\\OptTest1\\Opt Ex1 Post"
#Provide a path to your SWMM template file with replacement codes in it
mcoOpt$SWMMTemplateFile="O:\\departments\\Water Quality\\Users\\Peter\\programs\\RSWMM\\testingData\\Example1-Post.inp"
#set the working directory in R to be the directory of the swmm template so that LID reports 
#show up in the right place #this is a 1/24/2012 edit
setwd(dirname(mcoOpt$SWMMTemplateFile))

#This is the function call to NSGA2.  Change this if
# you want to further control the process.  (See the documentation for mco package: NSGS2 function)
if(useMCO){
  library(mco)
  out= nsga2(objectiveFunction,
            idim=length(mcoOpt$lower),
            odim=length(mcoOpt$performanceStats),
            baseOutputName= mcoOpt$baseOutputName,
            SWMMTemplateFile=mcoOpt$SWMMTemplateFile,
            SWMMexe=mcoOpt$SWMMexe,
            functionCallToEvalForASWMMTimeSeries=mcoOpt$functionCallToEvalForASWMMTimeSeries,
            performanceStat=mcoOpt$performanceStats,
            generations=500,
            lower.bounds=as.real(mcoOpt$lower),
            upper.bounds=as.real(mcoOpt$upper),
            constraints=NULL)
}

##################################################################################
########################END of multiobjective optimization########################
##################################################################################
</code></pre>
