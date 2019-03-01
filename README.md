# DataSciencePracticum-2
In this module, we will investigate the original raw data for the problems discussed in week 1. The ultimate goal, which will take multiple weeks to accomplish, is to build a comprehensive data analysis “dashboard” for this data set. This module is part of the process toward this goal.

The main task in this module is to clean up the raw data and transform it into the form that you saw in week 1. The input is the raw data (download link below). The output should be the same as, or similar to, the HDF5 data file given in Week 1.

You are suggested to go through the following steps. You may have to fill in some details, or refresh your memory how to do certain tasks by additional readings from books or internet resources such as https://pandas.pydata.org/pandas-docs/stable/index.html.

I suggest using Pandas (Python) for this, but you can use other tools of your choice.

Step 1
Download the raw data file (text file, approximately 1.7GB):

data file

Step 2
Read the raw data file in the Python.

This data file has about 4.5 million rows. If your computer is not very fast, you may take the first 5000 lines from the file to test your newly written code. After your code is working, run it against the complete data file to get the final results.

The raw data file has 45 columns. The column definitions, in the order of their appearance per row in the data file, are:

fields = [ 'qname', 'hostname', 'group', 'owner', 'job_name',
      'job_number', 'account', 'priority', 'submission_time', 'start_time',
      'end_time', 'failed', 'exit_status', 'ru_wallclock', 'ru_utime',
      'ru_stime', 'ru_maxrss', 'ru_ixrss', 'ru_ismrss', 'ru_idrss',
      'ru_isrss', 'ru_minflt', 'ru_majflt', 'ru_nswap', 'ru_inblock',
      'ru_oublock', 'ru_msgsnd', 'ru_msgrcv', 'ru_nsignals', 'ru_nvcsw',
      'ru_nivcsw', 'project', 'department', 'granted_pe', 'slots',
      'task_number', 'cpu', 'mem', 'io', 'category',
      'iow', 'pe_taskid', 'maxvmem', 'arid', 'ar_submission_time' ]
In Pandas, the command to read the file can be:

pd.read_csv(datafile, sep=':',error_bad_lines=False, header=None,
               names=fields, usecols=usecols, encoding = "ISO-8859-1")
where datafile is the file name of the data set.

Step 3
We will keep only the following columns, discarding the rest:

usecols = ['owner', 'group', 'job_number', 'task_number','granted_pe',
        'slots' ,'category','start_time', 'end_time','failed', 'exit_status']
Step 4
Step 4-1
A typical category field looks like:

...:-U campus -u stevendu -l h_data=4G,h_rt=86400,h_vmem=4G -pe single 1:...
In the category field, extract the h_data data, covert the value to gigabytes (see below for explanation) and make it a new column.

If the value of h_data ends with a “G” or “g”, the data is in the unit of “gigabytes”. If the value ends with “m” or “M”, the data is in the unit of megabytes:

20M or 20m  : 20 megabytes
4G or 4g    : 4 gigabytes
1024        : 1024 bytes
For example, if the category field has h_data=2048M,h_rt=86400,exclusive=TRUE, extract the 2048M, and convert it to 2048 / 1024 = 2 (gigabytes). (Recall: 1G = 1024M).

Step 4-2
In the category field, extract the h_rt data, which is in seconds. Make a new column for h_rt in the unit of hours. For example, if the h_rt value is 86400, convert it to 86400/(3600*24) = 24 (hours). In this case, the row value in the new h_rt column will be 24.

Step 4-3
Create a new column called highp. In the category field, if highp=TRUE or highp=true is identified, the row value of highp would be 1. Otherwise highp is 0 in the new column.

Step 4-4
Create a new column called exclusive. In the category field, if exclusive=TRUE or exclusive=true is identified, the row value of exclusive would be 1. Otherwise exclusive is 0 in the new column.

Step 4-5
Create a new column called h_vmem. Look for its value in the category field. Similar to h_data, convert the values to gigabytes.

Step 4-6
Create a new column called gpu. Look for the value in the category field. If required_gpu is identified, set the row value to 1. Otherwise the row value is 0.

Step 4-7
Create two new columns. One is named pe. Another one is slot. In the category field, look for the -pe data, e.g. -pe single 1. In this case, put the single (string) in the new pe column, and the value 1 (integer) in the new slot column.

If no pe data is found in the category field, enter none for the pe column, and 1 for the slot column.

Step 4-8
Create a new column campus. In the category field, if the value following -U is campus, set the value to 1 (integer). Otherwise, set it to 0.

Step 5
The raw data in the start_time, end_time and submission_time are the UNIX epoch time. Convert the data strings to a Pandas (or Python) data objects. The Timestamp function in Pandas can do this easily. See its documentation for details.

Step 6
Create a new column wait_time whose value is the difference of start_time - submission_time.

Create a new column wtime (short for “wall-clock time”) whose value is the difference of end_time - start_time.

Step 7
Identify duplicates and merge them. You can use the Pandas drop_duplicates function.

A “job” can be uniquely identified by the combination of “job_number”, “task_number” and “submission_time”. However, there are duplicated lines for some (job_number, task_number, submission_time) pairs. Identify them and remove the duplicates. For example, if you have:

job_number task_number submission_time      end_time ...
10            1          2018-10-03          ...
10            1          2018-10-03          ...
10            1          2018-10-03          ...
13            1          2018-10-04          ...
The two duplicated lines with (jobnumber, task)=(10,1) should be merged. After the merge, you should have something like:

job_number task_number submission_time      end_time ...
10            1        2018-10-03          ...
13            1        2018-10-04          ...
Step 8
You may or may not find some values in submission_time, start_time or end_time contain dates before year 2018. Drop (delete) these rows from the data.

Step 9
Remove these columns that we will not need: ['category', 'qname', 'job_name', 'account', 'project']

Step 10
If you have reached this step, congratulations, you now have a clean data set ready for analysis. The data set should have 23 columns:

['group', 'owner', 'job_number', 'submission_time', 'start_time',
   'end_time', 'failed', 'exit_status', 'granted_pe', 'slots',
   'task_number', 'maxvmem', 'h_data', 'h_rt', 'highp', 'exclusive',
   'h_vmem', 'gpu', 'pe', 'slot', 'wait_time', 'wtime', 'campus']
Save the data in HDF5 format. The Pandas to_hdf function can do this easily.

Step 11
Compare the HDF5 file you saved and the one from Week 1. Discuss your observations and their differences. If there are differences, how would you make yours more closer (if not identical) to Week 1’s HDF5 file. Or if you think yours is more correct or better, justify your version.
