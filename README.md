# Optimized ADF file copy pipelines
Usually Azure Data Factory (ADF)is used to transfer files (large or small) from A to B. Since most of the times these files are a crucial for system integration, reliability and recoverability are crucial. 

### Trigger or schedule
The most ideal approach is to trigger an ADF pipeline when a file is placed at a certain location, for instance using Azure Blob Storage trigger. In some cases, you might not have the ability to use triggers, for instance with Azure Files or on-premises storage platforms, in those situations you need to schedule your copy pipeline on an interval. Scheduling brings some extra complexity if you need reliable and predictable behavior. 
In order to have the latest data at the Destination Location B, ADF is used to periodically check new files on Source location A using a schedule. For the purpose of (cost) efficiency it is important to check on new files first before starting the transfer activity. Starting the transfer activity before checking on new files has two potential drawbacks. When you start a transfer without having new files results in a cost of an integration run time for each file in your list. So the more frequent ADF checks on many files, the more unnecessary integration run times are started without creating value. The other drawback in this scenario is about large files that cannot be transferred within a ADF check interval set. In this case, a new integration run time is started next to the existing one trying to copy the same file. This slows down the copy activity and might result in many other unnecessary integration times creating unreliability and high cost.

In order to optimise your pipelines it is a best practise to first verify on new files that are not use (by an already started integration run time).This verification can be done with one single integration run time. Based on the result only integration run times will be started for the new files that are not in use. So when applying this optimisation for a larger the number of files you want to keep up date in the Destination Location B, the higher the reliability and the lower the total cost. Below explains the details how to configure optimised ADF pipelines to benefit from reliability and cost.

## Best practices if you cannot use a triggered pipeline

- Use Lastmodified to make sure that the file you copy is not in use (reliability)
- Use Lastmodified alligned with your schedule to prevent same files to be copied multiple times (reliabilty / costs / performance)
- Use a trigger based schedule, so only trigger the pipeline when a file is published / present (costs)
- Use filters in time based triggers (costs)
- Delete the file in the source after copy is completed in seperate activity
- Use parameters to reduce the number of pipelines needed 

# Copy pipeline for a single directorie / container.
In this section I'll explain how you can create a smart copy pipeline to pick-up files from one directorie / container when the exist, if there is no file the pipeline stops executing and it will save you costs.

### Lastmodified.

You can set the lasmodified in the settings of an activity. For instance if you want to get a list of files in the Source Location A, you can use GET METADATA and in the settings parameter you can set the lastmodified configuration. If your schedule runs every 10 minutes you only want the files that are placed in the previous 10 minutes, you can do this by: 
- Start time (UTC): @getPastTime(11,'minute')
- End Time (UTC): @getPastTime(1,'minute')

![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/Lastmodified1.png)

### Filter Files.

Do not schedule a copy pipeline every xx minutes, instead run the copy pipeline only when a file actualy exists in the Source Location A. The reasson is that scheduling a copy pipeline frequently can be extremely costly. You can achieve this by using a trigger, for instance a storage trigger or http trigger. 
When you are not able to use an external trigger do the following: 

- Use a GET METADATA activity plus a FILTER activity to get a list of files that need to be copied
- Use a FOR EACH to copy these files from source to target

The benfit is that the "expensive" activity COPY is only executed when the file exists. This will save you costs. For example running a simple copy activity every 10 minutes for 350 files will cost you $12K per month, using the above approach will cost you $123,-.

The GET METADATA will lookup all files and folders in a given path that are posted in the last 11 minutes (-1), the FILTER activity will filter on files.  

  ![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/getfilelist.png)
  ![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/filterfiles1.png)
- @activity('FilterFilesOnly').output.value (use the output value from the FiltersFileOnly activity)
  ![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/foreachfile1.png)
- activity's are executed in paralel with MAX 20 jobs in paralel

### Copy and delete files.

In the foreach activity we need to copy the file from source tor target and delete it in the destination when completed:

  ![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/copyonefile1.png)
  ![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/delete1.png)
- @{item().name} is the name of the file that is parsed into the foreach
- @pipeline().parameters.SourceStore_Directory the source path is a pipeline parameter, if this is not a fixed path use a getmetadata to get the filepath

### Parameters.

In the above examples you can see that I have used parameters:
- On the Pipeline level, to set the SOURCE directory to scan and to set the TARGRet directory to copy files in
- On the activity level referencing to the outcome of previous steps
- On the dataset level to reference to path and files

You can go wild with parameters and make a pipeline, activities and datasets completely parametrized and dynamic. This will reduce the number of predefined pipelines and datasets however it can make it complex to master. 

# Copy pipeline for files from multiple directories.

In the previous example I explained how you can copy multiple files from a single folder or container. In this example I will show how you can copy files from multiple directories or containers. Below you see the complete pipeline:
![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/pipelinemultiplefolders.png)
- first step is to get a list of directories
- second step is to filter out the directories
- do a FOR EACH for every directory found
- get the number of files and file names in the directorie
- IF directories with one or multiple files a copy task is executed

Note: you can optimize this pipeline by adding a LASTMODIFIED parameter to copy files only one time, if the schedule is shorter than the pipeline run, typically when exchanging large files.

**see code sample in repo**








