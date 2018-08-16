# Learning-ARKit-Basics

## ARKit Project Setup

1. Create a new `Single View App`
2. Go to `Main.storyboard` and delete `UIView`
3. Go to `Object Liberary` and drag-and-drop `ARKit SpriteKit View` into the `View Controller`
4. Create a `SpriteKit Scene` file under iOS -> Resources section. Give it a name like `Scene` and hit OK.
5. Create a swift class called `Scene.swift` and associate it with `Scene.sks`. To do that, click `Cmd+N` and select Swift under iOS->Source section. Again, call is `Scene`
6. Replace the code with the following code snippet.
```
import SpriteKit
import ARKit

class Scene: SKScene {
    
}
```
7. Connect `Scene.sks` with `Scene.swift`. To do that, select `Scene.sks` and go to `Custom Class Inspector` on the right panel. Enter "Scene" to the `Custom Class` field.
8. Go to `Main.storyboard`. Click Assistant Editor view to see both `View Controller` and `Main.storyboard` at the same time. Ctrl+Click and drag from `ARSKView` to the `View Controller` to create a view outlet. You can call it `sceneView`.
```
@IBOutlet var sceneView: ARSKView!
```
9. Go to `View Controller` and import ARKit and SpriteKit
```
import ARKit
import SpriteKit
```
10. In order to display the `Scene`, we can add the following code in `viewDidLoad` method.
```
if let scene = SKScene(fileNamed: "Scene") {
    sceneView.presentScene(scene)
}
```
11. We need a permission from user to use the camera. In order to do that, go to Info.plist, right click and select `Add row` and from the list, select `Privacy - Camera Usage Description`. We can add a custom message indicating why we need this permission by specifying the `Value` field. It can be something like "This app will use the camera for AR"

## Check for AR Compatibility

1. Go the Info.plist, expand "Required device capabilities" list, select `Item 0` and click plus button. This will create another row with name "Item 1". Specify its value as "arkit"
2. Go to `AppDelegate` file and import ARKit
```
import ARKit
```
3. Add the following code to the `application` method in `AppDelegate`
```
guard ARWorldTrackingConfiguration.isSupported else {
    fatalError("ARKit is not available.")
}
```
4. Try running the application on Simulator. Since simulator does not have a camera, we will get a fatal error that we just created.

## Basic AR Configuration

1. First of all, add `viewWillAppear` and `viewWillDisappear` methods to `View Controller` and specify the configuration.
```
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)

    let configuration = ARWorldTrackingConfiguration()

    sceneView.session.run(configuration)
}

override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    sceneView.session.pause()
}
```
2. Extend `ViewController` with `ARSKViewCDelegate` by adding the following code in `ViewController`. In this section, we can show an error if something goes wrong about the session like if session interrupted or when the interruption ends.
```
extension ViewController : ARSKViewDelegate {
    func session(_ session: ARSession, didFailWithError error: Error) {
        // Present an error message to the user
        
    }
    
    func sessionWasInterrupted(_ session: ARSession) {
        // Inform the user that the session has been interrupted, for example, by presenting an overlay
        
    }
    
    func sessionInterruptionEnded(_ session: ARSession) {
        // Reset tracking and/or remove existing anchors if consistent tracking is required
        
    }
}
```
3. At this point, we are able to run the app and see the AR session is running on our phone.
