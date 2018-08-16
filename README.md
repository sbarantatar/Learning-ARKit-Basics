# Learning-ARKit-Basics

## Part 1

### ARKit Project Setup

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

### Check for AR Compatibility

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

### Basic AR Configuration

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

### Hit Testing

1. With ARKit and SpriteKit, we will add 2D object into the real world. Before doing that, let's add an image to our assets. We can do that by simply dragging and dropping our image to the assets folder in our project.
2. Add the following code to the `Scene.swift`
```
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        
    guard let sceneView = self.view as? ARSKView else {return}

    if let touchLocation = touches.first?.location(in: sceneView) {
        if let hit = sceneView.hitTest(touchLocation, types: .featurePoint).first {
            sceneView.session.add(anchor: ARAnchor(transform: hit.worldTransform))
        }
    }

}
```
3. Go to `ViewController` and let's assign our `ARSKViewDelegate` to our scene view. We can do that bay adding the following code to the `viewDidLoad` method.
```
sceneView.delegate = self
```
4. Add the following code snippet to the `ARSKViewDelegate`. This will create a node for our image for the anchor we created before.
```    
    func view(_ view: ARSKView, didAdd node: SKNode, for anchor: ARAnchor) {
        let imageNode = SKSpriteNode(imageNamed: "image-name")
        imageNode.xScale = 0.25
        imageNode.yScale = 0.25
        
        node.addChild(imageNode)
    }
```
5. Run the app and test the code by touching the screen.

## Part 2

### Creating a Menu

1. Add a new `Color Sprite` as a placeholder for our animation.
2. From action library, drag-and-drop `AnimationWithTextures Action` under newly created node.
3. Click on `Animate with Textures` action and observe that there is a `Textures` section on the right panel. Drag-and-drop animation images to this section.
4. Select the `Color Sprite` we created for our animation, and select the first animation image from the `Textures` dropdown menu on right panel. When we click the `Animate` button under the scene, we will see that our animation is fired up.
In order to make the animation continues,  select the animation action and hit the little circular arrow button on the bottom left corner and hit infinity symbol. 

### Sprite Interactions

1. Go to `MainMenuScene.swift` and add the following code. This piece of code finds the sprite nodes of the position where use touches. If we find that user touched to the `StartGame` node, then we will transition to the `GameScene`.
```
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {

    guard let touch = touches.first else {return}
    let positionInScene = touch.location(in: self)
    let touchedNodes = self.nodes(at: positionInScene)

    if let name = touchedNodes.last?.name {
        if name == "StartGame" {
            let transition = SKTransition.crossFade(withDuration: 0.9)

            guard let sceneView = self.view as? ARSKView else {return}

            if let gameScene = GameScene(fileNamed: "GameScene") {
                sceneView.presentScene(gameScene, transition: transition)
            }
        }
    }


}
```

### SKActions Setup

1. Now it is time to define our sprite programmatically, so that we can place them to the random location in real world. Go to `Bird.swift` file and add the following code.

```
var mainSprite = SKSpriteNode()
    
func setup(){

    mainSprite = SKSpriteNode(imageNamed: "bird1")
    self.addChild(mainSprite)

    let textureAtals = SKTextureAtlas(named: "bird")
    let frames = ["sprite_0", "sprite_1", "sprite_2", "sprite_3", "sprite_4", "sprite_5", "sprite_6"].map{textureAtals.textureNamed($0)}

    let atlasAnimation = SKAction.animate(with: frames, timePerFrame: 1/7, resize: true, restore: false)

    let animationAction = SKAction.repeatForever(atlasAnimation)
    mainSprite.run(animationAction)


    let left = GKRandomSource.sharedRandom().nextBool()
    if left {
        mainSprite.xScale = -1
    }

    let duration = randomNumber(lowerBound: 15, upperBound: 20)

    let fade = SKAction.fadeOut(withDuration: TimeInterval(duration))
    let removeBird = SKAction.run {
        // create a new bird
        self.removeFromParent()
    }

    let flySeqence = SKAction.sequence([fade, removeBird])

    mainSprite.run(flySeqence)

}
```





