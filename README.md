# Attachment-Handler-Swift
Photos, videos, documents are critical to iPhone’s users — It’s how they communicate and share everyday moments at their day to day life.

While developing this feature for the iOS version of the app, I realized that there was no single code-block to access media files from the device.

And that’s how this piece came about. If you need a single piece of code to do the same, feel free to plug this into your app.

The following is a step-by-step tutorial to create a custom class that will help developers access basic attachments such as Camera Image, Photo Library, Video and File Import using swift 4.

TL:DR — You can see the complete GIST file of it here. I have named it as AttachmentHandler.swift

## Step 1: Create Action Sheet using UIAlertController

Create an action sheet by using UIAlertController. I’m going to list down, all four options in action sheet along with cancel button in bottom.

If you want to know more about UIAlertController. I would kindly request you to check here. You can also look for UIActionSheet but this is depricated.

```    
    func showAttachmentActionSheet(vc: UIViewController) {
            currentVC = vc
            let actionSheet = UIAlertController(title: Constants.actionFileTypeHeading, message: Constants.actionFileTypeDescription, preferredStyle: .actionSheet)
            
            actionSheet.addAction(UIAlertAction(title: Constants.camera, style: .default, handler: { (action) -> Void in
                self.authorisationStatus(attachmentTypeEnum: .camera, vc: self.currentVC!)
            }))
            
            actionSheet.addAction(UIAlertAction(title: Constants.phoneLibrary, style: .default, handler: { (action) -> Void in
                self.authorisationStatus(attachmentTypeEnum: .photoLibrary, vc: self.currentVC!)
            }))
            
            actionSheet.addAction(UIAlertAction(title: Constants.video, style: .default, handler: { (action) -> Void in
                self.authorisationStatus(attachmentTypeEnum: .video, vc: self.currentVC!)
                
            }))
            
            actionSheet.addAction(UIAlertAction(title: Constants.file, style: .default, handler: { (action) -> Void in
                self.documentPicker()
            }))
            
            actionSheet.addAction(UIAlertAction(title: Constants.cancelBtnTitle, style: .cancel, handler: nil))
            
            vc.present(actionSheet, animated: true, completion: nil)
    }
```

## Step 2: Check For Authorisation Status

..from PHPhotoLibrary. This means that we have to verify if the user has given permission to access their photo or not.

Add two lines in Info.plist

```
Privacy — Camera Usage Description 
Privacy — Photo Library Usage Description
```

The above two lines should be added in to your app’s Info.plist with sample description as
``` “$(PRODUCT_NAME) would like to access your camera and $(PRODUCT_NAME) would like to access your photo.” ```

```
    func authorisationStatus(attachmentTypeEnum: AttachmentType, vc: UIViewController){
        currentVC = vc
        
        let status = PHPhotoLibrary.authorizationStatus()
        switch status {
        case .authorized:
            if attachmentTypeEnum == AttachmentType.camera{
                openCamera()
            }
            if attachmentTypeEnum == AttachmentType.photoLibrary{
                photoLibrary()
            }
            if attachmentTypeEnum == AttachmentType.video{
                videoLibrary()
            }
        case .denied:
            print("permission denied")
            self.addAlertForSettings(attachmentTypeEnum)
        case .notDetermined:
            print("Permission Not Determined")
            PHPhotoLibrary.requestAuthorization({ (status) in
                if status == PHAuthorizationStatus.authorized{
                    // photo library access given
                    print("access given")
                    if attachmentTypeEnum == AttachmentType.camera{
                        self.openCamera()
                    }
                    if attachmentTypeEnum == AttachmentType.photoLibrary{
                        self.photoLibrary()
                    }
                    if attachmentTypeEnum == AttachmentType.video{
                        self.videoLibrary()
                    }
                }else{
                    print("restriced manually")
                    self.addAlertForSettings(attachmentTypeEnum)
                }
            })
        case .restricted:
            print("permission restricted")
            self.addAlertForSettings(attachmentTypeEnum)
        default:
            break
        }
    }
```

Where as AttachmentType is a enum.

```
enum AttachmentType: String{
 case camera, video, photoLibrary
}
```

Authorisation status of PHPhotoLibrary comes with 4 statuses. They are authorized, denied, notDetermined, restricted.

If it is denied or restricted, we have to add an alert to the user saying the user to turn on access by going to settings manually. This will help us to access the attachments. if it is authorized, access the required type of attachment with the help of AttachmentType enum.

## Step 3: Opening camera, gallery and video

### openCamera — sourceType: Camera

Use UIImagePickerController class for camera, video and photo library. Then, change the sourceType property to the suitable one for camera and photo library.

```
func openCamera(){
    if UIImagePickerController.isSourceTypeAvailable(.camera){
        let myPickerController = UIImagePickerController()
        myPickerController.delegate = self
        myPickerController.sourceType = .camera
        currentVC?.present(myPickerController, animated: true, completion: nil)
    }
}
```

### accessPhotoLibrary — sourceType: photoLibrary

Just change the sourceType property here 

