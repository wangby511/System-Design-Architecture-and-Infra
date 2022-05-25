# System Design - File System

CREATED 2022/05/21

## Functional Requirements

Upload and download files (can support any file formats)

Files sync across multiple devices

Notifications

Share files with links to friends/public

## Non-Functional Requirements

Reliability - data loss is unacceptable

Scalability - should handle large volume of traffic

High availability

## Estimations

50 million users with 10 million DAU

Every user has 10 GB free space - 10 GB x 50 million = 500 PB

Every user uploads 10 files per day - 1000 QPS

## Basic Concepts

### Main Components

Load balancer - distribute (network traffic) requests among API servers

API servers - handle logic for uploading and downloading files

Metadata database - for files, users, blocks, versions, ect. We choose relational databases. For this database we need to set up data replication and sharding to meet availability and scalability requirements.

Metadata cache - store some of the metadata cache for fast retrieval.

Notification service - notify relevant clients or different platforms when a file is added/edited/removed.

**Block servers** - Upload blocks to cloud storage. A file can be split into several blocks, each with a unique hash value, stored in metadata database. E.g. Dropbox sets the maximal size of a block to 4MB. Block servers process files passed from clients by splitting a file into blocks, **compress** each block and **encrypt** them. And then upload them into cloud storage in parallel.

File/Cloud Storage - A file is split into smaller blocks and stored into cloud storage.

(Offline backup queue)

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

### Downloading Files

Fetch metadata of the file returned by API server from querying metadata DB. Then the client sends request to block servers to download all the blocks and re-construct the file.

## Notification Service

long-polling - used by Dropbox. Since communication here is not bi-directional. The server sends information about file changes to the client only. We do not adopt WebSocket since it is suited for real-time bi-directional communication such as a chat app.

## Reliability

Multiple versions of the same file are stored across multiple data centers.

## Error Handling

Load balancer failure - the secondary one becomes active

Block server failure - the other servers pick up the unfinished and pending tasks

API server failure - replaced with another one since it is stateless

Notification server failure - over 1 million connections are open per machine. Replaced with another machine and re-establish/re-connecting is a relatively slow process.

## Follow Up

De-duplicate files for the account level - Eliminating the block with the same hash value in the same account.

How to handle conflict between two users are modifying the same file? - First version that gets processed wins. The second one receives a conflict which needs to be handled manually.

Why we need block servers in the middle layer? - We need chunking, compression and encryption for all types of clients. All those logics are implemented in a centralized place. Secondly, a client could be hacked and therefore it is not safe to do encryption on the client side.

## Reference

[1] System Design Interview Book. Alex Xu. Chapter 15. Design Google Drive.
