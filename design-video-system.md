# System Design - Video System

SET UP 2022/02/22 & 2021-04-19

## Functional Requirements

upload a video

watch a video

share, like, comment, search, subscribe to a channel...

Questions to be confirmed - Type of clients, video resolutions, encryption required, size limit of video, leverage some existing cloud services like CDN or blob storage...

## Non-Functional Requirements

reliable(not losing data), available, real-time...

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

searchVideo(api_dev_key, search_word, user_location, maximum_videos_to_return, next_page_token)

streamVideo(api_dev_key, video_id, offset, codec, resolution)

## DB Schema

MySQL

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

## Service Flows

### Upload a Video

![img](https://systeminterview.com/imgs/ch14/14-5.png)

Upload Service: -> Distributed Storage & Metadata Database & Distributed Queue (tasks : Preprocessing, Video Splitter, Encoding)

Transcoding server 1 -> Transcoded storage -> CDN.

   　　　　　　　　   2 -> Completion events stored at completion queue. -> Completion handler updates metadata database and cache.

### Watch a Video

Watch Video Service: -> Distributed Storage & Metadata Database

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

API server down - back up machines.

DB server down - secondary DB changes to primary if the primary one has an outage.

## Reference

[1] <https://www.cnblogs.com/wangby511/p/14708649.html>

[2] <https://systeminterview.com/design-youtube.php>