```
func photoLibrary(){
    if UIImagePickerController.isSourceTypeAvailable(.photoLibrary){
        let myPickerController = UIImagePickerController()
        myPickerController.delegate = self
        myPickerController.sourceType = .photoLibrary
        currentVC?.present(myPickerController, animated: true, completion: nil)
    }
}
```

### accessVideo

To access video from the photo library, we have to use mediaType as kUTTypeMovie and kUTTypeVideo

mediaTypes = [kUTTypeMovie as String, kUTTypeVideo as String]

```
func videoLibrary(){
    if UIImagePickerController.isSourceTypeAvailable(.photoLibrary){
        let myPickerController = UIImagePickerController()
        myPickerController.delegate = self
        myPickerController.sourceType = .photoLibrary
        myPickerController.mediaTypes = [kUTTypeMovie as String, kUTTypeVideo as String]
        currentVC?.present(myPickerController, animated: true, completion: nil)
    }
}
```

## Step 4: accessFile

With iOS 11, Apple has brought in the File browser feature. With this, we can access any type of document, be it PDF, PPT etc. However, unlike Android, iOS doesn’t allow you to access the files stored physically on your device, but yeah, you can do the same via iCloud(or Google Drive, Dropbox, cloud storage spaces).

All you need to do is turn on iCloud and Keychain Sharing for your app. You can do this in capabilities section(XCode). Make sure to enable the CloudKit for your app.

```
func documentPicker(){
    let importMenu = UIDocumentMenuViewController(documentTypes: [String(kUTTypePDF)], in: .import)
    importMenu.delegate = self
    importMenu.modalPresentationStyle = .formSheet
    currentVC?.present(importMenu, animated: true, completion: nil)
}
```

## Step 5: Camera, PhotoLibrary and Video Delegate Methods

To start off, create an extension with two delegates UIImagePickerControllerDelegate, UINavigationControllerDelegate.

Then, create the following methods: imagePickerController, imagePickerControllerDidCancel.

As the name suggests, the first method is to pick an image and the next one is to cancel the picker.

```
@objc func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [String : Any]) {
        
// To handle image
    if let image = info[UIImagePickerControllerOriginalImage] as? UIImage {
        self.imagePickedBlock?(image)
    } else{
        print("Something went wrong in  image")
    }
// To handle video        

    if let videoUrl = info[UIImagePickerControllerMediaURL] as? NSURL{
        print("videourl: ", videoUrl)
        //trying compression of video
        let data = NSData(contentsOf: videoUrl as URL)!
        print("File size before compression: \(Double(data.length / 1048576)) mb")
        compressWithSessionStatusFunc(videoUrl)
    }
    else{
        print("Something went wrong in  video")
    }
    currentVC?.dismiss(animated: true, completion: nil)
}
```
Video can be picked up as a local media URL. We have used compression technique to compress it to 1280x720 pixel quality.

To compress videos, we’ve used AVAssetExportSession. You can use your own version of compression technique. If needed you check in the code for it also. which is available.

Now you’ve picked photos and videos from camera, and library to use it in any class. I have created a closure called videoPickedBlock and imagePickedBlock. Here is how we set it.

```
static let shared = AttachmentHandler()
fileprivate var currentVC: UIViewController?
//MARK: - Internal Properties
var imagePickedBlock: ((UIImage) -> Void)?
var videoPickedBlock: ((NSURL) -> Void)?
var filePickedBlock: ((URL) -> Void)?
```

## Step 6: File Import Delegate Methods

Now create another extension with two delegates (UIDocumentMenuDelegate, UIDocumentPickerDelegate) with methods didPickDocumentPicker, didPickDocumentAt and documentPickerWasCancelled. Use the picked file with filePickedBlock closure.

```
func documentMenu(_ documentMenu: UIDocumentMenuViewController, didPickDocumentPicker documentPicker: UIDocumentPickerViewController) {
        documentPicker.delegate = self
        currentVC?.present(documentPicker, animated: true, completion: nil)
}
    
    
func documentPicker(_ controller: UIDocumentPickerViewController, didPickDocumentAt url: URL) {
    print("url", url)
    self.filePickedBlock?(url)
}
    
//    Method to handle cancel action.
func documentPickerWasCancelled(_ controller: UIDocumentPickerViewController) {
    currentVC?.dismiss(animated: true, completion: nil)
}
```

## Usage
So yea.. Hurray! We’ve finally imported media and documents from iPhone using Swift 4.

This is the class we created. Two deceivingly simple lines of code to do all your heavy lifting.

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
 
## Constants used in class

```
struct Constants {
   static let actionFileTypeHeading = "Add a File"
   static let actionFileTypeDescription = "Choose a filetype to add..."
   static let camera = "Camera"
   static let phoneLibrary = "Phone Library"
   static let video = "Video"
   static let file = "File"   
   static let alertForPhotoLibraryMessage = "App does not have access to your photos. To enable access, tap settings and turn on Photo Library Access.
   static let alertForCameraAccessMessage = "App does not have access to your camera. To enable access, tap settings and turn on Camera."
   static let alertForVideoLibraryMessage = "App does not have access to your video. To enable access, tap settings and turn on Video Library Access."
   static let settingsBtnTitle = "Settings"
   static let cancelBtnTitle = "Cancel"     
}
```

 
