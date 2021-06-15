# Issues in SwiftUI

## ARC counting

code below as a workaround for viewModel signals's subscribtions would be released unexpectly under 14.5.

If *createLoginController()* method was implemented in a custom UIHostingController, the event flow will auto released under iOS 14.5.

```swift
    override func prepareTransition(for route: LoginGuideRoute) -> NavigationTransition {
        switch route {
        case .home:
            return .none()

        case .register:
            let controller = RegisterGuideController()
            return .push(controller)

        case .login:
            let controller = createLoginController()
            return .push(controller)

        case .root:
            appRouter.trigger(.loginSucceed)
            return .none()
        }
    }

    private func createLoginController() -> UIViewController {
        let viewModel = LoginViewModel()
        let view = LoginView(viewModel: viewModel)
        let parent = BaseHostingController(rootView: view)

        // as a workaround for viewModel signals's subscribtions would be released unexpectly under 14.5
        viewModel.loginSignal
            .receive(on: DispatchQueue.main)
            .sink { [weak self]  in
                self?.unownedRouter.trigger(.root)
            }
            .store(in: &cancelableSet)

        viewModel.validateSignal
            .receive(on: DispatchQueue.main)
            .flatMap { [weak parent] phoneNumber -> AnyPublisher<PhoneTuple, Never> in
                guard let parent = parent else {
                    return Just((phoneNumber: "", smsCode: ""))
                        .eraseToAnyPublisher()
                }
                return Future<PhoneTuple, Never> { [weak parent] promise in
                    guard let parent = parent else { return }
                    let viewModel = SmsCodeFillViewModel(phoneNumber: phoneNumber)
                    let view = SmsCodeValidateView(viewModel: viewModel)
                    let controller = PopupHostingController(rootView: view)
                    viewModel.actionSignal
                        .receive(on: DispatchQueue.main)
                        .sink(receiveValue: { [weak controller] value in
                            switch value {
                            case .cancel: controller?.dismiss(animated: true, completion: nil)

                            case let .confirm(smsCode, phone):
                                controller?.dismiss(animated: true, completion: {
                                    promise(.success((phone, smsCode)))
                                })
                            }
                        })
                        .store(in: &controller.cancelableSet)
                    controller.presented(by: parent)
                }
                .eraseToAnyPublisher()
            }
            .sink { [weak viewModel] phoneNumber, smsCode in
                viewModel?.loginWith(phone: phoneNumber, smsCode: smsCode)
            }
            .store(in: &cancelableSet)

        return parent
    }
```

## Memory Leask in SwiftUI

when both active `onTapGesture` and `matchedGeometryEffect` modifiers, data will not call deinit method

```swift
import SwiftUI

struct ContentView: View {
    @Namespace private var gridNameSpace

    @StateObject private var data = DataModel()
    let c = GridItem(.adaptive(minimum: 300, maximum: 400), spacing: 20)
    var body: some View {
        ScrollView {
            LazyVGrid(columns: [c], spacing: 20) {
                ForEach(data.items) { item in
                    AnimalImage(image: item.image, favorite: item.isFavorite)
                        .onTapGesture(count: 2) { toggleFavoriteStatus(item) }
                        .onTapGesture(count: 1) { openModal(item, fromGrid: true) }
                        .matchedGeometryEffect(id: item.id, in: gridNameSpace, isSource: true)
                }
            }
        }
    }

    func openModal(_ item: ItemData, fromGrid: Bool) {
//        withAnimation(.basic) {
//            // do something
//        }
    }

    func toggleFavoriteStatus(_ item: ItemData) {

//        if let idx = data.favorites.firstIndex(where: { $0.id == item.id }) {
//            item.isFavorite = false
//            withAnimation(.basic) {
//                // do something
//            }
//
//        } else {
//            withAnimation(.basic) {
//                // do something
//            }
//        }
    }
}
```