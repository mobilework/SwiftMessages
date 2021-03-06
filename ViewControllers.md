# View Controllers

[`SwiftMessagesSegue`](./SwiftMessages/SwiftMessagesSegue.swift) is a configurable subclass of `UIStoryboardSegue` that presents and dismisses modal view controllers by acting as the presenting view controller's `transitioningDelegate` and utilizing SwiftMessages' `show()` and `hide()` function on the destination view controller's view.

## Usage

### Interface Builder

First, create a segue by control-dragging from the sender element to the destination view controller. Then select "swift messages" (or the name of a `SwiftMessagesSegue` subclass) in the segue type prompt. In the image below, we've created a segue using the `VeryNiceSegue` subclass by selecting "very nice" from the prompt.

<p align="center">
  <img src="./Design/SwiftMessagesSegueCreate.png" />
</p>

### Programatic

`SwiftMessagesSegue` can be used without an associated storyboard or segue by doing the following in the presenting view controller.

````swift
let destinationVC = ... // make a reference to a destination view controller
let segue = SwiftMessagesSegue(identifier: nil, source: self, destination: destinationVC)
... // do any configuration here
segue.perform()
````

To dismiss, call the UIKit API on the presenting view controller:

````swift
dismiss(animated: true, completion: nil)
````

It is not necessary to retain `segue` because it retains itself until dismissal. However, you can retain it if you plan to `perform()` more than once.

### Configuration

`SwiftMessagesSegue` generally requires configuration to achieve specific layouts and optional behaviors. There are a few good ways to do this:

1. __(Recommended)__ Subclass `SwiftMessagesSegue` and apply configurations in `init(identifier:source:destination:)`. Subclasses will automatically appear in the segue type dialog using an auto-generated name. For example, the name for "VeryNiceSegue" would be "very nice".


    ```swift
    class VeryNiceSegue: SwiftMessagesSegue {
        override public  init(identifier: String?, source: UIViewController, destination: UIViewController) {
            super.init(identifier: identifier, source: source, destination: destination)
            configure(layout: .bottomCard)
            dimMode = .blur(style: .dark, alpha: 0.9, interactive: true)
            messageView.configureNoDropShadow()
        }
    }
    ```


1. Apply configurations in `prepare(for:sender:)` of the presenting view controller after down-casting the segue to `SwiftMessagesSegue`.

    ````swift
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if let segue = segue as? SwiftMessagesSegue {
            segue.configure(layout: .bottomCard)
            segue.dimMode = .blur(style: .dark, alpha: 0.9, interactive: true)
            segue.messageView.configureNoDropShadow()
        }
    }
    ````

The `configure(layout:)` method is a shortcut for configuring some basic layout and animation options that roughly mirror the options in `SwiftMessages.Layout`.

````swift
// Configure a bottom card-style presentation
segue.configure(layout: .bottomCard)
````

Many more styles can be achieved by customizing the underlying options. Some of the `SwiftMessages.Config` options are useful for view controller presentation and are mirrored on `SwiftMessagesSegue`.

````swift
// Turn off interactive dismiss
segue.interactiveHide = false

// Enable dimmed background with tap-to-dismiss
segue.dimMode = .gray(interactive: true)

// Specify the animation and positioning
segue.presentationStyle = .bottom
````

The `messageView` property provides access to an instance of [`BaseView`](./SwiftMessages/BaseView.swift), the superclass of [`MessageView`](./SwiftMessages/MessageView.swift), that serves as the view presented by SwiftMessages. The view controller's view is contained as a descendant of this view. There are some useful options available on `messageView`:

````swift
// Increase the internal layout margins. With the `.background` containment option,
// the margin additions specify the outer margins around `messageView.backgroundView`.
segue.messageView.layoutMarginAdditions = UIEdgeInsets(top: 20, left: 20, bottom: 20, right: 20)

// Collapse layout margin edges that encroach on non-zero safe area insets.
messageView.collapseLayoutMarginAdditions = true

// Add a default drop shadow.
segue.messageView.configureDropShadow()

// Indicate that the view controller's view should be installed
// as the `backgroundView` of `messageView`.
segue.containment = .background
````

The view controller's view is a direct subview of `containerView`, an instance of [`ViewControllerContainerView`](./SwiftMessages/ViewControllerContainerView.swift), which provides corner rounding options.

````swift
// Change the corner radius
segue.containerView.cornerRadius = 20
````

### Height

SwiftMessages relies on Auto Layout to determine the height the view controller's view.
    However, some view controllers, such as `UINavigationController` have zero Auto Layout height.
    There are a few ways to specify the height of these view controllers:
1. Add an explicit height constraint to the view controller's view.
2. Set the view controller's `preferredContentSize` ("Use Preferred Explicit Size" in Interface Builder's
   attribute inspector). Note that `preferredContentSize.width` is ignored and can be set to zero.
3. Set `SwiftMessagesSegue.messageView.backgroundHeight`.

See [`SwiftMessagesSegue`](./SwiftMessages/SwiftMessagesSegue.swift) for additional documentation and technical details.
