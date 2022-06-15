# System Design - File System

CREATED 2022/05/21 UPDATED 2022/06/13

## Functional Requirements

Upload, view and download files (can support any file formats)

Automatic files synchronization between multiple devices

Share files/folder with links to friends/public

Less important: Notifications

Extended: The system should support snapshot of files so that users can go back to any version of the files.

**Notice**: Need to clarify requirements like uploading, viewing and downloading, typical file size.

## Non-Functional Requirements

Reliability - data loss is unacceptable. Cloud storage ensures that users will never lose their data by keeping multiple copies of the data stored on different geographically located servers.

Scalability - should be able to handle large volume of traffic

High availability - Users can access their files/photos from any device whenever and wherever they like.

ACID-ity of files operations is required.

## Capacity Estimations

500 million users with 100 million DAU

Every user has 10 GB free space - 10 GB x 500 million = 5000 PB

Every user uploads 10 files per day - 10000 QPS

Assume that there are one million active connections per minute

## Basic Concepts

### Main Components

Load Balancer - distribute (network traffic) requests among API servers with load/traffic condition into consideration (more intelligent solution).

API Servers - handle logic for uploading and downloading files - Mainly 3 : Block Server, Metadata Server, Synchronization Server.

Metadata database - for files, users, blocks, versions, ect. We choose **relational databases** as they natively support ACID properties. For this database we need to set up data replication and sharding to meet availability and scalability requirements.

Metadata Cache & Storage - store some of the metadata cache for fast retrieval.

Notification service - notify relevant clients or different platforms when a file is added/edited/removed.

**Block servers** - Upload blocks to cloud storage. A file can be split into several blocks, each with a unique hash value, stored in metadata database. E.g. Dropbox sets the maximal size of a block to **4MB**. Block servers process files passed from clients by splitting a file into blocks, **compress** each block and **encrypt** them. And then upload them into cloud storage in parallel. If a user fails to upload a file, then only the failing block/chunk will be retried.

File/Cloud Storage - A file is split into smaller blocks and stored into cloud storage.

(Offline backup queue)

![image](https://cdn.hashnode.com/res/hashnode/image/upload/v1617952845935/_gcU4laIG.png)

### APIs

* Upload a file

* Download a file

* Get/List file revisions

All the APIs require user authentication and use HTTPS. Secure Sockets Layer (SSL) protects data transfer between the client and backend servers.

### Replication

Data can be replicated on the same-region and cross-region to guard against data loss and ensure availability.

## Metadata Database Schema

User DB - user id, email, name, profile photo, billing info

Device - device id, user id, last login time

File - fileId, file name, user id, is_directory, created time, last modified time, checksum, relative path

Block - stores everything related to a file block.

## Flows

### Uploading Files

**Delta sync** - When a file is modified, only the diff/delta blocks are synced instead of the whole blocks.

Steps: The client sends the new file metadata into metadata DB with marked "pending" status. Then uploads the content to block servers, forwarded to cloud storage. After the upload process is completed, mark status to "uploaded" in metadata DB and notify the user/clients.

Sync steps: Client A finishes uploading files, gets confirmation and notifications are sent to Clients B and C about the changes. Client B and C receive metadata changes and download updated chunks.

### Downloading Files

Fetch metadata of the file returned by API server from querying metadata DB. Then the client sends request to block servers to download all the blocks and re-construct the file.

## Notification Service

long-polling - used by Dropbox. Since communication here is not bi-directional. The server sends information about file changes to the client only. We do not adopt WebSocket since it is suited for real-time bi-directional communication such as a chat app.

## Reliability

### Replication Method (Preferred)

Multiple versions of the same file are stored across multiple data centers. Making 3 copies of data gives us 1 - 0.81% ^ 3 = 0.999999 reliability. (E.g. An Availability Zone (AZ) is one or more discrete data centers with redundant power, networking. Each AZ has many racks and each rack has many nodes.) 200% storage overhead.

### Erasure Coding Method

We need to calculate parities before data is written to disk. But only 50% storage overhead. Cons: In normal operation, every read has to come from multiple nodes in the cluster. Reads under a failure node are slower because missing data must be re-constructed first. Complicates the data model design.

## Metadata DB Partitioning

In our case, we can take the hash of the ‘FileID’ of the file object we are storing to determine the partition the file metadata will be stored.

## Error Handling

Load balancer failure - the secondary one becomes active

Block server failure - the other servers pick up the unfinished and pending tasks

API server failure - replaced with another one since it is stateless

Notification server failure - over 1 million connections are open per machine. Replaced with another machine and re-establish/re-connecting is a relatively slow process.

## Infra Graph

![image](https://cdn.hashnode.com/res/hashnode/image/upload/v1617952954142/fiX9kr9T_.png)

## De-duplication

De-duplicate files for the account level - Eliminating the block with the same hash value in the same account.

For each new incoming file, we can calculate a hash of it and compare that hash with all the hashes of the existing files to see if we already have the same file present in this account's storage. We prefer the **In-line de-duplication** approach instead of **Post-process de-duplication** to save network and bandwidth usage.

## Cache

To deal with hot files/chunks we can introduce **a cache layer for Block storage** with Least Recently Used (LRU) policy adopted. A high-end commercial server can have 144GB of memory; one such server can cache 36K chunks.

## Follow Up

How to handle conflict between two users are modifying the same file? - First version that gets processed wins. The second one receives a conflict which needs to be handled manually.

Why we need block servers in the middle layer? - We need chunking, compression and encryption for all types of clients. All those logics are implemented in a centralized place. Secondly, a client could be hacked and therefore it is not safe to do encryption on the client side.

Should we keep a copy of metadata with Clients? - Yes. Keeping a local copy of metadata not only enables us to do offline updates but also saves a lot of round trips to update remote metadata.

Load Balancer? - We can add the Load balancing layer at two places in this system: 1) Between Clients and Block servers and 2) Between Clients and Metadata servers.

Offline editing? - Users should be able to add/delete/modify files while offline. And as soon as they come online, all their changes should be synced to the remote servers and other online devices.

Message queue service - One of the clients sends the update of metadata to the **Request Queue** to Synchronization Service. Then it will send messages back through **Response Queues** that correspond to other individual subscribed clients. (When the Synchronization Service receives an update request, it checks with the Metadata Database for consistency and then proceeds with the update. Subsequently, a notification is sent to all subscribed users or devices to report the file update.)

How can clients efficiently listen to changes happening with other clients? - Long polling. If the server has no new data for the client when the poll is received, instead of sending an empty response, the server holds the request open and waits for response information to become available.

Permission? - We will be storing the permissions of each folder/file in the metadata DB to reflect whether the file of the folder is visible or modifiable by other users.

## Others

Block storage - good for virtual machines, high-performance applications like database.

File storage - general-purpose file system access.

Object storage - Binary data, unstructured data.

## Reference

[1] System Design Interview Book. Alex Xu. Chapter 15. Design Google Drive.

[2] <https://www.educative.io/courses/grokking-the-system-design-interview>

[3] <https://www.pankajtanwar.in/blog/system-design-how-to-design-google-drive-dropbox-a-cloud-file-storage-service>
