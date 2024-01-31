# HSA7
Choose proper architecture pattern for Instagram services

#### Reference
http://highscalability.com/blog/2011/12/6/instagram-architecture-14-million-users-terabytes-of-photos.html

## Requirements
- We analyse just news feed that contains video and photos.
- We interested only in likes and comments functionality

## Emplore Instagram infrastructure
![Instagram architecture](./Instagram_scheme.jpg?raw=true "Instagram architecture")

The system will have more read activity than write activity - read-heavy

#### Read steps:
1. For reading news feed, likes, and comments - the client use DNS to find a Load Balancer closer to the customer
2. The client use Load Balancer to reach the Read API
3. Read API provides content from the Cache service, if possible. Otherwise - read from Read DB (replica)
4. For displaying of video and images - the client use CDN to speed up loading of content
5. CDN fetching content from Object storage (like S3)

#### Write steps:
1. For writing of a new post - the client uses Load Balancer to reach the Write API
2. The new post starts a few actions: writing in Object storage, writing in the Main DB (primary), and sending notifications through the Notification service
New likes or comments only reach the Main DB (primary) and send notifications through the Notification service
3. Writing to Object storage calls Metadata service, which extracts information from media files and puts them in Main DB (primary)

#### Potential bottlenecks and SPOFs

Bottlenecks

For write steps:

- notification service in case of a single event generates multiple notifications
- Main DB (primary) as it is utilized by Write API directly and Metadata service after processed media data

For read steps:
- Cache service. In case the resource limit is reached - loading will be slowed down by slower Read DB
---

SPOFs

For write steps:
- Main DB (primary) as it is single storage for all writing actions
- Load Balancer as it is single entry point for all actions

For read steps:
- Load Balancer as it is single entry point for all actions
