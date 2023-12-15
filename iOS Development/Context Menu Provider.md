`Context Menu` is part of `UIKit` framework.
Here we define a custom class manager called `ContextMenuInteractionProvider` where we define all needed stuffs for our context menu to work along with `Drag & Drop` where it is known to cause issues with both drag & drop and context menu implemented using UITableView or UICollectionView's native context menu support.

```swift
import UIKit

class ContextMenuInteractionProvider: NSObject {
    private let identifier: NSCopying?
    private let previewProvider: UIContextMenuContentPreviewProvider?
    private let menu: MenuStub
    private var actions: [UIAction] = []

    init(
        identifier: NSCopying? = NSString(string: UUID().uuidString),
        previewProvider: UIContextMenuContentPreviewProvider? = nil,
        menu: MenuStub = .stub
    ) {
        self.identifier = identifier
        self.previewProvider = previewProvider
        self.menu = menu
    }

    func attachInteraction(to view: UIView) {
        view.addInteraction(UIContextMenuInteraction(delegate: self))
    }

    func removeInteraction(from view: UIView) {
        for interaction in view.interactions where interaction.isKind(of: UIContextMenuInteraction.self) {
            if let contextMenu = interaction as? UIContextMenuInteraction,
               let delegate = contextMenu.delegate,
               delegate.isKind(of: ContextMenuInteractionProvider.self) {
                view.removeInteraction(contextMenu)
            }
        }
    }

    func setContextMenuOptions(with actions: [UIAction]) {
        self.actions = actions
    }
}

extension ContextMenuInteractionProvider {
    struct MenuStub {
        let title: String
        let subtitle: String?
        let image: UIImage?
        let identifier: UIMenu.Identifier?
        let options: UIMenu.Options
        let preferredElementSize: UIMenu.ElementSize

        static var stub: MenuStub {
            .init(
                title: "",
                subtitle: nil,
                image: nil,
                identifier: nil,
                options: [],
                preferredElementSize: {
                    if #available(iOS 17.0, tvOS 17.0, *) { .automatic }
                    else { .large }
                }())
        }
    }
}

extension ContextMenuInteractionProvider: UIContextMenuInteractionDelegate {
    func contextMenuInteraction(
        _ interaction: UIContextMenuInteraction,
        configurationForMenuAtLocation location: CGPoint
    ) -> UIContextMenuConfiguration? {
        UIContextMenuConfiguration(
            identifier: identifier,
            previewProvider: previewProvider
        ) { [weak self] _ in
            guard let self else { return nil }
            return UIMenu(
                title: self.menu.title,
                subtitle: self.menu.subtitle,
                image: self.menu.image,
                identifier: self.menu.identifier,
                options: self.menu.options,
                preferredElementSize: self.menu.preferredElementSize,
                children: self.actions)
        }
    }
}
```

## Usage:

Here we initialized our context provider
```swift
let targetContextView = UIView()

let provider = ContextMenuInteractionProvider()
```

Here is how we attach our custom interaction for our context menu.
```swift
provider.attachInteraction(to: targetContextView)
```

Here is how we add the context menu's options available to our users
```swift
provider.setContextMenuOptions(with: []) // options == [UIAction]
```

And, here is how we remove our custom interaction added to specific view
```swift
provider.removeInteraction(from: targetContextView)
```