# NotionSwift

Unofficial [Notion](https://www.notion.so) SDK for iOS & macOS. 

This is still work in progress version, the module interface might change.

## API Documentation

This library is a client SDK for the official Notion API. 
For more details and documentation please check [Notion Developer Portal](https://developers.notion.com/)

## Supported Endpoints

### Databases
 * Retrieve a database ✅
 * Query a database ✅
 * List databases ✅
 
### Pages
* Retrieve a page ✅
* Create a page ✅
* Update page properties ✅

### Blocks 
* Retrieve block children ✅
* Append block children ✅

### Users
* Retrieve a user ✅
* List all users ✅

### Search 
* Search ✅


## Installation

### CocoaPods

```ruby
pod 'NotionSwift', :git => 'https://github.com/chojnac/NotionSwift.git'
```

### Swift Package Manager

```swift
dependencies: [
    .package(url: "https://github.com/chojnac/NotionSwift.git", .upToNextMajor(from: "0.1.0"))
]
```

## Usage

Currently, this library supports only the "internal integration" authorization mode. For more information about authorization and 
instruction how to obtain `NOTION_TOKEN` please check [Notion Offical Documentation](https://developers.notion.com/docs/authorization).

### Creating a Notion client.

```swift

let notion = NotionClient(accessKeyProvider: StringAccessKeyProvider(accessKey: "{NOTION_TOKEN}"))

```

### List all databases

```swift
// fetch avaiable databases
notion.databaseList {
    print($0)
}
```

### Query a database

In this example we will get all pages in the database. To narrow results use `params` argument.
```swift
let databaseId = Database.Identifier("{DATABASE UUIDv4}")

notion.databaseQuery(databaseId: databaseId) {
    print($0)
}
```

### Retrieve a database

```swift
let databaseId = Database.Identifier("{DATABASE UUIDv4}")

notion.database(databaseId: databaseId) {
    print($0)
}
```

### Retrieve a page

Retrieve page properties. 

```swift
let pageId = Block.Identifier("{PAGE UUIDv4}")

notion.page(pageId: pageId) {
    print($0)
}
```

Page content (text for example) is represented as an array of blocks. The example below loads properties and page content. 

```swift
let pageId = Block.Identifier("{PAGE UUIDv4}")

notion.page(pageId: pageId) { [notion] in
    print("---- Properties ----- ")
    print($0)
    switch $0 {
    case .success(let page):
        notion.blockChildren(blockId: page.id.toBlockIdentifier) {
            print("---- Children ----- ")
            print($0)
        }
    default:
        break
    }
}
```
**Note:** The API returns only the direct children of the page. If there is content nested in the block (nested lists for example) it requires other calls. 

### Create a page

```swift
let parentPageId = Block.Identifier("{PAGE UUIDv4}")

let request = PageCreateRequest(
    parent: .page(parentPageId),
    properties: [
        "title": .init(
            type: .title([
                .init(string: "Lorem ipsum \(Date())")
            ])
        )
    ],
    children: blocks
)

notion.pageCreate(request: request) {
    print($0)
}
```

### Update page properties

```swift
let pageId = Block.Identifier("{PAGE UUIDv4}")

// update title property
let request = PageProperiesUpdateRequest(
    properties: [
        .name("title"): .init(
            type: .title([
                .init(string: "Updated at: \(Date())")
            ])
        )
    ]
)

notion.pageUpdateProperties(pageId: pageId, request: request) {
    print($0)
}
```

### Retrieve block children

Note: This endpoint returns only the first level of children, so for example, nested list items won't be returned. In that case, you need to make another request with the block id of the parent block.

```swift

let pageId = Block.Identifier("{PAGE UUIDv4}")

notion.blockChildren(blockId: pageId) {
    print($0)
}

```

### Append block children

```swift
let pageId = Block.Identifier("{PAGE UUIDv4}")

// append paragraph with styled text to a page.
let blocks: [WriteBlock] = [
    .init(type: .paragraph(.init(text: [
        .init(string: "Lorem ipsum dolor sit amet, "),
        .init(string: "consectetur", annotations: .bold),
        .init(string: " adipiscing elit.")
    ])))
]
notion.blockAppend(blockId: pageId, children: blocks) {
    print($0)
}
```

### Retrieve a user

```swift
let id = User.Identifier("{USER UUIDv4}")
notion.user(userId: id) {
    print($0)
}
```

### List all users
```swift
notion.usersList() {
    print($0)
}
```

### Search

Search for pages & databases with a title containing text "Lorem"
```swift
notion.search(
    request: .init(
        query: "Lorem"
    )
) {
    print($0)
}
```

Search for all databases and ignore pages.
```swift
notion.search(
    request: .init(
        filter: .database
    )
) {
    print($0)
}
```

Get all pages & databases
```swift
notion.search() {
    print($0)
}
```

### Logging and debugging

`NotionSwift` provide an internal rudimental logging system to track HTTP traffic. 
To enable it you need to set a build-in or custom logger handler and decide about log level (`.info` by default).
With `.track` log level you can see all content of a request. This is useful to track mapping issues between library data models and API.


Example logging configuration:
```swift
// This code should be in the ApplicationDelegate

NotionSwift.Environment.logHandler = NotionSwift.PrintLogHandler() // uses print command
NotionSwift.Environment.logLevel = .trace // show me everything

```

**Important**
Integrations are granted access to resources (pages and databases) which users have shared with the integration. Once the integration has been added to a workspace by an Admin, users see the integration within `Share` menus inside Notion.

## License

**NotionSwift** is available under the MIT license. See the [LICENSE](https://github.com/chojnac/NotionSwift/blob/master/LICENSE) file for more info.