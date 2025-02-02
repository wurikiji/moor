# moor_ffi

Experimental Dart bindings to sqlite by using `dart:ffi`. This library contains utils to make
integration with [moor](https://pub.dev/packages/moor) easier, but it can also be used
as a standalone package. It also doesn't depend on Flutter, so it can be used on Dart VM
applications as well.

## Warnings
At the moment, `dart:ffi` is in preview and there will be breaking changes that this
library has to adapt to. This library has been tested on Dart `2.5.0`.
If you're using a development Dart version (this can include any Flutter channels that
are not `stable`), this library might not work.

Also, please don't use this library on production apps yet.

## Supported platforms
You can make this library work on any platform that lets you obtain a `DynamicLibrary`
in which sqlite's symbols are available (see below).

Out of the box, this library supports all platforms where `sqlite3` is installed:
- iOS: Yes 
- macOS: Yes
- Linux: Available on most distros
- Windows: Additional setup is required
- Android: Yes when used with Flutter

This library works with and without Flutter. 
If you're using Flutter, this library will bundle `sqlite3` in your Android app. This 
requires the Android NDK to be installed (You can get the NDK in the [SDK Manager](https://developer.android.com/studio/intro/update.html#sdk-manager)
of Android Studio). Note that the first `flutter run` is going to take a very long time as
we need to compile sqlite.

### On other platforms
Using this library on platforms that are not supported out of the box is fairly 
straightforward. For instance, if you release your own `sqlite3.so` next to your application,
you could use
```dart
import 'dart:ffi';
import 'dart:io';
import 'package:moor_ffi/database.dart';
import 'package:moor_ffi/open_helper.dart';

void main() {
  open.overrideFor(OperatingSystem.linux, _openOnLinux);

  final db = Database.memory();
  db.close();
}

DynamicLibrary _openOnLinux() {
  final script = File(Platform.script.toFilePath());
  final libraryNextToScript = File('${script.path}/sqlite3.so');
  return DynamicLibrary.open(libraryNextToScript.path);
}
```
Just be sure to first override the behavior and then open the database. Further,
if you want to use the isolate api, you can only use a static method or a top-level
function to open the library. For Windows, a similar setup with a `sqlite3.dll` library
should work.

### Supported datatypes
This library supports `null`, `int`, `double`, `String` and `Uint8List` to bind args.
Returned columns from select statements will have the same types.

## Using without moor
```dart
import 'package:moor_ffi/database.dart';

void main() {
  final database = Database.memory();
  // run some database operations. See the example for details
  database.close();
}
```

You can also use an asynchronous API on a background isolate by using `IsolateDb.openFile`
or `IsolateDb.openMemory`, respectively. Be aware that the asynchronous API is much slower,
but it moves work out of the UI isolate.

Be sure to __always__ call `Database.close` to avoid memory leaks!

## Using with moor
Add both `moor` and `moor_ffi` to your pubspec, the `moor_flutter` dependency can be dropped.

```yaml
dependencies:
  moor: ^1.7.0
  moor_ffi: ^0.0.1
dev_dependencies:
  moor_generator: ^1.7.0
```

In the file where you created a `FlutterQueryExecutor`, replace the `moor_flutter` import
with both `package:moor/moor.dart` and `package:moor_ffi/moor_ffi.dart`.
In all other project files that use moor apis (e.g. a `Value` class for companions), just import `package:moor/moor.dart`.

Finally, replace usages of `FlutterQueryExecutor` with `VmDatabase`.

Note that, at the moment, there is no direct counterpart for `FlutterQueryExecutor.inDatabasePath`.