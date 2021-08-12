# Ermes
This repository contains the general documentation of the project "Ermes", that won a prize in the "NOI Hackathon Summmer Edition 2021" that was held in Scena/Schenna, Italy. The chosen challenge consisted in developing a software for some ESP32-CAMs, that were to send periodically photos to a server an be easily configured.

## Our solution

Out solution consisted of:
1. The software (arduino) written for the ESP32-CAMs. It was written of course in C++ and his duty was taking the photo, using an SD as a fallback and sending the photos to a server when online.
2. A NodeJS server that provides some APIs used by the cameras to take configurations and current time. The server was also responsible to save the images in the filesystem.
3. A MongoDB database that stored information such as the configurations of each camera and their error logs.
4. A web frontend written in Vue.js, whose main aim was making it possible to configure the cameras remotly. In addition, it showed the photos and the error logs.

## More details

### The data

In the database, there are two collections, `configs` and `errorLogs`.

Configs contain the configurations of each camera, the properties are:
- **id**: (MAC addr.) of the camera
- **paused**: If the camera is paused, meaning it should not taking photos
- **backup**: If it should save in any case photos in the SD. (In case the SD is full, the oldest photo should be in any case erased, like in a normal queue)
- **resolution**: From `0` to `63`, it is the resolution of the taken photo
- **pxFormat**: For example `PIXFORMAT_JPEG` or `PIXFORMAT_GRAYSCALE`, to say the format of the pixels of the taken photo
- **hours**: It is an array of hours (es. `10:30` or `23:23`). Each day, at those hours, a photo should be taken
- **lastModified**: The timestamp of when the user last changed these settings from the frontend
- **lastPinged**: The timestamp of when the camera last pinged the server, by retrieving the updated configurations and the current timestamp

![image](https://user-images.githubusercontent.com/33126163/129223867-e91e18aa-e692-4167-bdea-76282c6f6fb7.png)

Error logs contain the logs of each camera:
- **id**: (MAC addr.) of the camera
- **error**: A number specifying the error code of the error. (For instance 440 could be 'SD not connected')
- **timestamp**: The timestamp of the error

![image](https://user-images.githubusercontent.com/33126163/129225429-aa1c09cb-a549-4cb8-95b5-f124ec0a4e83.png)

The only implemented error during the hackathon was the SD error, but the log could be useful for plenty of other situations, from each error that can occur to a "powerbank should be charged" warnings.

### The api

#### GET `/version`

Returns the version of the server

#### GET `/machines`

Returns all the avalable machines (cameras). They are all the cameras that ever connected to the server.

#### GET `/machines/:id`

Returns the configurations of the machine (camera) with that id and returns also the current time.
If the query param `?initialConnection=1` is provided, the server knows that this is the first connection that the camera does after having been turned on. This is because if `lastModified` is before `lastPinged`, the server knows that the configurations in the camera are updated and returns `null` instead of the configurations, but if the camera is turned off and then on, the camera needs in any case to get the configurations.

#### GET `/machines/:id/errors`

Returns all the errors of the machine with that id.

#### GET `/machines/:id/frontend`

Returns the config of the machine with that id, by knowing that the request is done by the frontend (for instance `lastPinged` is not updated).

#### POST `/machines/:id`

Updates the configs of the machine with that id

#### POST `/machines/:id/errors`

Adds an error to the database of the machine with that id.

#### POST `/machines/:id/image`

Uploads an image to the server, in the folder of the machine with that id

#### GET `/machines/:id/images`

Returns all the paths to the statically served images of the machine with that id

### The flow

Generally, the system was supposed to work like this:
1. A camera is turned on for the first time
2. The first thing it does is connecting to the server, by using its MAC address as unique id
3. The API call gets the current time and the default configurations. A document with the default configurations associated to the id is inserted in MongoDB.
4. 

## Possible applications

The cameras are not expansive and many of them could be bought without a huge amount of money. They require also not much power (this was also a requirement), so a normal powerbank could maintain them for weeks.

The application that was suggested by the ideators of the challenge was taking periodically photos in agricultural fields, sending them to a server. In the server the images are then analyzed so that various important information can entail a better management of that field.
