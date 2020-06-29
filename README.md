# smartcam
Smart, low-cost, self-contained, cooperative IP security camera system in .NET for Raspberry Pi devices.

2020-06 - This is a new project in the earliest stages of planning, testing, and design. As I make progress I will begin documenting things like the basic setup process.

## Requirements / Assumptions
* Raspberry Pi 4 with 4GB RAM (likely also compatible with v3, I haven't tried yet)
* Camera using the Pi's on-board camera port (5MP preferred over newer 8MP version)
* .NET and ASP.NET runtimes; SDK not required on-board the Pi; currently targeting 3.1.x
* Raspbian OS (lite is OK but some of the setup process may be easier through the GUI)
* Network-based centralized storage for video clips (e.g. a NAS or dedicated file server)
* Cameras are on a trusted network (e.g. adequately firewalled on your home network)

## Features / Goals
* Fast new-device setup: copy SD card, change one config file, boot, rename the Pi, run updates
* Self-organizing: the cameras discover other smartcams on the same network (group triggers)
* Low traffic: 100% local processing, only copying recordings to storage is high-traffic
* SD-card friendly: caching and encoding on ramdisk, logging on network
* Self-managing: `systemctl` watchdog processes
* User-friendly: easy web-based UI for interacting with the entire system
* DHCP-friendly, no need to pre-assign IPs (a common requirement with IP cams on DVR servers)
* TBD: support MJPEG / RTSP for interactive browser viewing?

## The smartcam Process
* Console program on each Pi device
* Manages the pre-record buffer
* Performs motion detection
* Manages recording and saving to storage
* Communicates via websockets with local "smartcam-host" process

## The smartcam-host Process
* Web server on each Pi device
* Basic browser UI for interacting with an individual camera (not secure)
* Simple REST API for interacting with other smartcam-host processes
* ASP.NET Kestrel reverse-proxy setup behind ngnix
* Interacts with "smartcam" process via websockets
* Discovers other smartcam servers on the same network
* Sends and receives group notifications when motion detection triggers recording
* Manages network storage maintenance (trimming old files by date and/or free space)

## The smartcam-master Server
* Web server, one per network, not necessarily Pi-hosted
* UI provides a system-wide view (similar to traditional IP cam DVRs)
* Basic security (we assume this is potentially publicly accessible)
* Interacts with all known smartcam-host processes

## Notes About Network Storage
* A top-level directory is dedicated to all cameras, for example `//NAS1/smartcams`
* Each smartcam creates it's own subdirectory, for example `//NAS1/smartcams/frontporch`
* The smartcam updates `discovery.txt` with information to help the hosts with discovery
