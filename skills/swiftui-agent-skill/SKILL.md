---
name: swiftui-agent-skill
description: Agent skill that helps AI coding assistants write smarter, modern SwiftUI code with best practices for API usage, design, performance, and accessibility
triggers:
  - "help me write better SwiftUI code"
  - "review this SwiftUI code for best practices"
  - "check for SwiftUI deprecated API"
  - "improve SwiftUI accessibility"
  - "optimize SwiftUI performance"
  - "use SwiftUI agent skill"
  - "check SwiftUI navigation patterns"
  - "review SwiftUI state management"
---

# SwiftUI Agent Skill

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

An agent skill that helps AI coding assistants write smarter, simpler, and more modern SwiftUI code. Provides guidance on API usage, design patterns, performance optimization, and accessibility. Covers navigation, layout, animations, state management, VoiceOver, deprecated APIs, and common LLM mistakes.

## Installation

Install via npx:

```bash
npx skills add https://github.com/twostraws/swiftui-agent-skill --skill swiftui-pro
```

Or via Claude Code marketplace:

```
/plugin marketplace add twostraws/SwiftUI-Agent-Skill
/plugin install swiftui-pro@swiftui-agent-skill
```

## Usage

### In Claude Code

```
/swiftui-pro
```

With specific focus:

```
/swiftui-pro Check for deprecated API
/swiftui-pro Focus on accessibility
/swiftui-pro Review navigation patterns
```

### In Codex

```
$swiftui-pro
```

With specific instructions:

```
$swiftui-pro Look for performance issues
$swiftui-pro Check VoiceOver support
```

### Natural Language

```
Use the SwiftUI Pro skill to review this view for best practices
Check this SwiftUI code with the agent skill
```

## Key SwiftUI Best Practices

### Modern Navigation

**Use NavigationStack and NavigationPath** (iOS 16+), not deprecated NavigationView:

```swift
struct ContentView: View {
    @State private var path = NavigationPath()
    
    var body: some View {
        NavigationStack(path: $path) {
            List {
                NavigationLink("Detail", value: "detail")
            }
            .navigationDestination(for: String.self) { value in
                DetailView(item: value)
            }
        }
    }
}
```

**Programmatic navigation:**

```swift
struct ContentView: View {
    @State private var path = NavigationPath()
    
    var body: some View {
        NavigationStack(path: $path) {
            Button("Go to Detail") {
                path.append("detail")
            }
            .navigationDestination(for: String.self) { value in
                DetailView(item: value)
            }
        }
    }
}
```

### State Management

**Use @State for view-local state:**

```swift
struct CounterView: View {
    @State private var count = 0
    
    var body: some View {
        Button("Count: \(count)") {
            count += 1
        }
    }
}
```

**Use @Observable for shared state** (iOS 17+), not ObservableObject:

```swift
@Observable
class AppState {
    var username = ""
    var isLoggedIn = false
}

struct ContentView: View {
    let state = AppState()
    
    var body: some View {
        Text("User: \(state.username)")
            .onChange(of: state.isLoggedIn) { oldValue, newValue in
                print("Login state changed")
            }
    }
}
```

**Use @Bindable for bindings to Observable objects:**

```swift
struct ProfileView: View {
    @Bindable var user: User
    
    var body: some View {
        TextField("Name", text: $user.name)
    }
}
```

### Accessibility

**Always provide accessibility labels for custom controls:**

```swift
struct CustomButton: View {
    var body: some View {
        Image(systemName: "star.fill")
            .foregroundStyle(.yellow)
            .onTapGesture {
                // action
            }
            .accessibilityLabel("Favorite")
            .accessibilityAddTraits(.isButton)
    }
}
```

**Use accessibilityElement for grouped content:**

```swift
struct UserCard: View {
    let name: String
    let age: Int
    
    var body: some View {
        VStack {
            Text(name)
            Text("\(age) years old")
        }
        .accessibilityElement(children: .combine)
    }
}
```

**Provide hints for complex interactions:**

```swift
Button("Delete") {
    // delete action
}
.accessibilityHint("Deletes this item permanently")
```

### Layout

**Use modern layout containers:**

```swift
// Grid for structured layouts
Grid(alignment: .leading, horizontalSpacing: 20) {
    GridRow {
        Text("Name:")
        Text("John")
    }
    GridRow {
        Text("Age:")
        Text("30")
    }
}

// ViewThatFits for responsive layouts
ViewThatFits {
    HStack {
        content
    }
    VStack {
        content
    }
}
```

**Avoid nested geometry readers:**

```swift
// BAD
GeometryReader { outer in
    GeometryReader { inner in
        // layout code
    }
}

// GOOD - use .containerRelativeFrame or layout protocol
Color.blue
    .containerRelativeFrame(.horizontal) { width, axis in
        width * 0.5
    }
```

### Performance

**Use lazy containers for large lists:**

```swift
// Always use LazyVStack/LazyHStack for scrolling content
ScrollView {
    LazyVStack {
        ForEach(items) { item in
            ItemView(item: item)
        }
    }
}
```

**Avoid expensive computations in body:**

