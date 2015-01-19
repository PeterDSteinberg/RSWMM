# RSWMM
<h3>Autocalibration for EPA Stormwater Management Model (SWMM) version 5 using multi- or single objective optimization in R.
</h3><h5>
You're fully responsible for the any problems related to this software - I'm not maintaining it because I am working on a similar, separate and larger programming effort (in parallelized Python not R). It was only a proof of concept.
</h5>
<pre>
Version 1: December 2011
Revision 1.1: January 1/10/2012, corrected problem in binary file reader
General Notes
Before editing this script do the following things:
Move your SWMM file to a directory that can hold a lot of files
Test that you can run your SWMM file from this directory and you haven't messed up paths to files or something
Take your SWMM input file and replace the uncertain parameters with codes like<code><pre>
   $1$, $2$,  $3$
   </pre></code>
You can repeat codes if you want the optimization algorithm to repeat a parameter.
For example, if you know 2 subcatchments should have the same infiltration rate, you
 can put the same code in for their infiltration rates and they will receive the same parameter
Create a parameter bounds CSV file that looks like this (without the comment sign ):
</pre>
<code><pre>
                 Code,Minimum,Maximum,Initial
                 $1$,10,32,15
                 $2$,10,31,20
                 $3$,4,15,5
                 $4$,2,8,7
                 $5$,25,100,33
                 $6$,20,75,33
                 $7$,20,60,50
</pre></code>
Create a calibration time series data CSV that looks like this (without the comment sign ):
<code><pre>
Date      ,(CFS)
1/1/07 0:01,0.08
1/1/07 0:02,0.22
1/1/07 0:03,0.38
1/1/07 0:04,0.54
1/1/07 0:05,0.67
1/1/07 0:06,0.83
</pre></code>
<pre>
REMEMBER YOU HAVE TO USE DOUBLE BACKSLASHES FOR ALL FILENAMES
You have to manually create all directories you provide.  RSWMM does not make directories.
Preliminaries: clear workspace and source the RSWMM code for a function library
edit this source line to reflect where you have saved RSWMM.r.
If you are doing a calibration run, you need to provide the following lines
 to direct the optimizer to your files
Calibration Data should be in a CSV with datetimes in the first column,
and data in the second column
The text file is assumed to have a one line header
Call this function with the correct dateFormat for your datetimes
the dateFormat is passed to strptime, so look for formatting information there
for example, dates like this 1/1/07 12:00, can be read with the default dateFormat
e.g.:
</pre>
<code><pre>
Date      ,(CFS)
1/1/07 0:01,0.08
1/1/07 0:02,0.22
1/1/07 0:03,0.38
1/1/07 0:04,0.54
1/1/07 0:05,0.67
1/1/07 0:06,0.83
</pre></code>
<pre>
if you have a non-stadard date format, you can provide that as an argument below, but in either case
 you have to call the function that reads the calData
getCalDataFromCSV(CSVFile=calDataCSV,dateFormat="%m/%d/%y %H:%M")
Provide a path for the CSV containing optimization history.  This is an empty file to start out.
Make sure you have created the directories that will hold this file
Provide a path for the CSV containing parameter bounds
For ease, make your parameter bounds file in this format (without the comment symbols):
</pre>
<code><pre>
                 Code,Minimum,Maximum,Initial
                 $1$,10,32,15
                 $2$,10,31,20
                 $3$,4,15,5
                 $4$,2,8,7
                 $5$,25,100,33
                 $6$,20,75,33
                 $7$,20,60,50
</pre></code>
<pre>
Initialize the iteration count and the optimization history, in case you
want to stop the model before the optimization function is complete. If you
 press the STOP button before the optimzation function returns, you can check your
csv provided above or the variable optimizationHistory for intermediate results
Select single or multiobjective optimization by setting one of the two following variables to TRUE
Set useOptim to TRUE if you are doing single objective optimization, otherwise FALSE
set useMCO to TRUE if you are doing multiobjective optimization, otherwise FALSE

Single Objective Calibration Begins Here

If you are doing multi-objective optimization, this section is ignored
Provide options for single objective optimization
Initialize the options object
Pick one of six methods for calibration:
 may be one of  = c("Nelder-Mead", "BFGS", "CG", "L-BFGS-B", "SANN", "Brent"),
Look at the documentation in RSWMM.R for the getSWMMTimeSeriesData function
to develop a function call that returns your time series
 of interest from the SWMM binary file
 a few notes on this funtion:
you always have to pass in headobj=headobj.  This is so that you don't reread the header on every iteration
you select an iType, which determines whether you are looking for a node, link, subcatchment, or system variable
you provide a vIndex, which is the parameter you want to return (e.g. depth)
you provide the nameInOutputFile.  This will be the same as the node, link, or subcatchment number in the input file.
if you are getting a system variable's results, you can leave nameInOutputFile as ""
you should provide a function call in quotes that will subsequently be evaled in the objective function
Put the parameter bounds and intialization in the optimization options: you shouldn't need to change these
 lines if you have imported them using RSWMM formats/functions
Provide a base name for the input/output files that are created
RSWMM will add the necessary extensions.  It also adds random text so that it is thread safe, and
 you can run more than one RSWMM.r optimization at the same time
Provide a SWMM template file that has the replacement codes
Provide a path to SWMM.exe.  The binary file reader is written for SWMM 5.0.022.  For earlier versions of SWMM,
 you would have to edit the binary file reader because the output format has changed.
Look at RSWMM.R's performanceStatsAsMinimization function and select one of the performance statistics
The following function call does the optimization.  It should not need any edits,
unless you want to look at the optim function documentation and provide more specific
 control parameters.
set the working directory in R to be the directory of the swmm template so that LID reports 
show up in the right place this is a 1/24/2012 edit

End of Single Objective Calibration Section


 Start of Multiobjective Optimization 

If you are doing single objective optimization by setting useOptim to TRUE, this
section is ignored.
initialize the multiobjective optimization options object
Look at the documentation in RSWMM.R for the getSWMMTimeSeriesData function
to develop a function call that returns your time series
 of interest from the SWMM binary file
 a few notes on this funtion:
you always have to pass in headobj=headobj.  This is so that you don't reread the header on every iteration
you select an iType, which determines whether you are looking for a node, link, subcatchment, or system variable
you provide a vIndex, which is the parameter you want to return (e.g. depth)
you provide the nameInOutputFile.  This will be the same as the node, link, or subcatchment number in the input file.
if you are getting a system variable's results, you can leave nameInOutputFile as ""
you should provide a function call in quotes that will subsequently be evaled in the objective function
Provide upper and lower bounds.  No need to edit these lines if you have imported parameters using RSWMM functions/formats
Provide path to SWMM.exe
Select one or more performance stats listed in RSWMM.R
Provide a base output name for SWMM input/output files
Provide a path to your SWMM template file with replacement codes in it
set the working directory in R to be the directory of the swmm template so that LID reports 
show up in the right place this is a 1/24/2012 edit
This is the function call to NSGA2.  Change this if
 you want to further control the process.  (See the documentation for mco package: NSGS2 function)

END of multiobjective optimization

<pre>
