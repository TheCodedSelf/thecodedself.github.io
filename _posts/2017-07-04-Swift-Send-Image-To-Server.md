---
title: "iOS: Working with Images from a Server"
---

If you're a mobile app developer, at some point in time you're going to need to interact with a backend. One of the tasks you might need to do is to retrieve and display images from a server, or submit an image to that server. What format should the image be in when you're submitting it? How do you convert bytes received from a service call into an image?

Let's build the entire stack from the server to an iOS App to find out how.

# Setting up a backend

We'll start off by building a [Kitura](https://github.com/IBM-Swift/Kitura) server that provides a RESTful API to do two things:
- Receive images from a client
- Provide the most recent image that was received to a client

I've put the finished server up [on my Github](https://github.com/TheCodedSelf/SwiftImageServer).

### Create the server project

Make a directory, and init a new executable Swift package.

```
mkdir mkdir SwiftImageServer && cd SwiftImageServer
swift package init --type executable
```

Edit your Package.swift file to specify that you require the Kitura package.

```swift
import PackageDescription

let package = Package(
    name: "SwiftImageServer",
    dependencies: [
      .Package(url: "https://github.com/IBM-Swift/Kitura.git", majorVersion: 1)
    ])
```

You can run a `swift package fetch` and you should see SwiftPM cloning Kitura and everything that it requires.

Spin up a xcodeproj with `swift package generate-xcodeproj` and let's get coding!

### Create a Kitura server

The backend is going to be super simple, so we'll just be working in `main.swift`.

Let's start off by adding all the boilerplate that we need:

```swift
import Kitura
import Foundation

// Create a Router that we can use to create REST endpoints
let router = Router()

// Specify that we want an HTTP server that we can reach with http://localhost:8090
Kitura.addHTTPServer(onPort: 8090, with: router)


// Start the server
Kitura.run()
```

I love how easy it is to create something these days. Three lines of code and you have a server running. It's a shame that it can't do much. Let's fix that.

### Returning an image from a GET endpoint

```swift
var latestImage: Data? = nil

// http://localhost:8090/latestImage
router.get("/latestImage") {
    request, response, next in
    
    defer { next() }
    
    guard let image = latestImage else {
        response.status(.preconditionFailed).send("No image is available")
        return
    }
    
    response.send(data: image)
}
```

<p style="text-align:center"><img alt="Wait a second!" src="https://media.giphy.com/media/l0G18gptStsYQrcL6/giphy.gif"/></p>

You thought we were sending images, you say? That looks like a Data object, **not** a UIImage? That's where the fun comes in. You never send an image **as an image**. All images are simply **data** packaged in an easy to use format. When we're sending images to and from our server, we need to package it as a Data object and send that instead. We'll represent it as a UIImage in our iOS app.

Notice the `guard let image = latestImage`. That'll fail until we set the `latestImage` variable. Let's build the endpoint that receives an image, so we can set the `latestImage` variable.

### Submitting images to a POST endpoint

Next, we'll build the endpoint that will be used to submit images.

```swift
// Create a POST endpoint: http://localhost:8090/image
router.post("/image") {
    request, response, next in
    
    defer { next() }
    
    var data = Data()
    
    do {
        // Read the body of the request into the data object
        try _ = request.read(into: &data)
        latestImage = data
	response.status(.OK).send("Image received")
    } catch(let error) {
        response.status(.internalServerError).send("Something went wrong when reading the image data")
    }
}
```

We've created an endpoint that expects a POST request with a raw body of the image data. Remember, the server only knows Data, not UIImage, so the iOS app is going to have to convert the image to a Data object.

That's our entire server done!

### Up and running

Here's a link to the completed server: [https://github.com/TheCodedSelf/SwiftImageServer](https://github.com/TheCodedSelf/SwiftImageServer)

Run the executable target. That's the one with the Matrix-style computer screen, not the yellow lunchbox.

<p style="text-align:center"><img alt="Kitura server executable target" src="{{ site.url }}/assets/images/SwiftImageServer/RunTarget.png"/></p>

Keep that running, and we'll build the iOS app.

# The client app

