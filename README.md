# danger-swift

Write your Dangerfiles in Swift 4+.

### TODO

Not ready for production use today, does work on our CI though. 
Requires Danger JS 2.x. Rough ETA of production use is ~2 weeks. Currently
blocked [on marathon](https://github.com/JohnSundell/Marathon/pull/127)

### Blockers

 - Ensure marathon installs Danger correctly
 - Ensure the right paths for the Danger lib in the runner

### What it looks like today

You can make a Dangerfile that looks through PR metadata, it's fully typed.

```swift
import Danger

let allSourceFiles = danger.git.modifiedFiles + danger.git.createdFiles

let changelogChanged = allSourceFiles.contains("CHANGELOG.md")
let sourceChanges = allSourceFiles.first(where: { $0.hasPrefix("Sources") })

if !changelogChanged && sourceChanges != nil {
  warn("No CHANGELOG entry added.")
}


// You can use these functions to send feedback:

message("Highlight something in the table")
warn("Something pretty bad, but not important enough to fail the build")
fail("Something that must be changed")

markdown("Free-form markdown that goes under the table, so you can do whatever.")
```

### The ideal flow

In your CI:

```sh
# Setup
npm install -g danger@alpha           # Get DangerJS
brew install marathon-swift           # Install the SwiftPM app installer
marathon install danger/danger-swift  # Install danger-swift locally

# Script
danger process danger-swift           # Run Danger
```

Creating a `Dangerfile.swift` is a bit tricky, because it relies on a library target which isn't available by default. 
Ideally we add a command to create a temporary Danger Xcodeproject with the right settings for editing with docs + tools.

### How it works

This project takes its ideas from how the Swift Package Manager handles package manifests. You can get the [long story here][spm-lr], but the TLDR is that there is a runner which compiles and executes a runtime lib which exports its data out into JSON when the libs process is over.

So this project will export a lib ^ and a CLI tool `danger-swift` which is the runner. It will handle turning the Danger DSL JSON [message from DangerJS][dsl] and passing that into the eval'd `Dangerfile.swift`. When that process is finished it's expected that the Swift `Danger` object would post the results into a place where they can easily be passed back to DangerJS.

### Dev

If you are not using Xcode 9 beta for command-line things, run the following command:

```sh
export DEVELOPER_DIR=/Applications/Xcode-beta.app/Contents/Developer
```

Now that tab of terminal will always be using the Xcode beta. You can skip this if you change the settings in Xcode's prefs.

```sh
git clone https://github.com/danger/danger-swift.git
cd danger-swift
swift build
swift package generate-xcodeproj
open Danger.xcodeproj
```

Then I tend to run it by eval the Dangerfile with:

```sh
swift build && swiftc --driver-mode=swift -L .build/debug -I .build/debug -lDanger Dangerfile.swift fixtures/eidolon_609.json fixtures/response_data.json
```

If you want to emulate how DangerJS's `process` will work entirely, then use:

```sh
swift build && cat fixtures/eidolon_609.json | ./.build/debug/danger-swift
```

### Long-term

I, orta, only plan on bootstrapping this project, as I won't be using this in production. I'm happy to help support others who want to own this idea and really make it shine though! So if you're interested in helping out, make a few PRs and I'll give you org access. 

[m]: https://github.com/JohnSundell/Marathon/issues/59
[spm-lr]: http://bhargavg.com/swift/2016/06/11/how-swiftpm-parses-manifest-file.html
[dsl]: https://github.com/danger/danger-js/pull/341
