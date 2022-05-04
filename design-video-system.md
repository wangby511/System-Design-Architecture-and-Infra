# System Design - Youtube Video System

SET UP 2022/02/22 & 2021/04/19

## Functional Requirements

Upload a video

Watch a video

Share, like, comment, search, subscribe to a channel, add to playlist...

Questions to be confirmed - Type of clients, video resolutions, encryption required, maximum size limit of a video, leverage some existing cloud services like CDN or blob storage...

## Non-Functional Requirements

Reliable(not losing data)

Availability

Smooth video streaming

## Other Features

Recommendations, Top K popular...

## Estimation

DAU(daily active users) 5 million.

10% of the DAU upload a video every day. Average video size is 300MB.

The daily storage space is 5 million *10%* 300 MB = 150TB.

The upload-view(write-read) ratio is about 1:200 - a read-heavy system.

**Videos are stored in CDN. Everything else except video streaming goes through API servers.**

## APIs

uploadVideo(api_dev_key, video_title, video_description, tags[], category_id, default_language, recording_details, video_contents)

searchVideo(api_dev_key, search_key_word, maximum_items_to_return, next_page_token)

streamVideo(api_dev_key, video_id, offset, codec, resolution)

## DB Schema

### MySQL

1 Video metadata storage - MySQL

Videos metadata can be stored in a SQL database. The following information should be stored with each video:

VideoID, Title, Timestamp, Description, Size, Thumbnail, User, Total number of likes, dislikes, Total number of views

2 User data storage - MySQL

UserID, Name, Email, Address, Age, etc.

(Billing information - MySQL)

### NoSQL

Cassandra. It can handle heavy write and read.

It records the viewing history of users.

Pure video chunks are saved in S3.

## Video Uploading Flow

Upload Service: -> Distributed Storage & Metadata Cache/DB & Distributed Queue (tasks : Preprocessing, Video Splitter, Encoding)

Transcoding servers 1 -> Transcoded storage -> CDN.

   　　　　　　　　    2 -> Completion events stored at completion queue. -> Completion handler updates metadata database and cache.

User -> [**Original Storage**] -> [**Transcoding Servers**] split 1 -> [**Completion queue**] -> [**Completion Handler**] -> update Metadata DB/Cache.

split 2 -> [**Transcoded Storage**] -> [**CDN**]

A **Binary Large Object (BLOB)** is a collection of binary data stored as a single entity in a database management system.

Completion handler contains a bunch of workers that continuously pull event data from the queue. It updates the metadata database and cache when video transcoding is complete.

Conclusion of two steps: Upload the actual video. Update video metadata. Metadata contains information about video URL, size, resolution, format, user info, etc. Those two steps can be in parallel.

## Video Watching Flow

Videos are streamed from CDN directly. When you click the play button, a video is streamed from the CDN. The edge server closest to you will deliver the video.

## Details

### Video Transcoding

Different users have different network bandwidth, devices and browsers. Therefore, switching video quality automatically or manually based on network conditions is essential for smooth user experience.

Containers(contains the video file, audio and metadata) and Codecs(compression and decompression algorithms).

### DAG(Directed Acyclic Graph) Model

A directed acyclic graph (DAG) programming model defines tasks in stages so that they can be executed sequentially or in parallel.

**Preprocessor** - video splitting into small chunks like Group of Pictures(GOP) alignment. Also it caches/stores GOPs and metadata in temporary storage for possible retry operations.

**DAG Scheduler** - splits a DAG graph into stages of tasks and puts them in the task queue in the resource manager.

Video -> Inspection, Video transcoding, Thumbnail, ..., Watermark

Audio -> Audio Encoding

Metadata Cache/DB

Those 3 steps above can be in parallel while the system is doing a video uploading task.

**Resource Manager**

Task queue - a priority queue that contains tasks to be executed.

Worker queue - a priority queue that contains worker utilization info.

Running queue - contains info about the currently running tasks and workers running the tasks.

Task scheduler - picks the optimal task/worker, and instructs the chosen task worker to execute the job.

The resource manager works as follows:

* The task scheduler gets the highest priority task from the task queue.

* The task scheduler gets the optimal task worker to run the task from the worker queue.

* The task scheduler instructs the chosen task worker to run the task.

* The task scheduler binds the task/worker info and puts it in the running queue.

* The task scheduler removes the job from the running queue once the job is done.

## Sharding

### Sharding based on UserID

Hot user issue OR a user has many videos.

Need to query all servers if a user wants to search some video name.

Unbalanced storage. Some users have a lot of videos while others have few.

### Sharding based on VideoID

Popular videos(need cache).

A centralized server is going to aggregate and rank these results before returning them to customers.

## CDN

A CDN is a system of distributed servers that deliver web content to a user based on the user’s geographic locations. It replicates the content in multiple places around the world.

## Error Handling

API servers down - API servers are stateless so that the requests can be redirected to different backup API servers.

Metadata DB server down - following DB server changes to primary if the primary one has an outage.

## Follow Up

In order to make the system more loosely coupled, we can introduce message queues between each components. Like Original Storage -> Download Module -> Encoding Module -> Upload Module -> Encoded Module -> CDN.

Live Streaming - we may need a different streaming protocol because of higher latency requirement. Small chunks of data are already processed in real-time. It also requires different sets of error handling without taking too much time.

## Reference

[1] System Design Interview Book. Alex Xu. Chapter 14. Design Youtube.

[2] <https://www.cnblogs.com/wangby511/p/14708649.html>

[3] <https://systeminterview.com/design-youtube.php>
