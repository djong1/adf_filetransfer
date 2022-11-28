# Optimized ADF file transfer pipelines
Many times Azure Data Factory is used to transfer files (large or small) from A to B. Since most of the times these files are a crucial for system integration, reliability and recoverability are crucial. Many times I see peopel using "copy" activity only without some smart handling and retry. This can result in high costs and unreliable file transfers. 

## Best practices

- Use Lastmodified to make sue that the file you copy is not in use (reliability)
- Use Lastmodified alligned with you schedule to prevent same files to be copied multiple times (reliabilty / costs / performance)
- Use a trigger based schedule, so only trigger the pipeline when a file is published / present (costs)
- Use filters in time based triggers (costs)

### Lastmodified

You can set the lasmodified in the settings of an activity. For instance if you want to get a list of files in a location you can use GET METADATA and in the settings parameter you can set the lastmodified configuration. If your schedule run's every 10 minutes you only want the files that are placed in the previous 10 minutes, you can do this by: 
- Start time (UTC): @getPastTime(11,'minute')
- End Time (UTC): @getPastTime(1,'minute')

![Image Alt Text]((https://gp3scdnstorage.blob.core.windows.net/private/Lastmodified.png))
