# Interpreting SMF 115 Statistics
## Adapter and Dispatcher Task data from SMF-QCTADP.csv and SMF-QCTDSP.csv

### Dispatcher Tasks
Channels connecting to a queue manager are assigned a dispatcher task upon connection. 10 channels are assigned to a dispatcher task at a time, then the subsequent 10 channels will be assigned to the next dispatcher task available. Because of this assignment technique, it is possible for activity to get skewed towards just a couple of dispatcher tasks. This impacts performance. Here, I will show you a couple of different ways you may notice your dispatcher tasks behaving. This CSV information is created through running MQSMFCSV against SMF 115 data. 
*Disclaimer: all of the following data is made up, so numbers are not exact, but rather meant to show patterns

There are four common scenarios you may notice in your dispatcher task data. 
1) **Heavily skewed distribution** where there is a lot of activity happening in just a few dispatcher tasks and none in the others. This is not ideal because it means just a few dispatcher tasks are doing all the work, despite having additional dispatcher tasks already allocated in your environment. 
![image](https://media.github.ibm.com/user/236584/files/1eb2fd55-7485-4435-96cf-031413587975)
2) **Moderately skewed distribution** where there is a moderate amount of work happening in a few dispatcher tasks and none in the others. This indicates that the queue manager is not that busy. Performance-wise, this is all well and good, but you may want to consider reducing your quantity of dispatcher tasks for the given queue manager
![image](https://media.github.ibm.com/user/236584/files/e4e4afcf-50e7-4bb9-8b36-e36b082e4e73)
3) **Evenly spread distribution** where channels have been quite evenly spread across the available dispatcher tasks, and none of the request counts are exordinarily high. This is a good, healthy situation for a queue manager to be in, but if there are spikes in workload anticipated, adding new dispatcher tasks might be worth considering. 
![image](https://media.github.ibm.com/user/236584/files/341cb886-6820-41b8-97ac-d935027b12ca)
4) **Evenly crowded distribution** where channels are spread evenly across the available dispatcher tasks, but the volumes of requests are quite high. Here, the dispatcher tasks have been well-spread, but there is too much activity potentially for the 10 dispatcher tasks to handle optimally. This would be an example of when to definitely allocate more dispatcher tasks.
![image](https://media.github.ibm.com/user/236584/files/01426360-3246-4fa6-8b20-41fe38d08481)

## Adapter Tasks
Adapter tasks are more straight forward in terms of how they are allocated than dispatcher tasks. Each command that an incoming channel connection might ask of a queue manager is allocated to an available adapter task, one at a time. 
1) **Heavily used** adapter tasks. If the adapter tasks look crowded, consider adding more adapter tasks. Adding additional adapter tasks can improve performance and does not add additional overhead, so seeing some lightly-used extra adapter tasks can provide cushioning in the event of performance peaks. 
![image](https://media.github.ibm.com/user/236584/files/45e2c77a-344c-47b9-ba1a-7b67341c2a52)

2) **Lightly used** adapter tasks. There is plenty of room for processing here. Data matching this pattern is usually associated with a lighter used queue manager.
![image](https://media.github.ibm.com/user/236584/files/1c55efa3-d174-42a6-bdd2-94f148a0155a)

## Python Script for analyzing SMF data
This script was invented for the purpose of scanning through the adapter, dispatcher, and queue summary SMF data for a very large environment. It points out queue managers where there might be further investigation required. This is not intended to be a holistic evaluation, but rather to aid with the evaluation particularly for large environments.

The script operates on some pattern recognition you will have noticed in the above sample data. For example, if there are a lot of zero's or low digits in the request count for the data, the script will denote that as not being particular busy. On the other end, when dispatcher tasks are particularly high, the script will point out that the distribution might be skewed in that queue manager. 
