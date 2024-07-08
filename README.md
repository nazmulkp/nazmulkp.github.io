

```
//
//  TestWebView.swift
//  ProjectSwiftUI
//
//  Created by MacBook Air M1 on 24/2/22.
//

import SwiftUI

struct WebLoadView: View {
    
    @State var word: String
    @State private var shouldRefresh = false
    
    @Binding var incomePresented: Bool
    
    @State var workState = WebViewManager.WorkState.initial
    @State private var countdown = 10
    @State private var isButtonActive = false
    let timer = Timer.publish(every: 1, on: .main, in: .common).autoconnect()
    
    var body: some View {
        VStack {
            HStack {
                Button(action: {
                    withAnimation {
                        incomePresented = false
                    }
                }) {
                    Image(systemName: "xmark")
                        .font(.system(size: 20, weight: .bold))
                        .foregroundColor(.black)
                }
                
                Spacer()
                
                Text("======")
                
                Spacer()
                
                Button(action: {
                    withAnimation {
                        self.shouldRefresh = true
                    }
                }) {
                    Image(systemName: "goforward")
                        .font(.system(size: 20, weight: .bold))
                        .foregroundColor(.black)
                }
                .padding(.bottom, 3)
            }
            .zIndex(100)
            .padding(.horizontal)
            
            ZStack {
                if workState == .loading {
                    Text("Loading...")
                        .foregroundColor(.secondary)
                }
                WebViewManager(query: word, reload: $shouldRefresh, workState: $workState)
            }
        }
        .onReceive(timer) { _ in
            if countdown > 0 {
                countdown -= 1
            } else {
                isButtonActive = true
            }
        }
    }
}
```

**Notes:**

**Timer Functionality:** I added an .onReceive modifier to handle the timer countdown. Make sure the countdown and button activation logic suits my use case.

**Loading State:** The ZStack now conditionally displays a loading message based on workState. Ensure WebViewManager.WorkState has a .loading case and that WebViewManager updates workState appropriately.

**Cleaner Button Actions:** Simplified the button action closures for readability.


I implementation of **WebViewManager** correctly uses the coordinator pattern. This pattern is essential for managing delegates and handlers for **UIKit** components within **SwiftUI**.

Here's a recap of how the coordinator pattern is used in my code and why it's necessary:

Coordinator Pattern in my Code
Coordinator Class:

I have defined a Coordinator class within **WebViewManager**. This class conforms to **NSObject** and **WKNavigationDelegate**, which are necessary for handling web view navigation events.
The coordinator class has a reference to its parent **WebViewManager** to update the state variables (workState).
**makeCoordinator()** Method:

This method creates an instance of the Coordinator class and returns it. This is necessary to instantiate the coordinator.
**makeUIView(context:)** Method:

This method creates and configures the WKWebView instance.
It sets the navigationDelegate of the web view to the coordinator, so navigation events are forwarded to the coordinator.
**updateUIView(_:context:)** Method:

This method updates the web view based on the state changes. For example, it loads a URL when the state is .initial.
Handling Navigation Events:

The coordinator handles web view navigation events **(didStartProvisionalNavigation, didFail, didFinish)** and updates the workState accordingly.
Here is a slightly cleaned-up version for clarity:

```
import SwiftUI
import WebKit

struct WebViewManager: UIViewRepresentable {
    enum WorkState: String {
        case initial
        case done
        case working
        case errorOccurred
    }
    
    let query: String
    @Binding var reload: Bool
    @Binding var workState: WorkState
    
    private let webview = WKWebView()
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    func makeUIView(context: UIViewRepresentableContext<WebViewManager>) -> WKWebView {
        let webView = WKWebView()
        webView.navigationDelegate = context.coordinator
        return webView
    }
    
    func updateUIView(_ uiView: WKWebView, context: UIViewRepresentableContext<WebViewManager>) {
        switch self.workState {
        case .initial:
            if let url = URL(string: self.query) {
                uiView.load(URLRequest(url: url))
            }
        default:
            break
        }
    }
    
    class Coordinator: NSObject, WKNavigationDelegate {
        var parent: WebViewManager
        
        init(_ parent: WebViewManager) {
            self.parent = parent
        }
        
        func webView(_ webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation!) {
            self.parent.workState = .working
        }
        
        func webView(_ webView: WKWebView, didFail navigation: WKNavigation!, withError error: Error) {
            self.parent.workState = .errorOccurred
        }
        
        func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
            self.parent.workState = .done
        }
    }
}
```
**Summary**

My **WebViewManager** uses the coordinator pattern effectively to manage web view navigation events and update the **SwiftUI** view state. This pattern is necessary for integrating **UIKit** components that require delegate methods into **SwiftUI**, ensuring smooth interoperability and state management.

