# meetopics
Sample Apache Beam 2.5 (January 2018) data engineering project to experiment with unit testing and both bounded and unbounded pipelines.

# Development
The pipeline DAG is made of the following steps:
1) Data is loaded as strings from meetup.json or ws://stream.meetup.com/2/rsvps websocket
2) JSON is parsed and loaded in a pojo class. Malformed record are redirected and stored. Event.time is added as Timestamp to each record to take advantage of the Beam windowing framework (simple FixedWindow pattern is used but can be easily modified).
3) Basic deduplication by RSVPid is added to avoid over-counting RSVPs that went through multiple edits in a single time window.
4) Relevant fields (event.eventTime and group.groupName) are splitted in single words and n-grams (groups of adjacent words) with basic stop-words removal. GroupTopics are added as-is with a prefix to avoid mixing with words or n-grams from other fields.
5) N-gram frequency is computed applying a GroupByKey transformation on the ngram PCollection and dividing the word count by the total count of words within the current window. Basic filtering is applied to discard rare words.
6) Slope is computed joining the window word frequency collection with baseline frequencies computed at the beginning of the pipeline and dividing the former by the latter. The baseline is computed from the whole meetup.json file without any windowing applied.
7) Top N n-grams by descending slope are extracted and written to a file named after the starting and ending time of the relevant window.

# Executing
command to run the solution
(Windows: replace ```run.sh``` with ```run.ps1``` after setting ```set-executionpolicy Unrestricted```):

```
# Build solution
mvn package -P direct-runner,spark-runner

# Bounded (batch) execution with Direct runner
./run.sh DirectRunner bounded

# Bounded (batch) execution with Spark runner
./run.sh SparkRunner bounded

# Unbounded (streaming) execution with Direct runner
./run.sh DirectRunner unbounded

```