The complete example code for the iOS app is available [on my Github](https://github.com/thecodedself/LatestImageRetriever).

### Create the project

Create a new Single View iOS Application. We're going to need to modify the `Info.plist`. We need permission to access the Photo Library to pick an image. We also need to modify the App Transport Security settings to make HTTP network requests, as opposed to HTTPS (our local Kitura server is HTTP).

Add the following to the project's `Info.plist`:

<p style="text-align:center"><img alt="Our changes to Info.plist" src="{{ site.url }}/assets/images/SwiftImageServer/Photo Library Permission.png"/></p>

### Submitting images to the server

We're going to add a button that'll allow us to select an image from the photo gallery and submit it to the server we built earlier.

We'll start off by adding a button to the view controller in `Main.storyboard`:

<p style="text-align:center"><img alt="The view with a Pick Image button" src="{{ site.url }}/assets/images/SwiftImageServer/Pick Image Screenshot.png"/></p>

Now, hook it up to `ViewController.swift` and add the code to post it to our backend.

Let's connect an IBAction to the 'Pick Image' button where we pick an image using a UIImagePickerController.

```swift
@IBAction func pickImage(_ sender: Any) {
        
        guard UIImagePickerController.isSourceTypeAvailable(.photoLibrary) else { return }
        let imagePickerController = UIImagePickerController()
        imagePickerController.sourceType = .photoLibrary
        imagePickerController.delegate = self
        present(imagePickerController, animated: true, completion: nil)
    }
```

We're setting the ViewController as the UIImagePickerController's delegate. Add an extension that conforms to the delegate protocol that also handles any picked images. Setting the delegate also requires conformance to UINavigationControllerDelegate, so add that in as well.

```swift
extension ViewController: UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    
    public func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [String: Any]) {
        
        if let image = info[UIImagePickerControllerOriginalImage] as? UIImage {
            submit(image: image)
        } else if let image = info[UIImagePickerControllerEditedImage] as? UIImage {
            submit(image: image)
        }
        
        picker.dismiss(animated: true)
    }
}
```

We can select an image, and we're ready to handle it. Notice the `submit(image:)` function in the above example. We'll create that function now, and submit the image to our server.

```swift
func submit(image: UIImage) {

        let session = URLSession(configuration: URLSessionConfiguration.default)
        
        guard let url = URL(string: "http://localhost:8090/image") else { return }
        var request = URLRequest(url: url)
        
        request.httpMethod = "POST"
        request.httpBody = UIImagePNGRepresentation(image)
        
        let dataTask = session.dataTask(with: request) { (data, response, error) in
            
            if let error = error {
                print("Something went wrong: \(error)")
            }
            
            if let response = response {
                print("Response: \n \(response)")
            }
        }
        
        dataTask.resume()
    }
```

### Receiving an image from the server

Now that we have the ability to submit images as Data objects, we need to build in the functionality to receive an image as Data and convert it to a plain old UIImage.

Start by adding an image view and another button to the view controller in Main.storyboard:

<p style="text-align:center"><img alt="Button and image view for receiving image from our server" src="{{ site.url }}/assets/images/SwiftImageServer/Completed Storyboard.png"/></p>

When the user taps on the new button we'll call the `latestImage` endpoint to retrieve the last image that was sent to our server as a Data object. We'll then convert it to a UIImage and display it in the image view.

```swift
@IBOutlet weak var imageView: UIImageView!
    
    @IBAction func showLatestImage(_ sender: Any) {
        
        let session = URLSession(configuration: URLSessionConfiguration.default)
        
        guard let url = URL(string: "http://localhost:8090/latestImage") else { return }
        
        var request = URLRequest(url: url)
        request.httpMethod = "GET"
        
        session.dataTask(with: request) { (data, response, error) in
            
            if let error = error {
                print("Something went wrong: \(error)")
            }
            
            if let imageData = data {
                DispatchQueue.main.async {
                    self.imageView.image = UIImage(data: imageData)
                }
            }
        }.resume()
    }
```

Looking good. You completed ViewController should look like this one: [https://github.com/TheCodedSelf/LatestImageRetriever/blob/master/LatestImageRetriever/ViewController.swift](https://github.com/TheCodedSelf/LatestImageRetriever/blob/master/LatestImageRetriever/ViewController.swift)

# A Finished Product

Here's our app that has submitted an image to the server and pulled an image from the server to display.

<p style="text-align:center"><img alt="Finished Product" src="{{ site.url }}/assets/images/SwiftImageServer/FinishedProduct.png"/></p>

When you tap on Pick Image, you're presented with a UIIImagePickerController that allows you to select an image from the Photos Library, and submits it to our backend after converting it to a Data object. The Show Latest Image button does a GET request to our server to retrieve the last image that was sent, converts it to a UIImage, and then displays it in our UIImageView.

You should now have a basic understanding of how to send objecs besides text to a server - and how to retrieve them. Congratulations!

<p style="text-align:center"><img alt="Congrats!" src="https://media.giphy.com/media/Wrv5v6egIh7Ww/giphy.gif"/></p>

Have fun, and happy Swifting!
