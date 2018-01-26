# Attachment-Handler-Swift
Access Camera, Photo Library, Video and File from User device using Swift 4 
To see complete description of this AttachmentHandler. I would like to take you guys here. [Click here](https://medium.com/@deepakrajmurugesan/swift-access-ios-camera-photo-library-video-and-file-from-user-device-6a7fd66beca2)

<H1><B> How to Use </B></H1>

```
AttachmentHandler.shared.showAttachmentActionSheet(vc: self)
 AttachmentHandler.shared.imagePickedBlock = { (image) in
 /* get your image here */
 }
 AttachmentHandler.shared.videoPickedBlock = {(url) in
 /* get your compressed video url here */
 }
 AttachmentHandler.shared.filePickedBlock = {(filePath) in
 /* get your file path url here */
 }
 ```
 
 
