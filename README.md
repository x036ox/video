# yout-video

## Description

HLS video streaming api with recommendation system with Minio, Kafka and Ffmpeg.
Project contains UI written in React and is built in docker image. User authorization and oauth2
architecture handled by Keycloak.

## Architecture

![image](architecture.png)

## Get started

### Prerequisites

 - [Docker](https://www.docker.com/) installed

### Steps to start application

1. run "git clone https://github.com/x036ox/video.git"
2. run "docker-compose up"
3. go to "http://localhost"


## Walkthrough

For a detailed overview of the application's functionality, please see the [Testing Guide](./docs/testing-guide.md).

## Links

1. Github: https://github.com/x036ox/video
1. Linkedin: https://www.linkedin.com/in/artur-fenchenko-4b30b42a3/

## About project

Spring video api application that handles various video-related tasks, including creation, processing, storage, and access. 
Application supports caching via redis, allowing faster access to data. 
Videos processed into m3u8 and ts files in order to support HLS protocol. That allows to get videos by parts.
Videos are stored in MinIO object storage, which is compatible with Amazon S3. 
The system is composed of multiple microservices, each responsible for a specific function. 
Strong emphasis on security. User authorization implemented using Keycloak and its SPI.
The project includes a recommendation system microservice that utilizes complex SQL queries.
Communication between microservices is managed using Kafka.

When visiting the platform for the first time, there may not be any videos available. Users can either log in with
the default user credentials, which include admin privileges, and access the admin panel to generate videos and user
accounts, or they can upload their own videos. However, for testing purposes, it is recommended to use the first
option (more details in the Admin Panel section below).

#### The project primarily consists of these microservices:

1. video-api: This is the main application that handles all http requests, interacts with database, etc.
2. media-processor: Responsible for processing pictures and videos.
3. recommendations: Provides recommendations via http requests.
4. notifications: Service that produces notifications.

These services are connected via Kafka except for recommendations service. When a client sends an HTTP request, such as a request to create a video,
it is directed to the Youtback service. Youtback receives the request, creates an entity in the database, uploads
the received video and thumbnail to an object storage service, and sends a message through Kafka to notify the
Media Processor service of the processing requirements. The Media Processor then downloads the original video and
thumbnail from the object storage service, processes them, uploads the results to the object storage service, and sends
a response message back to Youtback. Once these steps are completed, the request is fulfilled. For a clearer understanding,
see architecture above. 

## Admin panel
This panel enables users to search for videos or users using multiple criteria such as the number of likes, the
number of videos, etc. Users can create new users and videos by simply clicking on the corresponding button. The data
for these entities will be sourced from pre-existing folders containing default videos and pictures. Every media (pictures
and videos) will be processed, so creating will take some time (max 3 min, depends on server's hardware).
Videos will be created immediately with a random number of likes from random users, and with a random date within the past 30 days. This setup
simulates real-world conditions to effectively test the recommendation system.

## Video uploading

To upload a video, click on the corresponding icon in the top right corner of the page (authentication required).
You can then upload an MP4 file, a thumbnail, set a title, description, and category for the video. Category is a
required field for the recommendation system (further details below). After uploading, the video will be converted to
M3U8 and TS files to support HLS technology. The thumbnail will be compressed and saved in the object storage service,
while metadata will be stored in the database. Once the upload is complete, the video will appear in your channel's
videos (top right corner) or can be found by title.

## Recommendation system

To collect data on the types of videos users are interested in, each video contains additional fields in its metadata.
Each video is assigned a category (e.g., "music"), and when a user watches a video, the system records the number of
videos watched within that category, referred to as points. For instance, if a user watches 50% of a video, a
corresponding number of points will be added. Consequently, the more points a user accumulates in a particular
category, the more videos of that category will be recommended. Additionally, the recommendation system considers
the languages preferred by the user. This logic mirrors the approach described earlier. If a user watches videos
without logging in, recommendations will be based on the languages detected in their browser settings. To optimize
performance, the system utilizes a single SQL request for each recommendation request (refer to the Javadoc for more details).
The most recommended videos will be with the most likes. Also takes in count the date when user liked this videos in order to
select the fastest growing videos recently. When creating video via admin panel, it will has random amount of likes from random users
so this will make testing easier.

