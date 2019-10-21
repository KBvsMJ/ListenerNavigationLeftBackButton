```

private protocol NavigationBarLeftBackButtonDelegate {
    var shouldPopOnBackButtonPress: Bool { get }
}

extension UIViewController: NavigationBarLeftBackButtonDelegate {

    /// 是否启用监听导航栏左边返回按钮点击事件
    
    @objc open var enableListenerLeftBackButtonEvent: Bool {
        return false
    }

    private struct NavigationLeftBackButtonHandler {
        static var nameForKey: String = "NavigationLeftBackButtonHandler"
    }

    typealias NavigationBarLeftBackButtonClickHandler = () -> Void 
    
    /// 如果需要监听导航栏左边返回按钮事件(备注: 比如点击返回按钮需要做业务逻辑处理) 
    
    /// 则需要重写enableListenerLeftBackButtonEvent属性并设置true。
    
    /// 不需要监听导航栏左边返回按钮事件，则不需要重写enableListenerLeftBackButtonEvent属性 
    
    var leftBackButtonEventHandler: NavigationBarLeftBackButtonClickHandler? {
        get {
            return objc_getAssociatedObject(self, &NavigationLeftBackButtonHandler.nameForKey) as? NavigationBarLeftBackButtonClickHandler
        } set {
            objc_setAssociatedObject(self, &NavigationLeftBackButtonHandler.nameForKey, newValue, .OBJC_ASSOCIATION_COPY_NONATOMIC)
        }
    }

    var shouldPopOnBackButtonPress: Bool {
        if enableListenerLeftBackButtonEvent {
            leftBackButtonEventHandler?()
        }
        return !enableListenerLeftBackButtonEvent
    }
}

extension UINavigationController: UINavigationBarDelegate {
    public func navigationBar(_ navigationBar: UINavigationBar, shouldPop item: UINavigationItem) -> Bool {
        if let barItemCount = navigationBar.items?.count, barItemCount > viewControllers.count {
            return true
        }
        var shouldPop = true
        if let viewController = topViewController {
            shouldPop = viewController.shouldPopOnBackButtonPress
        }
        if shouldPop {
            DispatchQueue.main.async {
                self.popViewController(animated: true)
            }
        } else {
            for subView in navigationBar.subviews where subView.alpha < 1.0 {
                UIView.animate(withDuration: 0.25, animations: {
                    subView.alpha = 1.0
                })
            }
        }
        return false
    }
}







```
