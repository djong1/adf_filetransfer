# Optimized ADF file copy pipelines
Many times Azure Data Factory is used to transfer files (large or small) from A to B. Since most of the times these files are a crucial for system integration, reliability and recoverability are crucial. Many times I see peopel using "copy" activity only without some smart handling and retry. This can result in high costs and unreliable file transfers. 

## Best practices

- Use Lastmodified to make sue that the file you copy is not in use (reliability)
- Use Lastmodified alligned with you schedule to prevent same files to be copied multiple times (reliabilty / costs / performance)
- Use a trigger based schedule, so only trigger the pipeline when a file is published / present (costs)
- Use filters in time based triggers (costs)
- Delete the file in the source after copy is completed in seperate activity 

### Lastmodified

You can set the lasmodified in the settings of an activity. For instance if you want to get a list of files in a location you can use GET METADATA and in the settings parameter you can set the lastmodified configuration. If your schedule run's every 10 minutes you only want the files that are placed in the previous 10 minutes, you can do this by: 
- Start time (UTC): @getPastTime(11,'minute')
- End Time (UTC): @getPastTime(1,'minute')

![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/Lastmodified.png)

### Filter Files

Do not schedule a copy pipeline every xx minutes, instead run the copy pipeline only when a file actualy exists in the source. The reasson is that scheduling a copy pipeline frequently can be extremely costly. You can achive this by using a trigger, for instance a storage trigger or http trigger. 
When you are not able to use an external trigger do the following: 

- Use a GET METADATA activity to get a list of files that need to be copied
- Use a FOR EACH to copy these files from source to target

The benfit is that the "expensive" activity COPY is only executed when the file exists. This will save you costs. For example running a simple copy activity every 10 minutes for 350 files will cost you $12k per month, using the above approach will cost you $123,-.

![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/getfilelist.png)
![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/filterfiles.png)
- @activity('FilterFilesOnly').output.value (use the output value from the FiltersFileOnly activity)
![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/foreachfile.png)
- activity's are executed in paralel with MAX 20 jobs in paralel

### Copy and delete files

In the foreach activity we need to copy the file from source tor target and delete it in the destination when completed:

- ![Image Alt Text](https://gp3scdnstorage.blob.core.windows.net/private/copyonefile.png)
- @{item().name} is the name of the file that is parsed into the foreach
- @pipeline().parameters.SourceStore_Directory the source path is a pipeline parameter, if this is not a fixed path use a getmetadata to get the filepath
-

