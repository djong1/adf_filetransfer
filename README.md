# Optimized ADF file transfer pipelines
Many times Azure Data Factory is used to transfer files (large or small) from A to B. Since most of the times these files are a crucial for system integration, reliability and recoverability are crucial. Many times I see peopel using "copy" activity only without some smart handling and retry. This can result in high costs and unreliable file transfers. 

## Best practices

- Use Lastmodified to make sue that the file you copy is not in use (reliability)
- Use Lastmodified alligned with you schedule to prevent same files to be copied multiple times (reliabilty / costs / performance)
- Use a trigger based schedule, so only trigger the pipeline when a file is published / present (costs)
- Use filters in time based triggers (costs)

### Lastmodified

to be done