```swift
struct ContentView: View {
    @State private var items: [Item] = []
    
    // BAD - computed every time body runs
    var body: some View {
        let sortedItems = items.sorted()
        List(sortedItems) { item in
            Text(item.name)
        }
    }
}

// GOOD - cache computed values
struct ContentView: View {
    @State private var items: [Item] = []
    
    private var sortedItems: [Item] {
        items.sorted()
    }
    
    var body: some View {
        List(sortedItems) { item in
            Text(item.name)
        }
    }
}
```

**Use task modifiers instead of onAppear for async work:**

```swift
// GOOD
struct ContentView: View {
    @State private var data: [Item] = []
    
    var body: some View {
        List(data) { item in
            Text(item.name)
        }
        .task {
            await loadData()
        }
    }
    
    func loadData() async {
        // async loading
    }
}
```

### Animations

**Use withAnimation for state-driven animations:**

```swift
struct AnimatedView: View {
    @State private var isExpanded = false
    
    var body: some View {
        VStack {
            if isExpanded {
                Text("Details")
                    .transition(.move(edge: .top))
            }
            
            Button("Toggle") {
                withAnimation(.spring(response: 0.3)) {
                    isExpanded.toggle()
                }
            }
        }
    }
}
```

**Use animation modifier for continuous animations:**

```swift
struct RotatingView: View {
    @State private var rotation = 0.0
    
    var body: some View {
        Image(systemName: "arrow.clockwise")
            .rotationEffect(.degrees(rotation))
            .animation(.linear(duration: 1).repeatForever(autoreverses: false), value: rotation)
            .onAppear {
                rotation = 360
            }
    }
}
```

### Common Deprecated APIs to Avoid

**NavigationView** → Use `NavigationStack`

**sheet(isPresented:onDismiss:content:)** with @State bool is fine, but for complex flows use `NavigationStack`

**GeometryReader for sizing** → Use `containerRelativeFrame` or layout protocol

**ObservableObject** → Use `@Observable` macro (iOS 17+)

**@Published** → Not needed with `@Observable`

**@StateObject** → Use `@State` with `@Observable` objects

**onChange(of:perform:)** → Use `onChange(of:initial:_:)` with old/new values

## Configuration

### Project-wide Installation

Install for all projects:

```bash
npx skills add https://github.com/twostraws/swiftui-agent-skill --skill swiftui-pro
# Select "all projects" during installation
```

### Project-specific Installation

Install for current project only:

```bash
npx skills add https://github.com/twostraws/swiftui-agent-skill --skill swiftui-pro
# Select "this project only" during installation
```

## Common Patterns

### Form with Validation

```swift
@Observable
class FormData {
    var email = ""
    var password = ""
    
    var isValid: Bool {
        !email.isEmpty && password.count >= 8
    }
}

struct LoginForm: View {
    @State private var formData = FormData()
    
    var body: some View {
        Form {
            TextField("Email", text: $formData.email)
                .textContentType(.emailAddress)
                .keyboardType(.emailAddress)
            
            SecureField("Password", text: $formData.password)
                .textContentType(.password)
            
            Button("Login") {
                // login action
            }
            .disabled(!formData.isValid)
        }
    }
}
```

### Environment Injection

```swift
@Observable
class AppSettings {
    var theme = "light"
    var fontSize = 14.0
}

@main
struct MyApp: App {
    @State private var settings = AppSettings()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(settings)
        }
    }
}

struct ContentView: View {
    @Environment(AppSettings.self) private var settings
    
    var body: some View {
        Text("Theme: \(settings.theme)")
    }
}
```

### Custom View Modifiers

```swift
struct CardStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(.background.secondary)
            .clipShape(RoundedRectangle(cornerRadius: 12))
            .shadow(radius: 2)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardStyle())
    }
}

// Usage
Text("Hello")
    .cardStyle()
```

## Troubleshooting

### "Purple warnings" in console

Usually caused by state updates during view updates. Use `Task` or `DispatchQueue.main.async`:

```swift
.onAppear {
    Task {
        await loadData()
    }
}
```

### Views not updating

Ensure Observable objects are properly injected:

```swift
// BAD
struct ChildView: View {
    let settings: AppSettings // Won't observe changes
}

// GOOD
struct ChildView: View {
    @Environment(AppSettings.self) private var settings
}
```

### Navigation state not persisting

Use proper navigation path management:

```swift
@State private var path = NavigationPath()

// Restore path from storage
.task {
    if let savedPath = try? await loadPath() {
        path = savedPath
    }
}
```

### Performance issues with List

Use lazy loading and id-based updates:

```swift
List(items, id: \.id) { item in
    ItemRow(item: item)
}
.id(items.map(\.id)) // Force refresh when needed
```

## Related Skills

- [SwiftData Pro](https://github.com/twostraws/SwiftData-Agent-Skill) - SwiftData best practices
- [Swift Concurrency Pro](https://github.com/twostraws/Swift-Concurrency-Agent-Skill) - Async/await patterns
- [Swift Testing Pro](https://github.com/twostraws/Swift-Testing-Agent-Skill) - Modern Swift testing

## Resources

- [Hacking with Swift](https://www.hackingwithswift.com) - Free SwiftUI tutorials
- [Swift Agent Skills](https://github.com/twostraws/Swift-Agent-Skills) - Collection of Swift agent skills
- [Agent Skills](https://agentskills.io/home) - Agent Skills format documentation
