#NOTE: Some stream are "network" streams, you CANNOT call "FileData".GetStream().Length. You have to use .DataArray to read everything into memory first!

# FilePicker Plugin for Xamarin.Forms

Simple cross-platform plug-in that allows you to pick files and work with them.

The original project can be found [here](https://github.com/Studyxnet/FilePicker-Plugin-for-Xamarin-and-Windows/), but seems abandoned, this one was forked and further developed.

## Build status
### Stable [![Build status](https://jfversluis.visualstudio.com/FilePicker%20plugin/_apis/build/status/FilePicker%20Plugin)](https://jfversluis.visualstudio.com/FilePicker%20plugin/_build/latest?definitionId=36) [![NuGet version](https://badge.fury.io/nu/Xamarin.Plugin.XFilePicker.svg)](https://badge.fury.io/nu/Xamarin.Plugin.XFilePicker)
 
NuGet: [https://www.nuget.org/packages/Xamarin.Plugin.XFilePicker/](https://www.nuget.org/packages/Xamarin.Plugin.XFilePicker/)
 
### Development feed (possibly instable)

Add this as a source to your IDE to find the latest packages: [https://www.myget.org/F/filepicker-plugin/api/v3/index.json](https://www.myget.org/F/filepicker-plugin/api/v3/index.json)

## Setup

* Install into your Xamarin.Android, Xamarin.iOS, Xamarin.Forms, Xamarin.Mac, Xamarin.WPF project and Client projects.

**Platform Support**

|Platform               |Supported  |Version    |Remarks                                                            |
|:---------------------:|:---------:|:---------:|:-----------------------------------------------------------------:|
|Xamarin.Android        |Yes        |API 8+    |                                                                   |
|Xamarin.iOS            |Yes        |iOS 6+     |                                                                   |
|Xamarin.iOS Unified    |Yes        |iOS 6+     |                                                                   |
|Xamarin.Mac            |Yes        |iOS 10.12+ |Has only been tested on MacOS 10.12                                |
|WPF                    |Yes        |N/A        |Using .NET Framework 4.5                                           |
|Tizen                  |Initial    |4+         |Initial platform dependency support, still need to write the code  |
|WatchOS                |Initial    |1+         |Initial platform dependency support, still need to write the code  |
|TVOS                   |Initial    |1+         |Initial platform dependency support, still need to write the code  |
|Windows 10 UWP         |Yes        |10+        |                                                                   |


### API Usage

Call **XFileManager.Current** from any platform or .NET Standard project to gain access to APIs.

### Example

            try
            {
                FileData fileData = await XFileManager.Current.PickFile();
                if (fileData == null)
                    return; // user canceled file picking

                string fileName = fileData.FileName;
                string contents = System.Text.Encoding.UTF8.GetString(fileData.DataArray);

                System.Console.WriteLine("File name chosen: " + fileName);
                System.Console.WriteLine("File data: " + contents);
            }
            catch (Exception ex)
            {
                System.Console.WriteLine("Exception choosing file: " + ex.ToString());
            }

### Methods

#### `async Task<FileData> PickFile(string[] allowedTypes = null)`

Starts file picking and returns file data for picked file. File types can be
specified in order to limit files that can be selected. Note that this method
may throw exceptions that occured during file picking.

Note that on Android it can happen that PickFile() can be called twice. In
this case the first PickFile() call will return null as it is effectively
cancelled.

Parameter `allowedTypes`:
Specifies one or multiple allowed types. When null, all file types can be
selected while picking.

On Android you can specify one or more MIME types, e.g. "image/png"; also wild
card characters can be used, e.g. "image/*".

On iOS you can specify UTType constants, e.g. UTType.Image.

On UWP, specify a list of extensions, like this: `".jpg", ".png"`.

On WPF, specify strings like this: `"Data type (*.ext)|*.ext"`, which
corresponds how the Windows file open dialog specifies file types.

### Data structures

The returned `FileData` object contains multiple properties that can be accessed:

    public class FileData
    {
        /// When accessed, reads all data from the picked file and returns it.
        public byte[] DataArray { get; }

        /// Filename of the picked file; doesn't contain any path.
        public string FileName { get; }

        /// Folder path of the picked file/folder.
        public string FolderPath { get; }

        /// Full file path of the picked file/folder; note that on some platforms the
        /// file path may not be a real, accessible path but may contain an
        /// platform specific URI; may also be null.
        public string FilePath { get; }

        /// Returns a stream to the picked file; this is the most reliable way
        /// to access the data of the picked file.
        public Stream GetStream();
    }

Note that `DataArray` is filled on first access, so be sure to rewind the stream when
you access it via GetStream() afterwards.

### **IMPORTANT**
**Android:**
The `READ_EXTERNAL_STORAGE` permission is automatically added to your Android
app project. Starting from Android 6.0 (Marshmallow) the user is asked for the
permission when not granted yet. When the user denies the permission,
`PickFile()` returns null.

The `WRITE_EXTERNAL_STORAGE` is required if you call SaveFile() and must be
added to your Android app project by yourself.

**iOS:** //I don't think this is needed??'
You need to [Configure iCloud Driver for your app](https://developer.xamarin.com/guides/ios/platform_features/intro_to_cloudkit).

## Troubleshooting

### All platforms

**InvalidOperationException "Only one operation can be active at a time"**

This occurs when `PickFile()` is called multiple times and the task being awaited didn't return or
throws an exception that wasn't caught. Be sure to catch any exceptions and handle them
appropriately. See the example code above.

### Android

**Exception "This functionality is not implemented in the portable version of this assembly. You should reference the NuGet package from your main application project in order to reference the platform-specific implementation."**

This occurs when you are using the old-style NuGet references (not the PackageReference mechanism)
and you forgot to add the NuGet package to the Android package. When using PackageReference this
is not necessary anymore because the bait-and-switch assemblies of FilePicker are correctly
resolved.

### iOS

**Picked file path is invalid, file doesn't exist**

On iOS the plugin uses UIDocumentPickerViewController and specifies the mode
UIDocumentPickerMode.Import. After picking is done, iOS copies the picked file
to the app's temporary "Inbox" folder where it can be accessed. iOS also cleans up the
temporary inbox folder regularly. After picking the file you have to either
copy the file to another folder, access the data by getting the property
DataArray or opening a stream to the file by calling GetStream().

## Contributors
* [jfversluis](https://github.com/jfversluis)
* [rafaelrmou](https://github.com/rafaelrmou) (original author)
* [vividos](https://github.com/vividos)
* [saschaelble](https://github.com/EvoPulseGaming)
 
Thanks!

## License
[MIT Licence](LICENSE)
