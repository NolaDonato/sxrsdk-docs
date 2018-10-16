##Overview

Now that you've learnt how to use 3D model and animation with the SXR SDK. We are going to learn how to use controllers to make VR app more interactive.

##Create Project
Create an SXR project by copying the [template project](https://github.com/sxrsdk/sxrsdk-demos/tree/master/template/SXRApplication) 

Perform the following steps to make sure your project runs correctly

1. (if developing for Gear VR) Copy your [Oculus signature file](https://developer.oculus.com/osig/) to `app/src/main/assets` folder.
1. Change the `applicationId` in `build.gradle` to a unique name to avoid naming conflict when you test the app later
1. Change the `app_name` in `res/values/strings.xml` to avoid confusion when you debug the app.

##Intro

Being able to interact with in the VR environment helps a lot with the immersion. Currently, there are two ways to interact with VR content

1. GearVR controller

    Gear VR controller provides 3 degrees of freedom orientation tracking and trackpad control, it's highly recommended for a more immersive VR experience
    ![](/images/gear_vr_controller.jpg)

2. Gaze controller

    Gaze controller is available by default from GearVR headset, based on the direction of the headset and touch input on the touchpad
    ![](/images/gear_vr_headset_sm.jpg)

!!!note
    The SXR SDK supports both controllers and GearVR controller have higher priority over gaze controller. It will automatically switch when user turn controller on/off.

## Using VR Controller

For VR controller to work correctly, we need to implement these key elements

1. Cursor
2. Collider
3. PickHandler
4. Controller

### 1. Create Cursor
Cursor indicates the location and object that user is pointing. It's a important way for user to interact with the VR environment.

A cursor can be anything, an image, a cube or even something that's animated. Developer can pick anything that fits their needs. 

In this tutorial we'll use a simple reticle texture as an example.

![](/images/cursor.png)

First let's create a quad to display this texture, notice we turn off the depth test and set the rendering order as `OVERLAY` make it always visible to the user.

```java
    SXRTexture cursor_texture = sxrContext.getAssetLoader().loadTexture(new SXRAndroidResource(sxrContext, "cursor.png"));
    final SXRSceneObject cursor = new SXRSceneObject(sxrContext, sxrContext.createQuad(1f, 1f), cursor_texture);
    cursor.getRenderData().setDepthTest(false);
    cursor.getRenderData().setRenderingOrder(SXRRenderData.SXRRenderingOrder.OVERLAY);
```


### 2. Collider

In order to know which object dose the user picked in the scene, we need to add collider to the SceneObject.

There are three types of colliders

1. BoxCollider

    BoxCollider can detect picking within a bounding box

2. SphereCollider

    SphereCollider can detect picking within a sphere

3. MeshCollider

    MeshCollider can detect picking of a complex 3d Model, however it's slower than the previous two.

Collider can be added to any SceneObject as a component
```java
    sceneObject.attachComponent(new SXRMeshCollider(getSXRContext(), false));
```

You can use the following function to create cubes with collider
```java
    private SXRSceneObject createCube()
    {
        SXRMaterial material = new SXRMaterial(getSXRContext(), SXRMaterial.SXRShaderType.Color.ID);
        material.setColor(Color.GRAY);

        SXRCubeSceneObject cube = new SXRCubeSceneObject(getSXRContext());
        cube.getRenderData().setMaterial(material);

        cube.attachComponent(new SXRMeshCollider(getSXRContext(), false));

        return cube;
    }
```

### 3. PickHandler

PickHandler will be triggered once the cursor enter/exit a SceneObject with collider, base on the events developer can implement different feedback for the user

PickHandler have 4 major methods, onEnter/onExit/onTouchStart/onTouchEnd

Here is an example of how to use each method

```java
    private ITouchEvents mPickHandler = new SXREventListeners.TouchEvents()
    {
        private SXRSceneObject movingObject;

        public void onEnter(SXRSceneObject sceneObj, SXRPicker.SXRPickedObject pickInfo)
        {
            sceneObj.getRenderData().getMaterial().setColor(Color.RED);
        }

        public void onTouchStart(SXRSceneObject sceneObj, SXRPicker.SXRPickedObject pickInfo)
        {
            if (movingObject == null)
            {
                sceneObj.getRenderData().getMaterial().setColor(Color.BLUE);
                if (mController.startDrag(sceneObj))
                {
                    movingObject = sceneObj;
                }
            }
        }

        public void onTouchEnd(SXRSceneObject sceneObj, SXRPicker.SXRPickedObject pickInfo)
        {
            sceneObj.getRenderData().getMaterial().setColor(Color.RED);
            if (sceneObj == movingObject)
            {
                mController.stopDrag();
                movingObject = null;
            }
        }

        public void onExit(SXRSceneObject sceneObj, SXRPicker.SXRPickedObject pickInfo)
        {
            sceneObj.getRenderData().getMaterial().setColor(Color.GRAY);
            if (sceneObj == movingObject)
            {
                mController.stopDrag();
                movingObject = null;
            }
        }
    };
```

### 4. Enable Controller

With cursor, collider and pickhandler, let's enable the controller.

Everytime a new controller connects to Gear VR it will trigger the `onCursorControllerSelected` event. It's a great place to initialize the controller

Initialize the controller using following code
```java
    //Initialize controller
    sxrContext.getInputManager().selectController(new SXRInputManager.ICursorControllerSelectListener()
    {
        public void onCursorControllerSelected(SXRCursorController newController, SXRCursorController oldController)
        {
            if (oldController != null)
            {
                oldController.removePickEventListener(mPickHandler);
            }
            mController = newController;
            newController.addPickEventListener(mPickHandler);
            newController.setCursor(cursor);
            newController.setCursorDepth(DEPTH);
            newController.setCursorControl(SXRCursorController.CursorControl.PROJECT_CURSOR_ON_SURFACE);
        }
    });
```

### 5. Build
Build and run the project, if you have a Gear VR controller, you should be able to see a Gear VR controller in VR that mirrors your move, otherwise you should be able to see a cursor on the center of the screen



## Source Code
Complete [Source Code](https://github.com/sxrsdk/sxrsdk-demos/tree/master/tutorials/tutorial_5_controller) for this sample
