This is a dummy project demonstrating a possible issue in [gradle-xcodePlugin](https://github.com/openbakery/gradle-xcodePlugin).

:warning: Note that the [`settings.gradle`](settings.gradle) in this branch creates a composite build with
the gradle-xcodePlugin residing in a sibling directory to this project. This can be updated or disabled
by editing `settings.gradle`.

## The Issue

My theory is that the plugin is not supporting running `xcodebuild` for projects located in a subfolder rather than in the current directory.

### Example

Running `gradle xcodebuild` works in this case:

```
MyApp/
  |
  |_ MyApp.xcodeproj
```

But doesn't work in this:

```
MyProject/
  |
  |_ MyApp/
    |
    |_ MyApp.xcodeproj
```

To reproduce clone the repo and run `gradle xcodebuild` from its root, you should get an output similar to the one in `gradle_output.txt`.

The relevant error message being:

```
xcodebuild: error: The directory /Users/me/path/to/repo/GradleXcodeIssueDemo does not contain an Xcode project or workspace.
```

Note that setting the value of `projectFile` doesn't solve the issue. _See this project's `build.gradle`._

### Proposed Solution

`xcodebuild` itself has a `-project` option to specify the location of the `xcodeproj` folder.

The plugin should set the `-project` option for `xcodebuild` using the value of the `projectFile` configuration, if any is set.

From what I can understand the proper place where to do it is in the [`build()` task action of `XcodeBuildTask`](https://github.com/openbakery/gradle-xcodePlugin/blob/59906f72aaebb1082a7e218f60a12d42a617bb66/plugin/src/main/groovy/org/openbakery/XcodeBuildTask.groovy#L39).

To verify that using the `-project` option works run the `successful_xcodebuild` executable. 
